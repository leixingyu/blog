---
title: "Boost Your Unreal Rendering Workflow with Python Automation"
tags:
  - unreal
  - python
  - rendering
categories:
  - unreal cinematic
date: 2022-12-03
description: Discover how to boost your Unreal rendering workflow with Python automation using our comprehensive tutorial. Our guide provides practical examples and best practices for optimizing your production pipeline, automating tedious tasks, and improving your overall efficiency. Whether you're a beginner or an advanced user, our step-by-step tutorial will help you master the techniques needed to take your Unreal rendering workflow to the next level with Python automation
---
  

## Introduction

In this post, I'll cover Python automation using Movie Render Queue (MRQ) and 
Movie Scene Capture.

I'll go over how to make Python tools to run in the Unreal Editor as well
as outside Unreal through command line; How to change settings and add call backs.

The MRQ is more preferred over Movie Scene Capture as it's much more customizable,
both in supported render setting and in automation (In a later post you'll see).

## Movie Render Queue

> Movie Render Queue is a successor to the Sequencer Render Movie feature, and is built for higher quality, easier integration into production pipelines, and user extensibility. With Movie Render Queue you can accumulate multiple render samples together to produce the final output frame, which allows for higher quality anti-aliasing, radial motion blur, and reduced noise in ray tracing.

The Movie Render Queue (MRQ) is the latest and the greatest from Unreal for
Rendering out cinematic, over the legacy Movie Scene Capture Tool.

![MRQ](https://docs.unrealengine.com/4.26/Images/AnimatingObjects/Sequencer/Workflow/RenderAndExport/HighQualityMediaExport/settings2.webp)

### Movie Render Queue in Editor

Here you can find the full example from Epic: 
`Engine\Plugins\MovieScene\MovieRenderPipeline\Content\Python\MoviePipelineEditorExample.py`

#### Bare Minimal

Through Unreal Editor, we use a render executor called PlayInEditor (PIE) executor.

```python
def movie_queue_render(u_level_file, u_level_seq_file, u_preset_file):
    subsystem = unreal.get_editor_subsystem(unreal.MoviePipelineQueueSubsystem)
    queue = subsystem.get_queue()
    executor = unreal.MoviePipelinePIEExecutor()
    
    # config render job with movie pipeline config
    job = queue.allocate_new_job(unreal.MoviePipelineExecutorJob)
    job.job_name = 'test'
    job.map = unreal.SoftObjectPath(u_level_file)
    job.sequence = unreal.SoftObjectPath(u_level_seq_file)
    preset = unreal.EditorAssetLibrary.find_asset_data(u_preset_file).get_asset()
    job.set_configuration(preset)
    
    subsystem.render_queue_with_executor_instance(executor)
```

#### Override Settings

All the render settings are stored as an `unreal.MoviePipelineMasterConfig` object.

It has sub setting components class `unreal.MoviePipeline`, for example: `unreal.MoviePipelineOutputSetting`,
`unreal.MoviePipelineCameraSetting`, `unreal.MoviePipelineColorSetting` and etc

Each Setting class then have properties, for `unreal.MoviePipelineOutputSetting`, it has
properties such as:
- `use_custom_frame_rate`
- `frame_number_offset`
- `output_directory`
- etc

All of which can be modified, attached, and removed.

Example:
```python
uPreset = unreal.MoviePipelineMasterConfig()
uSetting = uPreset.find_setting_by_class(unreal.MoviePipelineOutputSetting)
uSetting.output_resolution = unreal.IntPoint(1280, 720)
```

#### Callback

Callbacks can be added upon each Shot, Job, Queue Finished. The callback function
has to be kept as global variable here to prevent it being destroyed by GC.

```python
def render_finished(params):
    unreal.log(params)

executor.on_individual_job_work_finished_delegate.add_callable_unique(render_finished)
```

or.

```python
def render_errored(executor, pipeline, is_fatal, error_msg):
    pass

ERROR_CALLBACK = unreal.OnMoviePipelineExecutorErrored()
ERROR_CALLBACK.add_callable(render_errored)
exector.on_executor_errored_delegate = ERROR_CALLBACK
```

See my convenient class for PIE here: https://github.com/leixingyu/unrealUtil/blob/master/render/render.py

### Movie Render Queue Commandline

The default MRQ command line offers customization by using 
either an Unreal Render Preset/Config asset or an Unreal Render Queue asset.

Since there are so many settings in the MRQ you can modify, it's hard to expose
all of the little details through command line. If you do wish to do so, I have
an entire blog about [Unreal MRQ Custom Executor](https://www.xingyulei.com/post/ue-rendering-custom-executor).

```python
def movie_queue_render(u_level_file, u_level_seq_file, u_preset_file):
    command = [
        UNREAL_EXE,
        U_PROJECT,
        u_level_file,

        # required
        "-LevelSequence=%s" % u_level_seq_file,  # The sequence to render
        "-MoviePipelineConfig=\"%s\"" % u_preset_file,
        "-game",

        # options
        "-NoLoadingScreen",
        "-log",

        # window size not resolution
        "-Windowed",
        "-ResX=800",
        "-ResY=600",
    ]
    proc = subprocess.Popen(command)
    return proc.communicate()
```


## Movie Scene Capture

**Now this part may or may not be relevant anymore as many people treats the
Movie Scene Capture as deprecated**. But I thought I write it down anyway,
as it might be helpful as a comparison to MRQ

### Movie Scene Capture Editor

![MSC](https://docs.unrealengine.com/4.27/Images/AnimatingObjects/Sequencer/Overview/RenderMovieSettings/MovieSettingsWindow.webp)

Here you can find an example from Epic:

`Engine\Plugins\MovieScene\SequencerScripting\Content\Python\sequencer_examples.py`

#### Bare Minimal

```python
import unreal
capture = unreal.AutomatedLevelSequenceCapture()
capture.level_sequence_asset = unreal.SoftObjectPath(sequencer_path)
unreal.SequencerTools.render_movie(capture, unreal.OnRenderMovieStopped())
```

In this example, you only need to supply the path to the Unreal sequencer asset.

Note that the render happens in the current level, so make sure to pre-load the map
before running the Movie Scene Capture.

#### Callback

```python
def on_render_movie_finished(success):
	unreal.log("Movie has finished rendering")
	
on_finished_callback = unreal.OnRenderMovieStopped()
on_finished_callback.bind_callable(on_render_movie_finished)
```

Note the `on_finished_callback` is a global variable so it is persistent and won't
get destroyed during GC.

The next step is to register the callback function:
`unreal.SequencerTools.render_movie(capture_settings, unreal.OnRenderMovieStopped())`

#### Override Setting

The settings can be modified through class [`unreal.MovieSceneCaptureSettings`](https://docs.unrealengine.com/5.0/en-US/PythonAPI/class/MovieSceneCaptureSettings.html#unreal.MovieSceneCaptureSettings)

Example:
```python
capture = unreal.AutomatedLevelSequenceCapture()
setting = capture.settings
setting.output_directory = unreal.DirectoryPath(export_path)
```

### Movie Scene Capture Commandline

A lot of custom flags can be specified through the command line, see the full list
of control commands: [Command Line Arguments for Rendering Movies](https://docs.unrealengine.com/4.26/en-US/AnimatingObjects/Sequencer/Workflow/RenderAndExport/RenderingCmdLine/)

Console variables can even be set through command line which is decent.

```python
import subprocess

def movie_scene_capture(u_level_file, u_level_seq_file, output_folder):
    command = [
        UNREAL_EXE,
        U_PROJECT,
        u_level_file,

        # required
        "-LevelSequence=%s" % u_level_seq_file,  # The sequence to render
        "-MovieSceneCaptureType=/Script/MovieSceneCapture.AutomatedLevelSequenceCapture",
        "-NoLoadingScreen",
        "-game",
        "-log",
        "-MovieCinematicMode=yes",

        # general property
        "-MovieFolder=%s" % output_folder,

        # if doing .avi
        "-MovieFormat=Video",  # JPG, BMP, PNG or Video
        "-MovieFrameRate=24",
        "-MovieQuality=100",  # compression quality in percentage
        "-Windowed",
        "-ResX=1920",
        "-ResY=1080",

        # other
        "-NoTextureStreaming",  # for final render
        "-NoScreenMessages",  # no screen debug message
    ]
    proc = subprocess.Popen(command)
    return proc.communicate()
```

## Additional References

[Unreal Forums - Unable to execute MoviePipelineQueue from Python](https://forums.unrealengine.com/t/unable-to-execute-moviepipelinequeue-from-python/467250/5)
