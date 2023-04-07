---
title: "Take Your Unreal Movie Renders to the Next Level with Custom MRQ Executors"
tags:
  - unreal
  - python
  - rendering
  - pipeline
categories:
  - unreal cinematic
date: 2023-01-16
description: Take your Unreal movie renders to the next level with our tutorial on custom MRQ executors. Learn how to optimize your rendering workflow, improve efficiency, and achieve better results with our step-by-step guide. Whether you're a beginner or an advanced user, our practical examples and best practices will help you master the techniques needed to create stunning movie renders using Unreal and custom MRQ executors.
---

## Introduction

In an earlier blog: [Learn to Automate Unreal Rendering Using Python](https://www.xingyulei.com/post/ue-rendering-basic/)
I mentioned how Unreal Movie Render Queue (MRQ) is more powerful than the legacy
Movie Scene Capture but didn't really have time to show it. 

In this blog,
I will create a customized Unreal Python executor using the Movie Render Pipeline
Plugin; The example script provided by Epic can be found here: 
_"Engine\Plugins\MovieScene\MovieRenderPipeline\Content\Python\MoviePipelineExampleRuntimeExecutor.py"_


## Walk-through

You can find the example code provided by Epic mentioned above, or 
from my render farm tool located [here](https://www.xingyulei.com/post/ue-rendering-remote-farm)

### Executor Class

> An "executor," which is responsible for deciding how a queue 
> is rendered, giving you complete control over the before, during, 
> and after of each render.

The Python class inherit from the `unreal.MoviePipelinePythonHostExecutor`,
is a class override which allows us to make an executor that can process render
jobs in an Unreal standalone commandline launch with '-game' flag; 
It also provides 
async HTTP request functionality, which can be useful in implementing remote rendering.

```python
import unreal


@unreal.uclass()
class MyExecutor(unreal.MoviePipelinePythonHostExecutor):
 
    pipeline = unreal.uproperty(unreal.MoviePipeline)
    queue = unreal.uproperty(unreal.MoviePipelineQueue)
    job_id = unreal.uproperty(unreal.Text)

    def _post_init(self):
        self.pipeline = None
        self.queue = None
        self.job_id = None
```

Instead of using `__init__()` function in Python, the `_post_init()` gets called when created by C++ or Python.

Per the example provided by Unreal, this is also how we declare class properties/variables,
which we have to specify the type.

> Python classes are transient, to allow C++ to spawn a Python
type, we have to specify the type as a member variable. There is special
code in C++ which checks that type and spawns that type (instead of the default)

### Executed Delayed

The `execute_delayed()` is called once the map has finished loading and the 
executor has fully instantiated.

This is the best place to parse commandline arguments and connect to socket,
and also where we can initialize a render job using the commandline arguments.

```python
@unreal.ufunction(override=True)
def execute_delayed(self, queue):
    # parse commandline
    (cmd_tokens, cmd_switches, cmd_parameters) = unreal.SystemLibrary.\
        parse_command_line(unreal.SystemLibrary.get_command_line())

    self.map_path = cmd_tokens[0]
    self.job_id = cmd_parameters['JobId']
    self.seq_path = cmd_parameters['LevelSequence']
    self.preset_path = cmd_parameters['MoviePipelineConfig']

    # initialize pipeline
    self.pipeline = unreal.new_object(
        self.target_pipeline_class,
        outer=self.get_last_loaded_world(),
        base_type=unreal.MoviePipeline
    )

    # initialize queue
    self.queue = unreal.new_object(unreal.MoviePipelineQueue, outer=self)
    job = self.queue.allocate_new_job(unreal.MoviePipelineExecutorJob)
    job.map = unreal.SoftObjectPath(self.map_path)
    job.sequence = unreal.SoftObjectPath(self.seq_path)

    preset_path = unreal.SoftObjectPath(self.preset_path)
    u_preset = unreal.SystemLibrary.\
        conv_soft_obj_path_to_soft_obj_ref(preset_path)
    job.set_configuration(u_preset)
    self.pipeline.initialize(job)
```

### Add Callbacks

In the `execute_delayed()` method, we can add callback functions
when the current pipeline job has finished and when the entire pipeline has finished.
We even have
control over when the pipeline has finished processing individual shots of
a render job's sequence using: `on_movie_pipeline_shot_work_finished_delegate`. 

```python
self.pipeline.on_movie_pipeline_finished_delegate.add_function_unique(
    self,
    "on_job_finished"
)
self.pipeline.on_movie_pipeline_work_finished_delegate.add_function_unique(
    self,
    "on_pipeline_finished"
)
```

There are also callback functions for socket message received: `socket_message_recieved_delete` and
http response received: `http_response_recieved_delegate`. We could
register those in the `_post_init()` method.

The `on_begin_frame()` built-in callback function is called once every frame, 
which can be used to update elapse time, remaining time and the completion percentage of the pipeline.

```python
@unreal.ufunction(override=True)
def on_begin_frame(self):
    super(MyExecutor, self).on_begin_frame()        
    
    if not self.pipeline:
        return
        
    unreal.log(unreal.MoviePipelineLibrary.get_completion_percentage(self.pipeline))
```

#### Pipeline Finished Callbacks

Here's how I implement the job finished callback
`on_movie_pipeline_finished_delegate` and pipeline finished callback `on_movie_pipeline_work_finished_delegate`.

After a job has finished, I manually called the `on_executor_finished_impl()`
which marks the end of the render;
But if there were more than one job to process per execution (for example, a multi-pass
render), we should process
the next job of a render pass during this callback function instead of ending it.

```python
@unreal.ufunction(ret=None, params=[unreal.MoviePipeline, bool])
def on_job_finished(self, pipeline, error):
    self.pipeline = None
    unreal.log("Finished rendering movie!")
    self.on_executor_finished_impl()

    
@unreal.ufunction(ret=None, params=[unreal.MoviePipelineOutputData])
def on_pipeline_finished(self, results):
    output_data = results
    if output_data.success:
        for shot_data in output_data.shot_data:
            render_pass_data = shot_data.render_pass_data
            for k, v in render_pass_data.items():
                if k.name == 'FinalImage':
                    outputs = v.file_paths
                    # get all final output images
                    unreal.log(outputs)
```

### Using Commandline

In the `init_unreal.py` Unreal startup script, we should include the module where we declared
the customized python class.

```python
# init_unreal.py
import myExecutor
```

```python
# myExecutor.py
class MyExecutor(unreal.MoviePipelinePythonHostExecutor):
    pass
```

Now in the commandline, we need to include
`-MoviePipelineLocalExecutorClass=/Script/MovieRenderPipelineCore.MoviePipelinePythonHostExecutor`
and then
`-ExecutorPythonClass=/Engine/PythonTypes.MyExecutor` which is
the custom python executor class.

An example of the full commandline arguments can be found as follows,
I also included the "-JobId" custom flag which we are able to parse above.

```python
command = [
    UNREAL_EXE,
    UNREAL_PROJECT,

    umap_path,
    "-JobId=a5qo3e80",
    "-LevelSequence={}".format(useq_path),
    "-MoviePipelineConfig={}".format(uconfig_path),

    # required
    "-game",
    "-MoviePipelineLocalExecutorClass=/Script/MovieRenderPipelineCore.MoviePipelinePythonHostExecutor",
    "-ExecutorPythonClass=/Engine/PythonTypes.MyExecutor",

    # render preview
    "-windowed",
    "-resX=1280",
    "-resY=720",

    # logging
    "-StdOut",
    "-FullStdOutLogOutput"
]
```

### Preset

For convenience, I created a custom Unreal render preset class
to easily overwrite configuration settings by passing in custom flags.

```python
import unreal


@unreal.uclass()
class MyPreset(unreal.MoviePipelineMasterConfig):

    def __init__(self, preset):
        super(MyPreset, self).__init__(preset)
        self.copy_from(preset)

        self.set_flush_disk()

    @unreal.ufunction(ret=None, params=[])
    def set_flush_disk(self):
        u_setting = self.find_setting_by_class(
            unreal.MoviePipelineOutputSetting
        )
        u_setting.flush_disk_writes_per_shot = True

    @classmethod
    def get_base_preset(cls):
        u_preset = unreal.MoviePipelineMasterConfig()
        u_setting = u_preset.find_setting_by_class(
            unreal.MoviePipelineOutputSetting
        )

        # A job should have a render pass and a file output.
        u_setting.output_resolution = unreal.IntPoint(1280, 720)

        render_pass = u_preset.find_or_add_setting_by_class(
            unreal.MoviePipelineDeferredPassBase
        )
        render_pass.disable_multisample_effects = True

        u_preset.find_or_add_setting_by_class(
            unreal.MoviePipelineImageSequenceOutput_PNG
        )
        u_preset.initialize_transient_settings()
        return cls(u_preset)

    @property
    def output_path(self):
        u_setting = self.find_setting_by_class(
            unreal.MoviePipelineOutputSetting
        )
        return u_setting.get_editor_property('output_directory')

    @unreal.ufunction(ret=None, params=[str])
    def set_output_path(self, path):
        u_setting = self.find_setting_by_class(
            unreal.MoviePipelineOutputSetting
        )
        u_setting.set_editor_property(
            'output_directory',
            unreal.DirectoryPath(path)
        )
```

This can be used like such, the benefit of this is that it will use a base preset
.uasset and create a temporary copy of it to overwrite settings. Once used, the 
preset created are discarded.

```python
@unreal.ufunction(override=True)
def execute_delayed(self, queue):
    # parse commandline
    ...

    self.preset_path = cmd_parameters['MoviePipelineConfig']
    self.output_path = cmd_parameters['OutputDirectory']
    
    ...
    preset_path = unreal.SoftObjectPath(self.preset_path)
    u_preset = unreal.SystemLibrary.conv_soft_obj_path_to_soft_obj_ref(preset_path)
    my_preset = MyPreset(u_preset)
    my_preset.set_output_path(self.output_path)
    ...
```

## Reference

[Unreal Forum - Command Line Rendering With Unreal Engine Movie Render Queue](https://dev.epicgames.com/community/learning/tutorials/nZ2e/command-line-rendering-with-unreal-engine-movie-render-queue)