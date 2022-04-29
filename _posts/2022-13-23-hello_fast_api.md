---
layout: custom
title: "Startup project with FastAPI"
date: 2022-11-23
---
# Table of contents
* [Overview](#1)
* [Pet project](#2)updated date
* [Local Run](#3)
* [Scope](#4)
* [Testing](#5)
* [Considerations](#6)
* [References](#7)

##Overview <a name="1">
For the series "**startup project with...**" I'm going to approach python web development and I've tried to use [FastAPI web framework](https://fastapi.tiangolo.com/). I was very surprised by it's easy of use and I wanted to go in deeper about it; above all it's testing features.

##Pet project <a name="2">
In order to test functionality I implemented this [pet project](https://github.com/sabatinim/fast_api_hello_world). Basically it is a simple server with one rest api.
###Local Run <a name="3">
After checkout the code you can simply execute "script/ci" on your terminal application. The script will execute these steps:
- build python app in docker container
- code checks (like unused code)
- test execution

If you want to try the API in a local host container you can just execute these commands:
```bash
docker-compose up api  
curl http://localhost:8080/items/123\?q\="test"
```
The second one should provide this response:
```bash
{"item_id":123,"q":"test","use_case":"Production Code"}
```
##Scope <a name="4">
Regarding the code design I just wanted to achieve two points:
1. encapsulate controller api on a class
2. inject collaborator in order to easily test API

Regarding the point 1) I have:
```python
class AppController:
    def __init__(self, app: FastAPI, use_case: UseCase):
        @app.get("/items/{item_id}")
        def read_item(item_id: int, q: Optional[str] = None):
            return {
                "item_id": item_id, "q": q, "use_case": use_case.run()
``` 
The class AppController could be built with a FastAPI and a UseCase objects.
The first one is something related FastAPI configuration and run the second one is my business logic.

Having this class I can configure my application using a startup() method where wiring the code I need for the App:
```python
def startup(use_case: UseCase = ProductionUseCase()):
    app = FastAPI()
    AppController(app, use_case)
    return app
```
Finally I can run my application using docker container. This is the container definition:
```docker
FROM python:3.10.3-slim-bullseye

WORKDIR /code
COPY ./requirements.txt /code/requirements.txt

RUN pip install --no-cache-dir --upgrade -r /code/requirements.txt
COPY ./app /code/app
CMD ["uvicorn", "app.main:startup", "--host", "0.0.0.0", "--port", "8080", "--workers", "4"]
```
##Testing <a name="5">
Off course I can also use startup() method for injecting dependencies in order to easily test the application.
These are two tests for the controller:

```python
 def test_e2e_production_code(self):
        client = TestClient(startup())
        response = client.get("/items/987?q=this%20is%20the%20query")
        self.assertEqual(200, response.status_code)
        self.assertEqual(
            {"item_id": 987, "q": "this is the query", "use_case": "Production Code"},
            response.json()
        )

class TestableUseCase(UseCase):
    def run(self):
        return "Test Code"

 def test_e2e_testable_code(self):
        client = TestClient(startup(TestableUseCase()))
        response = client.get("/items/987?q=this%20is%20the%20query")
        self.assertEqual(200, response.status_code)
        self.assertEqual(
            {"item_id": 987, "q": "this is the query", "use_case": "Test Code"},
            response.json()
        )
``` 
The first test is using production code dependencies, building the client with production use case:
```python
client = TestClient(startup())
```
Sometimes, during the development, we cannot use real connection or we just want to decompose test (unit, integration ...) using stubbed collaborators [(ex. contract test)](https://dev.to/ticinoswcraft/tests-infrastructure-1gko).
For this reason we could inject a TestableCollaborator like in the second test (**test_e2e_testable_code**):
```python
client = TestClient(startup(TestableUseCase()))
```
##Considerations <a name="6">
From design point of view FastAPI is super useful framework to use to build our microservices. Fits very well with container logic and cloud development. It's also very easy to build a test suite for the application we're going to develop.
Of course you have to pay attention to not introduce complexity with some feature it provides like dependencies or DBMS access.

####References <a name="7">
[Github project link](https://github.com/sabatinim/fast_api_hello_world)