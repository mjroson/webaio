webaio
======
.. image:: https://circleci.com/gh/francom77/webaio.svg?style=shield
   :target: https://circleci.com/gh/francom77/webaio
.. image:: https://coveralls.io/repos/github/francom77/webaio/badge.svg?branch=master
   :target: https://coveralls.io/github/francom77/webaio?branch=master

Main Features
-------------

* class-bassed ViewSets and Routers to create RESTFul APIs
* APIException class to define and manage errors in an elegant way.
* Middlewares to catch and generate proper HttpResponses when an APIException is raised in the context of a endpoint.
* Scripts to create fully dockerized projects and apps with a set of tools to ease the development, documentation and deployment process.



Installing
----------
    
Install using pip:

.. code-block:: python

    pip install webaio



Creating a project
--------------------
cd into a directory where you’d like to store your code, then run the following command:

.. code-block:: text

    $ webaio-create-project.py

a set of questions will be made in order to parametrize your project:

.. code-block:: text

    $ webaio-create-project.py 
    project_slug [webaio-project]: quickstart
    project_name [Webaio Project]: Quickstart
    description [A short description of the project]: A example project
    port [8080]: 8080

This will create a quickstart directory in your current directory.

Let’s look at what  webaio-create-project.py created:


::

    quickstart/
    ├── apps
    │   └── __init__.py
    ├── conftest.py
    ├── docker-compose.yml
    ├── Dockerfile
    ├── .env
    ├── local.env
    ├── main.py
    ├── README.md
    ├── requirements.txt
    ├── routes.py
    └── settings.py


These files are:

* The outer quickstart/ root directory is a container for your project. Its name doesn’t matter to webaio; you can rename it anytime you like.
* The inner apps/ directory applications will be placed. We will create them in the future.
* quickstart/conftest.py: settings to run tests with pytest. We will write them later.
* quickstart/docker-compose.yml: a simple docker-compose file to run a container with the proyect.
* quickstart/Dockefile: a simple Dockerfile to create the container image.
* quickstart/.env: file to define enviroment variables. This will be used from docker-compose.
* quickstart/local.env: file to define enviroment variables. They will be available from the container and from the settings module.
* quickstart/main.py: module that instantiates and exposes an aiohttp app.
* quickstart/README.md: autogenerated readme with the proyect_name and description asked bellow.
* quickstart/requirements.txt: requirements used by the project template. You will add your own requirements here.
* quickstart/routes.py: module to define our project-level routes.
* quickstart/settings.py: module to define project-level settings. Environment variables will be accessible from here.


Develpment server
-----------------
Let’s verify if the project works. The project template works with Docker and docker-compose. You can find information about `Install docker <https://docs.docker.com/install/>`_ and `Install docker compose <https://docs.docker.com/compose/install/>`_ in the official documentation.

In the project root directory, run:

.. code-block:: text

    $ docker-compose up


For the first time the docker image will be built. After that the development server will run. webaio uses `aiottp-devtools`  development server. You can read the full documentation `here <https://github.com/aio-libs/aiohttp-devtools>`_.

You will see an ouput like the following to indicate the project is running:

.. code-block:: text

    quickstart    | 01-31 14:03 adev.server.dft INFO     Starting aux server at http://0.0.0.0:8081 ◆
    quickstart    | [14:03:15] Starting aux server at http://0.0.0.0:8081 ◆
    quickstart    | [14:03:15] Starting dev server at http://0.0.0.0:8080 ●
    quickstart    | 01-31 14:03 adev.server.dft INFO     Starting dev server at http://0.0.0.0:8080 ●


You can enter to the swagger interface `here <http://localhost:8080/swagger>`_.
We will explain how to register and document enpoints later.


Creating an application
-----------------------
Each application you write in webaio consists of a Python package that follows a certain convention. webaio comes with a utility that automatically generates the basic directory structure of an app, so you can focus on writing code rather than creating directories.

To create your app, make sure you’re in the root directory and type this command:

.. code-block:: text

    $ webaio-create-app.py 
    app_name [app]: api

That’ll create a directory api, which is laid out like this:

::

    api/
    ├── __init__.py
    ├── routes.py
    ├── tests
    │   ├── __init__.py
    │   └── test_api.py
    └── views.py

This directory structure will house the api application.


These files are:

* routes.py: this module exposes a list of route patters. As we will see later this routes can be generated by a router or manually.
* The inner tests/ directory will contain tests for the application.
* views.py: module to define function or class bassed views that can be registered in one or more routes.


