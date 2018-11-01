---
title: "Installation on Docker"
icon: "fas fa-info-circle"
layout: "installationDocumentationTemp"
isInstallationDocumentation: true
menuIndex: 4
---

## Prerequisites

The ReDBox stack can be run using [Docker](https://www.docker.com/) containers. There are images published on Docker Hub for all the required components in the stack:
* [ReDBox Portal](https://hub.docker.com/r/qcifengineering/redbox-portal/)
* [ReDBox Storage](https://hub.docker.com/r/qcifengineering/redbox/)
* [MongoDB](https://hub.docker.com/r/mvertes/alpine-mongo/)

As well as some optional components:
* [Mint](https://hub.docker.com/r/qcifengineering/mint/)
* [Nginx](https://hub.docker.com/r/nginx/)


This installation guide uses the Docker Compose tool. For instructions on how to install docker-compose are [available from Docker](https://docs.docker.com/compose/install/)

## Installation

1. Set up a [Compose file (docker-compose.yml)](https://docs.docker.com/compose/compose-file/). Here's an example but you may wish to customise parts of it to your needs including mounting volumes (See the [reference](https://docs.docker.com/compose/compose-file/) for more information)

Note: copying the example below won't preserve indentation as required, if you'd like to copy it please use the version [here](https://raw.githubusercontent.com/redbox-mint/redbox-portal/master/support/deployment-examples/docker/docker-compose.yml)
```
version: '3.1'
networks:
    main:
services:
    redboxportal:
        image: 'qcifengineering/redbox-portal:latest'
        ports:
            - '80:80'
        restart: always
        expose:
            - '80'
        environment:
            - NODE_ENV=docker
            - PORT=80
            - sails_redbox__apiKey=xxxx-xxxx-xxxx
            - sails_record__baseUrl_redbox=http://redbox:9000/redbox
            - sails_appUrl=http://localhost
        networks:
            main:
                aliases: [rbportal]
        entrypoint: '/bin/bash -c "cd /opt/redbox-portal && node app.js"'
    redbox:
        image: 'qcifengineering/redbox:2.x'
        expose:
            - '9000'
        environment:
            - RB_API_KEY=xxxx-xxxx-xxxx
        networks:
            main:
                aliases: [redbox]
        ports:
            - '9000:9000'
    mongodb:
        image: 'mvertes/alpine-mongo:latest'
        networks:
            main:
                aliases: [mongodb]
        ports:
            - '27017:27017'
```
The environment variables `sails_redbox__apiKey` and `RB_API_KEY` should be set the same and is the API key the portal uses on API requests to storage.

If you are not running this on your local machine, set the `sails_appUrl` to the URL of the server it is running on.
2. Once the docker-compose.yml has been created, you can bring up the stack by running `docker-compose up`. See the [Docker Compose command-line reference](https://docs.docker.com/compose/reference/) for other options.
