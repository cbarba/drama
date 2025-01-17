## [drama](https://github.com/benhid/drama) 

![CI](https://github.com/benhid/drama/actions/workflows/ci.yml/badge.svg)
![Release](https://github.com/benhid/drama/actions/workflows/release.yml/badge.svg)
![GitHub Release](https://img.shields.io/github/release/benhid/drama.svg)
![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)

*`drama` is an asynchronous workflow executor engine supported by a command line tool and RESTful API.*

Proudly based on [Dramatiq](https://dramatiq.io/). This project requires the following dependencies to work:

* [Apache Kafka](https://kafka.apache.org/)
* RabbitMQ
* Mongodb
* MinIO / HDFS

You can setup a minimal development environment using Docker Compose.

```commandline
docker-compose [-f docker-compose-kafka.yml] up -d
```

### 🚀 Setup 

#### Installation

Via source code using [Poetry](https://github.com/python-poetry/poetry):

```commandline
git clone https://github.com/benhid/drama.git
cd drama
poetry install
```

Before running `drama`, save a copy of [`.env.template`](.env.template) as `.env` and insert your own values. 
`drama` will then look for a valid `.env` file in the **current working directory**. In its absence, it will attempt to determine the values from environmental variables (with prefix `DRAMA_`).

After that you can run:

```commandline
poetry run drama -h
```

#### Spawn workers

Spawns multiple concurrent worker processes to execute tasks.

```commandline
poetry run drama worker --processes 4
```

> For a full list or valid command line arguments that can be passed to `drama worker`, checkout `dramatiq -h`

#### Deploy server 

_(optional)_

Server can be [deployed](https://fastapi.tiangolo.com/deployment/) with *uvicorn*, a lightning-fast ASGI server, using the command-line client.

```commandline
poetry run drama server
```

Alternatively, use the provided [`Dockerfile`](Dockerfile):

```commandline
sudo docker build . -t drama-server
sudo docker run -p 8004:8004 drama-server
```

Online documentation is available at `/api/docs`.

### ✨ Getting started

#### Defining components

Take a look at the [catalog](examples) for some examples on how to develop your own components.

#### Executing workflows

##### Using Python

To run a workflow from Python, provide a list of `TaskRequest`s and wrap them up in a `WorkflowRequest`.

```python
from drama.models.task import Task
from drama.models.workflow import Workflow
from drama.worker.scheduler import Scheduler

task_import_file = Task(
    name="LoadIrisDataset",
    module="drama.core.catalog.load.ImportFile",
    params={
        "url": "https://gist.githubusercontent.com/netj/8836201/raw/6f9306ad21398ea43cba4f7d537619d0e07d5ae3/iris.csv"
    }
)

workflow_request = Workflow(tasks=[task_import_file])

with Scheduler() as scheduler:
    scheduler.run(workflow_request)
```

_(This script is complete, it should run "as is")_

##### Using requests

Alternatively, a workflow can also be run using the built-in server.

```python
import requests

r = requests.post(
    url='http://localhost:8001/api/v2/workflow/run',
    json={
      "tasks": [
        {
          "name": "LoadIrisDataset",
          "module": "drama.core.catalog.load.ImportFile",
          "params": { 
            "url": "https://raw.githubusercontent.com/cs109/2014_data/master/countries.csv"
          }
        }
      ]
    })
print(r.text)
```

_(This script is complete, it should run "as is")_