Creating a view
----------------

**Function bassed views**

Firstly we will create a function to handle requests to the index enpoint:

`quickstart/api/views.py`


.. code-block:: python

    from aiohttp import web
    

    async def index(request):
        message = 'Welcome to your first webaio project'
        text = f'<h1>{message}</h1>'
        return web.Response(text=text, content_type='text/html')


To call the view, we need to map it to a route.

`quickstart/api/routes.py`

.. code-block:: python

    from aiohttp import web

    from .views import index

    routespatters = [
        web.get('/', index)
    ]

Lastly we will register the api's routes in the project routes:

`quickstart/routes.py`

.. code-block:: python
    

    from apps.api.routes import routespatters as api_routes_patterns

    routespatters = []
    routespatters += api_routes_patterns

You have now wired an index view into the routes. Verify it’s working by accessing to http://localhost:8080/.


**Class bassed views**

A view (or set of views) can be defined as a class bassed view.
To do this we need to extend the `BaseViewSet` class provided by webaio.
There are 6 known methods: create, detail, destroy, list, update and partial_update. Those methods will be routed when the viewset is registered in a router. You can define the ones that you need for your viewset.
If you need other methods you can implement a custom action by decorating the method with the `action` decorator.

Let's imagine we need to manage a list of artists. We will develop a Restful API to acomplish that target. The endpoints we will expose are the following:

* `GET /artist/`: Returns the artists list
* `POST /artist/`: Creates a new artist
* `GET /artist/id/`: Returns the detail of an artist
* `DELETE /artist/id/`: Deletes an artist
* `PUT /artist/id/`:Updates an artist

As an aditional feature we want to be able to retrive the google url of an artist. We will expose this enpoint in the following route:

`GET /artist/id/google-it/`: Returns the artist's google url. 

First of all, we need to define our Viewset in the `views.py` module.

`quickstart/api/views.py`

.. code-block:: python

    import uuid
    from aiohttp import web
    from webaio.decorators import action
    from webaio.viewsets import BaseViewSet


    async def index(request):
        message = 'Welcome to your first webaio project'
        text = f'<h1>{message}</h1>'
        return web.Response(text=text, content_type='text/html')


    class ArtistViewSet(BaseViewSet):

        ARTISTS = dict()

        async def create(request):

            data = await request.json()
            id = uuid.uuid4().hex
            artist = {"id": id, **data}
            ArtistViewSet.ARTISTS[id] = artist
            return web.json_response(artist, status=201)

        async def list(request):
            data = [value for value in ArtistViewSet.ARTISTS.values()]
            return web.json_response(data, status=200)

        async def detail(request):
            id = request.match_info['id']
            data = ArtistViewSet.ARTISTS[id]
            return web.json_response(data, status=200)

        async def destroy(request):

            id = request.match_info['id']
            del ArtistViewSet.ARTISTS[id]
            return web.json_response({"detail": "Ok"}, status=204)

        async def update(request):
            id = request.match_info['id']
            data = await request.json()
            artist = {"id": id, **data}
            ArtistViewSet.ARTISTS[id] = artist
            return web.json_response(artist, status=200)

        @action(detail=True, method='get')
        async def google_it(request):
            id = request.match_info['id']
            name = ArtistViewSet.ARTISTS[id].get('name')
            url = f'https://www.google.com/search?q={name}'
            return web.json_response({'url': url}, status=200)

To expose the viewset we need to register it in a router:

`quickstart/api/routes.py`


.. code-block:: pythoǹ

    from aiohttp import web
    from webaio.routers import SimpleRouter

    from .views import ArtistViewSet, index

    routespatters = []

    router = SimpleRouter()
    router.register(ArtistViewSet, basename='artist')
    routespatters += router.routespatters

    routespatters += [
        web.get('/', index)
    ]


And that's it! You can try the defined enpoints with your favorite Testing API client. 

Testing 
-------
To test the defined endpoints, we will extend the class `AioHTTPTestCase` provided by `aiohttp`. Let's create a simple test case for our viewset.

`quickstart/api/tests/test.py`


