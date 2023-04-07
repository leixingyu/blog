---
title: "Python REST API: Build and Deploy Your Own Custom Server"
tags:
  - server
  - python
categories:
  - learning log
date: 2023-01-07
description: Learn how to build and deploy your own custom Python REST API server with our comprehensive tutorial. Our step-by-step guide provides practical examples and best practices for creating a secure, scalable, and reliable server that meets your specific needs. Whether you're a beginner or an advanced user, our tutorial will help you master the techniques needed to build and deploy your own custom Python REST API server
---

## HTTP Server Basic

In a client-server model, HTTP server plays the role of a server and implements
using HTTP/HTTPS network protocol. HTTP runs on a socket and uses TCP/IP connection,
but is a higher-level connection that takes care of the send and receive process
in a predefined format.

### Request and Respond

An HTTP request contains a URL, a method, a header and a body;
"POST" and "GET" are the two most common request methods:
- **"POST"**: from the client submits information to the server
- **"GET"**: from the client expect information back from the server

A response is what the client receives from the server in answer to the HTTP 
request. An HTTP response contains a status code, a header and a body

### Rest API

A REST (also known as RESTful) API is a web API that conforms to the constraints of 
REST (representational state transfer) architectural style and allows for interaction with RESTful web services.

When a client request is made via a RESTful API, 
it transfers a representation of the state of the resource to the requester or endpoint.

The touch point of this request/communication is the endpoint. Each endpoint is
the location (URL of a service) from which APIs can access the resources they need to carry out their
function. The resource or information is delivered in one of several formats via HTTP 
(JSON being the most popular).

## Using `http.server`

To build a simple HTTP Server with "POST" and "GET" methods:

```python
import json
import logging
import re
from http.server import BaseHTTPRequestHandler, HTTPServer


class LocalData(object):
    records = {}


class HTTPRequestHandler(BaseHTTPRequestHandler):
    def do_POST(self):
        if re.search('/api/post/*', self.path):
            length = int(self.headers.get('content-length'))
            data = self.rfile.read(length).decode('utf8')

            record_id = self.path.split('/')[-1]
            LocalData.records[record_id] = data

            logging.info("add record %s: %s", record_id, data)
            self.send_response(200)
        else:
            self.send_response(403)
        self.end_headers()

    def do_GET(self):
        if re.search('/api/get/*', self.path):
            record_id = self.path.split('/')[-1]
            if record_id in LocalData.records:
                self.send_response(200)
                self.send_header('Content-Type', 'application/json')
                self.end_headers()

                # Return json, even though it came in as POST URL params
                data = json.dumps(LocalData.records[record_id]).encode('utf-8')
                logging.info("get record %s: %s", record_id, data)
                self.wfile.write(data)

            else:
                self.send_response(404, 'Not Found: record does not exist')
        else:
            self.send_response(403)
        self.end_headers()

        
if __name__ == '__main__':
    server = HTTPServer(('localhost', 8000), HTTPRequestHandler)
    logging.info('Starting httpd...\n')
    try:
        server.serve_forever()
    except KeyboardInterrupt:
        pass
    server.server_close()
    logging.info('Stopping httpd...\n')
```

On the client side, we're utilizing the `requests` module for sending requests:

```python
import requests

id = 'test'
data = {'key': 'value'}
r = requests.post('http://localhost:8000/api/post/{}'.format(id), json=data)
```

Here, we send a "POST" method request to access the `http://localhost:8000/api/post`
endpoint which added a dictionary
type data payload. The data has an id of `test` and content of `{'key': 'value'}`.

```python
id = 'test'
r = requests.get('http://localhost:8000/api/get/{}'.format(id))
print(r.text)
```

Then, we can send a "GET" request to retrieve the data we just added
by accessing another endpoint `http://localhost:8000/api/get/`;
this can be done either through a browser with the corresponding id
(http://localhost:8000/api/get/test), or by sending an HTTP "GET" request using
`requests` module.

The response from the server gives the output: `{'key': 'value'}`

![result](https://i.imgur.com/qNFlDFS.jpg)

## Using `flask`

> Flask is a micro web framework written in Python

I am going to re-created the HTTP server above with "POST" and "GET" endpoints 
using `flask`, this is much simpler:

```python
from flask import Flask
from flask import request

app = Flask(__name__)

class LocalData(object):
    records = {}

# POST endpoint
@app.post('/api/post/<id>')
def create_entry(id):
    data = request.get_json(force=True)
    LocalData.records[id] = data
    return data


# GET endpoint
@app.get('/api/get/<id>')
def get_record(id):
    return LocalData.records[id]
```

Now to run the server, we run the following command `flask --app <appName.py> run`;
Additional `--debug` flag can be added before `run` to auto reload the latest script
change when debugging.

### Rendering Webpage

One of the common cases, when hosting an HTML server, is that users have the ability 
access our endpoint with their browser. Instead of spitting out raw data, we often
want to format it to a nice HTML page.

```python
@app.route('/')
def index_page():
    datas = get_all_local_datas()
    if not datas:
        return 'Welcome!'

    return render_template('index.html', jsons=datas)
```

Here we are specifying an 'index.html' landing page using the root path ('\'),
then a dummpy function that gets all the data from our storage (e.g. database).

**For HTML**

- We can pass in the datas to a variable named `json` which is defined in the .html:

    ```html
    <body>
        <p>{{jsons}}</p>
    </body>
    ```

- Note: it is required that the .html files rest in a folder called "templates",
unless we overwrite that configuration.

**For JavaScript**
- In javascript, we need to use a `jinja2` filter:
    
    ```html
    <script>
        myfunc( {{jsons|tojson|safe}} );
    </script>
    ```

### Linking External Files

If you would like to include additional external `.js` or `.css` files,
they all need to rest in a folder called "static".

Linking the files can then be done as such:

```html
<script src = {{url_for('static', filename="myJS.js")}}></script>
```

```html
<link rel="stylesheet" type="text/css" href= {{url_for('static', filename="main.css")}}>
```

## References

[Cloud Flare - What is HTTP?](https://www.cloudflare.com/learning/ddos/glossary/hypertext-transfer-protocol-http/)

[Red Hat - What is a REST API](https://www.redhat.com/en/topics/api/what-is-a-rest-api)

[Flask](https://flask.palletsprojects.com/en/2.2.x/)

[Linode - Create a RESTful API Using Python and Flask](https://www.linode.com/docs/guides/create-restful-api-using-python-and-flask/)

[Stack Overflow - Difference between socket programing and http programming](https://stackoverflow.com/questions/15108139/)

[Stack Overflow - Pass Variable from python (flask) to HTML in render template?](https://stackoverflow.com/questions/45149420)

[GitHub Gist - simple_server.py by dfrankow](https://gist.github.com/dfrankow/f91aefd683ece8e696c26e183d696c29)
