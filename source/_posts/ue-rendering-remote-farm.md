---
title: "Unreal Distributed Rendering Server Handbook: Render Farm Implementation"
tags:
  - unreal
  - python
  - rendering
categories:
  - unreal cinematic
date: 2023-02-04
description: Get started with implementing an Unreal Distributed Rendering Server for your render farm using our comprehensive handbook. Our step-by-step guide provides practical examples and best practices for setting up a scalable and efficient rendering environment that meets your specific needs. Whether you're a beginner or an advanced user, our handbook will help you master the techniques needed to implement an Unreal Distributed Rendering Server and streamline your rendering pipeline
---

## Introduction

> Previous blogs that you may find relevant
> 
> - [Learn to Automate Unreal Rendering Using Python](https://www.xingyulei.com/post/ue-rendering-basic/)
> - [Unreal Movie Render Queue (MRQ) Custom Executor](https://www.xingyulei.com/post/ue-rendering-custom-executor/)
> - [Building HTTP Server with REST API in Python](https://www.xingyulei.com/post/py-http-server/)

> GitHub link to the complete project code:
[Unreal Render Farm](https://github.com/leixingyu/unrealRenderFarm)

The first thing to do when creating a distributed remote render farm is to establish
its framework. I personally have not much experience using render farm, not to
mention designing one. (It is not surprising since I don't see anyone would build
a render farm for personal use). Nevertheless, it's still such an interesting
topic that I couldn't resist exploring.

**Important: the following is a simple implementation of a distributed rendering server/
render farm with dummy functionalities and are meant to be substituted based on project needs.**

## Distributed Rendering Framework

Here are two examples of how other farm system is structured

> Update: I noticed Epic Games **just** posted a new blog which introduces an
> overview of setting up the render farm. It is worth the read.
> Moreover, it also includes the source code of remote rendering
> in Deadline, the code is not hard to follow but it's different from my implementation.
> See more details here: [Overview of Setting Up A Render Farms With Unreal Engine](https://dev.epicgames.com/community/learning/tutorials/xY9O/overview-of-setting-up-a-render-farms-with-unreal-engine)

### Blender Network Rendering

This is a blender legacy network rendering system. It consists of three roles:

- Regular client: 
  - client, sends render requests
- Slave: 
  - also a client, used to process render job
- Master: 
  - server, coordinate (process requests from clients and assign jobs to slave)

The clients need to pre-configure server address in order to talk to master.
For both types of clients, they have the ability to send request to update job status (add, delete, modify).
A regular client can also request information about the currently active slaves connected
to the server.


### Unreal Swarm Farm

This is an Unreal farm system to build lighting. There are two components:

- Swarm agent: 
  - client, an agent can send light map bake request, it also can process light map bake from the server
- Swarm coordinator: 
  - server, used to coordinate light map bake request

Similarly, the swarm agent initialize connection by pinging coordinator using a
pre-configured address. The swarm agent role can be split into two which exactly
maps to the role of a regular client and a slave in the Blender Network Rendering
Example.

There is still one question remains: how does the server sends job to client?

- the client send request for jobs periodically 
- a listener server on client side that the server pings to
- broadcast
- [long polling](https://blog.oddbit.com/post/2013-11-23-long-polling-with-ja/)

I decided to go with the easiest setup: client request for jobs periodically.

### Summary

![framework](https://i.imgur.com/h6BqZBP.jpg)

So, finally I have my Unreal remote rendering framework like above:

- Manager: server, used to host the HTTP server, stores job request
information connected to a database, and coordinates/manages request among render nodes


- Submitter: client, connect to the manager and submit render job request
  - since the submitter only need to send request, this could be entirely web-based


- Worker: client, connect to manager, periodically pings for new job
to process, renders the job and periodically pings to update current render progress
  - **requires Unreal installed, capable for rendering jobs**

  
## Render Job Request Data Structure

### Structure

Here's a data class I've made for representing a render job request, which fundamentally
is a dictionary structure:

```python
class RenderRequest(object):
    def __init__(
            self,
            uid='',
            name='',
            owner='',
            worker='',
            time_created='',
            priority=0,
            category='',
            tags=[],
            status='',
            umap_path='',
            useq_path='',
            uconfig_path='',
            output_path='',
            width=0,
            height=0,
            frame_rate=0,
            format='',
            start_frame=0,
            end_frame=0,
            time_estimate='',
            progress=0
    ):
        self.uid = uid or str(uuid.uuid4())[:4]
        self.name = name
        self.owner = owner or socket.gethostname()
        self.worker = worker
        self.time_created = time_created or datetime.now().strftime("%m/%d/%Y, %H:%M:%S")
        self.priority = priority or 0
        self.category = category
        self.tags = tags
        self.status = status or RenderStatus.unassigned
        self.umap_path = umap_path
        self.useq_path = useq_path
        self.uconfig_path = uconfig_path
        self.output_path = output_path
        self.width = width or 1280
        self.height = height or 720
        self.frame_rate = frame_rate or 30
        self.format = format or 'JPG'
        self.start_frame = start_frame or 0
        self.end_frame = end_frame or 0
        self.length = self.end_frame - self.start_frame
        self.time_estimate = time_estimate
        self.progress = progress
```

### Database

For the time-being and testing purpose, I didn't deploy an actual database. Instead,
I used localized .json files for render job request storage. 
A render request's unique `uid` serves as the primary key.

Each render request is serialized as a .json file mimicing each row of a database, 
therefore, making each render job independent: 
a GET method queries one .json file (an entry) or all .json files in the 'database'; 
a POST method creates a .json file;
a PUT method updates a .json file;
and a DELETE method removes a .json file.

```python
def remove_db(uid):
    """
    Remove a RenderRequest object from the database
    :param uid: str. request uid
    """
    os.remove('{}.json'.format(uid))
```

## Manager

I'm using flask - a micro web framework in Python to host my HTTP server with REST API,
which makes it easier for my client to communicate with the manager. 
For more information on that, check out my previous blog: [Building HTTP Server with REST API in Python](https://www.xingyulei.com/post/py-http-server/#using-flask)

### REST API

For the server communication part, I implemented four methods:

- GET: api for a render job(s) query request from the clients (submitter or worker)
- POST: api for a new render job creation request from the submitter
- PUT: api for a render job update request from the worker
- DELETE: api for a render job deletion request from the clients

```python
@app.delete('/api/delete/<uid>')
def delete_request(uid):
    """
    Server DELETE api response, delete render request from database
    :param uid: str. render request uid
    """
    renderRequest.remove_db(uid)
```

### Job Coordination

Another important role of the manager is to coordinate jobs among the available
worker/render node. This is obviously a complex problem to solve on its own and
is not the focus of my simple render farm implementation, thus, I currently have
it automatically assign any new job to a specific worker:

```python
def new_request_trigger(rrequest):
    """
    Triggers when a client posts a new render request to the server
    """
    if rrequest.worker:
        return

    # currently, as a test, assigns all job to one worker
    worker = 'RENDER_MACHINE_01'
    assign_request(rrequest, worker)

    # if multiple jobs came in at once, interval between each assignment
    time.sleep(4)
    LOGGER.info('assigned job %s to %s', rrequest.uid, worker)
```


### Render HTML Landing Page

I experimented with rendering the index page to display the all render job
requests in the database, which looks like this:

![index-page](https://i.imgur.com/nk6CKQY_d.jpg?maxwidth=520&shape=thumb&fidelity=high)

```python
@app.route('/')
def index_page():
    rrequests = renderRequest.read_all()
    jsons = [rrequest.to_dict() for rrequest in rrequests]
    return render_template('index.html', requests=jsons)
```

The tables are generated using a javascript where the columns can be customized

```javascript
function createTable(list){
    // header row
    cols = ['uid', 'name', 'owner', 'worker', 'time_created', 'status', 'priority', 'time_estimate', 'progress'];
    
    var table = document.createElement("table");
    var tr = table.insertRow(-1);
    for (var i = 0; i < cols.length; i++) {
        // header th elements
        var theader = document.createElement("th");
        theader.innerHTML = cols[i];
        tr.appendChild(theader);
    }

    // data rows
    for (var i = 0; i < list.length; i++) {
        var trow = table.insertRow(-1);
        var uid = list[i]['uid'];
        for (var j = 0; j < cols.length; j++) {
            var cell = trow.insertCell(-1);
            cell.innerHTML = list[i][cols[j]];
        }
    }

    var el = document.getElementById("table");
    el.innerHTML = "";
    el.appendChild(table);
}
```

For the progress bar render

```javascript
// add id tag to specific cells for ajax
if (cols[j] == 'progress'){
    // container
    cell.innerHTML = '';
    cell.style.width = '200px';
    var container = document.createElement("div");
    cell.appendChild(container);

    // bar
    var bar = document.createElement("div");
    container.appendChild(bar);
    bar.style.width = list[i]['progress'] + '%';
    bar.innerHTML = list[i]['progress'] + '%';
}
```

### Asynchronous Update using AJAX

To update data on the index page asynchronously with the PUT method, I opted in
for AJAX (Asynchronous JavaScript and XML).

I've attached ids to specific HTML elements and set it up
so the webpage updates the _render status_, _time remaining estimated_ and
_overall progress_ every given period of time.

```javascript
$(document).ready(function(){
    setInterval(
        function(){ajaxRequest()},
        2000)
})

function ajaxRequest(){
    $.getJSON('/api/get', function(data){
        updateProgress(data);
    })
}

function updateProgress(data){
    var requests = data.results;
    for (var i = 0; i < requests.length; i++){
        var uid = requests[i]['uid'];
        var progress = requests[i]['progress'];
        var time_estimate = requests[i]['time_estimate'];
        var status = requests[i]['status'];

        var bar = document.getElementById(uid + '_progress');
        bar.style.width = progress + '%';
        bar.innerHTML = progress + '%';

        document.getElementById(uid + '_estimate').innerHTML = time_estimate;
        document.getElementById(uid + '_status').innerHTML = status;
    }
}
```

## Submitter

Since the server side is established, the submitter part is actually quite
easy. All we need to do is to pass a data with render request data structure
and use the POST method (I'm also using the `requests` standard library)

```python
import requests


def add_request(d):
    try:
        response = requests.post(SERVER_API_URL+'/post', json=d)
    except requests.exceptions.ConnectionError:
        LOGGER.error('failed to connect to server %s', SERVER_API_URL)
        return
    
    return RenderRequest.from_dict(response.json())

    
if __name__ == '__main__':
    test_job = {
        'name': 'test',
        'owner': 'TEST_SUBMITTER_01',
        'umap_path': '/Game/Cinematics/Street/Level_Cin_Street.Level_Cin_Street',
        'useq_path': '/Game/Cinematics/Street/LS_Master_Street.LS_Master_Street',
        'uconfig_path': '/Game/Cinematics/Preset/Test.Test'
    }
    
    rrequest = add_request(test_job)
    if rrequest:
        LOGGER.info('request %s sent to server', rrequest.uid)
```

## Worker

Each worker represents a render node; The render node unlike other render machine,
is a full-fledged PC with the complete Unreal Engine installed. In order to render
the requested job, it also needs to have the corresponding Unreal project installed.

Before a worker starts rendering a job, it first needs to fetch the latest change
of the Unreal project, this can be done using the versioning system supported by
Unreal, like Unreal Game Sync with Perforce.

When a worker is brought online, it continuously works on the job assigned to it,
then periodically pings the server for new jobs. 

When a job is available, the worker launches an Unreal render process from the
command line and calls for the custom Unreal Python executor.

### Unreal Custom Executor

An Unreal custom executor instance is created using command line, for more detail
please read the blog regarding [Unreal Movie Render Queue (MRQ) Custom Executor](https://www.xingyulei.com/post/ue-rendering-custom-executor/)

```python
def render(uid, umap_path, useq_path, uconfig_path):
    command = [
        UNREAL_EXE,
        UNREAL_PROJECT,
        umap_path,
        "-JobId={}".format(uid),
        "-LevelSequence={}".format(useq_path),
        "-MoviePipelineConfig={}".format(uconfig_path),
        "-game",
        "-MoviePipelineLocalExecutorClass=/Script/MovieRenderPipelineCore.MoviePipelinePythonHostExecutor",
        "-ExecutorPythonClass=/Engine/PythonTypes.MyExecutor",
    ]
    env = os.environ.copy()
    env["UE_PYTHONPATH"] = MODULE_PATH
    proc = subprocess.Popen(
        command,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,
        env=env
    )
    return proc.communicate()
```

Within the executor class, a PUT request is sent to the server after
every frame render with the latest time remaining estimate and overall
pipeline progress.

```python
@unreal.uclass()
class MyExecutor(unreal.MoviePipelinePythonHostExecutor):
  
    ...

    @unreal.ufunction(override=True)
    def on_begin_frame(self):
        super(MyExecutor, self).on_begin_frame()

        if not self.pipeline:
            return

        status = renderRequest.RenderStatus.in_progress
        progress = 100 * unreal.MoviePipelineLibrary.\
            get_completion_percentage(self.pipeline)
        time_estimate = unreal.MoviePipelineLibrary.\
            get_estimated_time_remaining(self.pipeline)

        if not time_estimate:
            time_estimate = unreal.Timespan.MAX_VALUE

        days, hours, minutes, seconds, _ = time_estimate.to_tuple()
        time_estimate = '{}h:{}m:{}s'.format(hours, minutes, seconds)

        self.send_http_request(
            '{}/put/{}'.format(client.SERVER_API_URL, self.job_id),
            "PUT",
            '{};{};{}'.format(progress, time_estimate, status),
            unreal.Map(str, str)
        )
```

## Conclusion

And that's concludes the break-down of my dummy distributed rendering system; once
again the complete project source code can be found [here](https://github.com/leixingyu/unrealRenderFarm)
along with a [video demo](https://youtu.be/4pGhaMQACy8)

There are still lots of components that can be expanded further, namely the job
coordination functionality, the web application implementation 
(can replace the submitter), the database implementation and actually testing
this across multiple machines in a network. Unfortunately, I don't have the 
resources to touch upon these.

Hope this is useful and see you in the next blog!

## References

[YouTube - UE4 | Creating a Swarm Farm](https://youtu.be/30AtJT8yoR8)

[YouTube - DIY Renderfarm Building Tutorial for Distributed Blender Rendering](https://youtu.be/FNhUnPWzVaw)

[CG Director - How to Build your own Render Farm [Ultimate Guide]](https://www.cgdirector.com/how-to-build-your-own-render-farm/)