.. code-block:: python


    from aiohttp.test_utils import AioHTTPTestCase, unittest_run_loop

    from main import get_web_app


    class TestArtistViewSet(AioHTTPTestCase):

        EXAMPLE_ARTIST = {
            "name": "Gustavo Cerati",
        }

        async def get_application(self):

            app = await get_web_app()
            return app

        async def _create_artist(self):
            data = self.EXAMPLE_ARTIST

            response = await self.client.request(
                "POST", "/artist/", json=data
            )

            assert response.status == 201
            json_response = await response.json()

            return json_response

        @unittest_run_loop
        async def test_create(self):
            await self._create_artist()

        @unittest_run_loop
        async def test_list(self):
            response = await self.client.request("GET", "/artist/")
            assert response.status == 200

        @unittest_run_loop
        async def test_detail(self):
            artist = await self._create_artist()
            artist_id = artist.get('id')
            response = await self.client.request("GET", f"/artist/{artist_id}/")
            assert response.status == 200

        @unittest_run_loop
        async def test_update(self):
            artist = await self._create_artist()
            artist_id = artist.get('id')

            data = {
                "name": "Zeta Bosio"
            }
            response = await self.client.request("PUT", f"/artist/{artist_id}/", json=data)
            assert response.status == 200

        @unittest_run_loop
        async def test_google_it(self):
            artist = await self._create_artist()
            artist_id = artist.get('id')

            response = await self.client.request("GET", f"/artist/{artist_id}/google-it/")
            assert response.status == 200

        @unittest_run_loop
        async def test_destroy(self):
            artist = await self._create_artist()
            artist_id = artist.get('id')

            response = await self.client.request("DELETE", f"/artist/{artist_id}/")
            assert response.status == 204


To run them you can execute the following command:

::

    $ docker-compose run --rm --service-port quickstart pytest

You should see an output like this:

::

    ============================= test session starts ==============================
    platform linux -- Python 3.7.5, pytest-5.3.1, py-1.8.1, pluggy-0.13.1
    rootdir: /app
    plugins: cov-2.8.1, aiohttp-0.3.0
    collected 6 items                                                              

    apps/api/tests/test_api.py ......                                        [100%]

    ============================== 6 passed in 0.32s ===============================


Defining API Exceptions
----------------------
webaio handles APIException subclasses, and deals with returning appropriate error responses. This is accompished by adding the webaio middleware `api_exception_handler` to the `aiohttp` app. If you created the project using the `webaio-create-project.py` script, the middleware is alredy added to the app.
When `APIException` (or subclasses) is raised, webaio will return a response with an appropiate status code. The body of the response will include any additional details regarding the nature of the error.

Let's try what happen if we try to get the detail of a non-existing Artist in our ViewSet. We will write a test to check that: 

`quickstart/api/tests/test.py`


.. code-block:: python

    @unittest_run_loop
    async def test_detail_404(self):
        artist_id = 'non-existent-id'
        response = await self.client.request("GET", f"/artist/{artist_id}/")
        assert response.status == 404

If we run the tests we will see the following error:

::

    ================================================== test session starts ===================================================
    platform linux -- Python 3.7.5, pytest-5.3.1, py-1.8.1, pluggy-0.13.1
    rootdir: /app
    plugins: cov-2.8.1, aiohttp-0.3.0
    collected 7 items                                                                                                        

    apps/api/tests/test_api.py ...F...                                                                                 [100%]

    ======================================================== FAILURES ========================================================
    ___________________________________________ TestArtistViewSet.test_detail_404 ____________________________________________

    self = <apps.api.tests.test_api.TestArtistViewSet testMethod=test_detail_404>

        @unittest_run_loop
        async def test_detail_404(self):
            artist_id = 'non-existent-id'
            response = await self.client.request("GET", f"/artist/{artist_id}/")
    >       assert response.status == 404
    E       AssertionError: assert 500 == 404


This happens because the error is not properly handdled in the view.

Firstly we need to add an `exceptions.py` module and define the exception in there:

`quickstart/api/exceptions.py`


.. code-block:: python

    from webaio.exceptions import APIException


    class ArtistDoesNotExist(APIException):
        status_code = 404
        detail = "Artist does not exist"


After that, we will modify the detail view:

`quickstart/api/views.py`


.. code-block:: python


    async def detail(request):
        id = request.match_info['id']
        try:
            data = ArtistViewSet.ARTISTS[id]
        except KeyError:
            raise ArtistDoesNotExist()

        return web.json_response(data, status=200)


We can check the behavior by running the tests:

::

    ================================================== test session starts ===================================================
    platform linux -- Python 3.7.5, pytest-5.3.1, py-1.8.1, pluggy-0.13.1
    rootdir: /app
    plugins: cov-2.8.1, aiohttp-0.3.0
    collected 7 items                                                                                                        

    apps/api/tests/test_api.py .......                                                                                 [100%]

    =================================================== 7 passed in 0.36s ====================================================


