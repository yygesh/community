---
title: Running a Mastodon Instance with Docker on Google Compute Engine
description: Learn to run your own instance of Mastodon on a Google Compute Engine virtual machine using Docker.
author: tswast
tags: Compute Engine, Docker, Mastodon
date_published: 2017-04-18
---
This tutorial will guide you through running a [Mastodon social network
server](https://github.com/tootsuite/mastodon) on Google Compute Engine using
Docker.

## Objectives

- Run an instance of the Mastodon social network server

## Before you begin

- Follow the [tutorial for running an NGINX reverse proxy with Docker on Google
  Compute
  Engine](https://cloud.google.com/community/tutorials/nginx-reverse-proxy-docker).

  The reverse proxy makes it easy to

  1. Get TLS/SSL certificates for HTTPS support,
  2. Run more than one web application on the same machine.

After following that tutorial, you will have a Google Cloud Platform project
with a Google Compute Engine instance running CoreOS.

## Costs

This tutorial uses billable components of Cloud Platform including

- [Google Compute Engine](https://cloud.google.com/compute/pricing)

Use the [Pricing
Calculator](https://cloud.google.com/products/calculator/#id=a55c860c-d09f-4c08-aec9-2c4de63e6f54)
to estimate the costs for your usage.

## Resizing a Compute Engine instance

Mastodon requires more memory than the amount included in an f1-micro instance.
You may need to resize the instance you created in the [running an NGINX
reverse proxy with Docker on Google Compute Engine
tutorial](https://cloud.google.com/community/tutorials/nginx-reverse-proxy-docker).

1.  Select your instance from the [instances
    page](https://console.cloud.google.com/compute/instances) of the Google
    Cloud Platform console by clicking the instance name.
1.  **Stop** the instance. Wait for it to stop, which can take up to two minutes.
1.  **Edit** the instance.
1.  Change the **Machine type** to 1 vCPU (n1-standard-1).
1.  Scroll to the bottom and click the **Save** button.
1.  **Start** the instance.

## Setting up a domain name for your Mastodon host

Just as you have done in the [reverse proxy
tutorial](https://cloud.google.com/community/tutorials/nginx-reverse-proxy-docker),
create an [A type DNS
record](https://en.wikipedia.org/wiki/List_of_DNS_record_types) pointing at the
static, external IP address of your Compute Engine instance.

This tutorial will assume you are using the domain `social.example.com`.

## Installing Docker Compose

The CoreOS image includes Docker pre-installed, but it does not include [Docker
Compose](https://docs.docker.com/compose/overview/).

Click the **SSH** button on the [instances
page](https://console.cloud.google.com/compute/instances) to open a terminal
connected to your instance.

Make a directory to hold the binary.

    mkdir $HOME/bin

Dow docker-compose -vnload the [latest Docker Compose release](https://github.com/docker/compose/releases).

    curl -L https://github.com/docker/compose/releases/download/1.12.0/docker-compose-`uname -s`-`uname -m` > $HOME/bin/docker-compose

Add executable permissions.

    chmod +x $HOME/bin/docker-compose

Add the binary to your path.

    export PATH="$HOME/bin:$PATH"

Verify the `docker-compose` command works.

    docker-compose -v

## Creating a Docker network

As discussed in the [reverse proxy
tutorial](https://cloud.google.com/community/tutorials/nginx-reverse-proxy-docker),
the `nginx-proxy` container must be on the same network as the web application
containers it proxies.

1.  Create a new Docker network.

        docker network create --driver bridge reverse-proxy
 
1.  Stop and remove your web application containers, the `nginx-proxy` container,
    and the `nginx-letsencrypt` container.
    
        docker stop my-container
        docker rm my-container
        docker stop nginx-proxy
        docker rm nginx-proxy
        docker stop nginx-letsencrypt
        docker rm nginx-letsencrypt

1.  Run the proxy and other containers, specifying the network with the
    `--net reverse-proxy` command-line parameter.

## Running Mastodon

Follow the [Mastodon Docker production
guide](https://github.com/tootsuite/documentation/blob/master/Running-Mastodon/Docker-Guide.md)
but with a few modifications to run behind a reverse proxy.

1.  Create a directory to hold the Mastodon configuration.

        mkdir $HOME/mastodon
        cd $HOME/mastodon

1.  Download the `docker-compose.yml` configuration file from the [Mastodon
    repository](https://github.com/tootsuite/mastodon).

        curl -L https://raw.githubusercontent.com/tootsuite/mastodon/master/docker-compose.yml > docker-compose.yml

1.  Download the sample `.env.production.sample` configuration file.

        curl -L https://raw.githubusercontent.com/tootsuite/mastodon/master/.env.production.sample > .env.production

### Configuring Docker Compose

1.  Modify the `docker-compose.yml` configuration file to use the pre-compiled
    [Mastodon Docker image](https://hub.docker.com/r/gargron/mastodon/). This
    tutorial uses the `vim` command to edit files, since it is included with
    CoreOS.

        vim docker-compose.yml

    Remove every line that says `build: .`.

    Find the most recent [tag of the Mastodon Docker
    image](https://hub.docker.com/r/gargron/mastodon/tags/) which as a version
    number. For example, `v1.2`. Change the image definitions to include a tag name.

        image: gargron/mastodon

    becomes

        image: gargron/mastodon:v1.2


### Populating the Environment

