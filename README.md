webaio
======

webaio provides a simple way to create async web applications based on aiottp.

Main Features
-------------

* class-bassed ViewSets and Routers to create RESTFul APIs
* APIException class to define and manage errors in an elegant way.
* Middlewares to catch and generate proper HttpResponses when an APIException is raised in the context of a endpoint.
* Scripts to create fully dockerized projects and apps with a set of tools to ease the development, documentation and deploy.



Installing
----------

Install using pip:

.. code-block:: text

    pip install webaio



Creating a project
--------------------
cd into a directory where you’d like to store your code, then run the following command:

.. code-block:: text

    webaio-create-project.py

a set of questions will be made in order to parametrize your project:

.. code-block:: text

    $ webaio-create-project.py 
    project_slug [webaio-project]: quickstart
    project_name [Webaio Project]: Quickstart
    description [A short description of the project]: A example project

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

* The outer quickstart/ root directory is a container for your project. Its name doesn’t matter to webaio; you can rename it to anything you like.
* The inner apps/ directory will contain the applications. We will create them in the future.
* quickstart/conftest.py: settings to run tests with pytest
* quickstart/docker-compose.yml: a simple docker-compose file to run a container with the proyect.
* quickstart/Dockefile: a simple Dockerfile to create the container image.
* quickstart/.env: file to define enviroment variables. This will be used from docker-compose.
* quickstart/local.env: file to define enviroment variables. This will be used from docker-compose.
* quickstart/main.py: module that instantiate and exposes an aiohttp app.
* quickstart/README.md: autogenerated readme with the proyect_name and description asked bellow.
* quickstart/requirements.txt: requirements used by the project template.
* quickstart/routes.py: module to define our project-level routes.
* quickstart/settings.py: module to define project-level settings. Environment variables will be accessible from here.

