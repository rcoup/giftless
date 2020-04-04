Giftless - a Pluggable Git LFS Server
=====================================

[![Build Status](https://travis-ci.org/datopian/giftless.svg?branch=master)](https://travis-ci.org/datopian/giftless)
[![Maintainability](https://api.codeclimate.com/v1/badges/58f05c5b5842c8bbbdbb/maintainability)](https://codeclimate.com/github/datopian/giftless/maintainability)

Giftless a Python implementation of a [Git LFS](1) Server. It is designed 
with flexibility in mind, to allow pluggable storage backends, transfer 
methods and authentication methods. 

Giftless supports the *basic* Git LFS transfer mode with the following 
storage backends:
* Local storage 
* [Azure Blob Storage](https://azure.microsoft.com/en-us/services/storage/blobs/) 
  with direct-to-cloud or streamed transfers 

Additional transfer modes and storage backends could easily be added and
configured.

Installation & Quick Start
--------------------------

### Running using Docker
You can build a Docker image as described below and then run it:

    $ docker run --rm -P 5000:5000 datopian/giftless
    
This will run the server in WSGI mode, which will require an HTTP server 
such as *nginx* to proxy HTTP requests to it. 

Alternatively, you can specify the following command line arguments to 
have uWSGI run in HTTP mode, if no complex HTTP setup is required:

    $ docker run --rm -P 8080:8080 datopian/giftless \
        -M -T --threads 2 -p 2 --manage-script-name --callable app \
        --http 0.0.0.0:8080


### Installing & Running from Source
You can install and run `giftless` from source:

    $ git clone git@gitlab.com:datopian/giftless.git
    
    # Initialize a virtual environment
    $ cd giftless
    $ python3 -m venv .venv
    $ . .venv/bin/activate
    $ (.venv) pip install -r requirements.txt
    
    # Install uWSGI or any other WSGI server
    $ (.venv)  pip install uwsgi
    
    # Run uWSGI (this will vary based on your WSGI server of choice)
    $ (.venv)  uwsgi -M -T --threads 2 -p 2 --manage-script-name \
        --module giftless.wsgi_entrypoint --callable app --http 127.0.0.1:8080
    
Note that for non-production use you may avoid using a WSGI server and rely
on Flask's built in development server. This should **never** be done in a
production environment:

    $ (.venv) ./flask-develop.sh

### Configuration

You can configure Giftless using a YAML file who's path is provided 
at runtime as an environment variable:

    $ cat giftless.yaml
    
    TRANSFER_ADAPTERS:
      basic:
        factory: giftless.transfer.basic_external:factory
        options:
          storage_class: ..storage.azure:AzureBlobsStorage
          storage_options:
            connection_string: GetYourAzureConnectionStringAndPutItHere==
            container_name: lfs-storage
            path_prefix: large-files

    $ export GIFTLESS_CONFIG_FILE=giftless.yaml
    
    # Run uWSGI in HTTP mode on port 8080
    $ uwsgi -M -T --threads 2 -p 2 --manage-script-name \
        --module giftless.wsgi_entrypoint --callable app --http 127.0.0.1:8080

See `giftless/config.py` for some default configuration options. 

Development
-----------
`giftless` is based on Flask, with the following additional libraries:

* [Flask Classful](http://flask-classful.teracy.org/) for simplifying API 
endpoint implementation with Flask
* [Marshmallow](https://marshmallow.readthedocs.io/en/stable/) for 
input / output serialization and validation
* [flask-jwt-simple](https://flask-jwt-simple.readthedocs.io/en/latest/) for 
handling JWT tokens
* [figcan](https://github.com/shoppimon/figcan) for configuration handling

You must have Python 3.7 and newer set up to run or develop `giftless`.

### Code Style
We use the following tools and standards to write `giftless` code:
* `flake8` to check your Python code for PEP8 compliance
* `import` statements are checked by `isort` and should be  organized 
accordingly 
* Type checking is done using `mypy`

Maximum line length is set to 120 characters. 

### Setting up a Virtual Environment
You should develop `giftless` in a virtual environment. We use [`pip-tools`](2)
to manage both development and runtime dependencies. 

The following snippet is an example of how to set up your virtual environment
for development:

    $ python3 -m venv .venv
    $ . .venv/bin/activate
    
    (.venv) $ pip install -r dev-requirements.txt
    (.venv) $ pip-sync dev-requirements.txt requirements.txt

### Running tests
Once in a virtual environment, you can simply run `make test` to run all tests
and code style checks:

    $ make test

We use `pytest` for Python unit testsing. 

In addition, simple functions can specify some `doctest` style tests in the
function docstring. These tests will be tested automatically when unit tests
are executed. 
 
### Building a Docker image
Simply run `make docker` to build a `uWSGI` wrapped Docker image for `giftless`.
The image will be named `datopian/giftless:latest` by default. You can change 
it, for example:

    $ make docker DOCKER_REPO=mycompany DOCKER_IMAGE_TAG=1.2.3

Will build a Docekr image tagged `mycompany/giftless:1.2.3`. 
 
License
-------
Copyright (C) 2020, Viderum, Inc. 

Giftless is free / open source software and is distributed under the terms of 
the MIT license. See [LICENSE](LICENSE) for details.  


 [1]: https://git-lfs.github.com/
