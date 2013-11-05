---

layout: post
title: Learning Docker Basics
date: 2013-07-20 14:05:59
author: russ
category:
hackernews: http://news.ycombinator.com/item?id=4745067
tags:
  - docker

---

Docker is awesome. If you haven't heard already, Docker is an opensource project by the team at DotCloud which seemingly gives you super fast, super easy virtualization for Linux. In reality it's a really nice layer on top of Linux Containers, otherwise known as [LXC](http://lxc.sourceforge.net/).

Ignoring the technology behind it, what can you use it for? The answer is pretty much anything that you would normally virtualize.

Here at [Rainforest QA](https://www.rainforestqa.com/) we use it to enable our customers test Linux software in a easy consistent way. With Docker, their software can be installed, run and reported on in a quick, safe and repeatable way.

## Components of Docker

Docker has a few main parts, all of which come in a single binary.

1. The deamon which waits in the background and interacts directly with LXC.
2. The command line client, which you use to interact with the docker daemon
3. The Remote API, which lets you script stuff
4. An external image hosting service, so you can share 'machines'

## Getting started

Assuming you're using ubuntu (> 12.04), it's super simple get started. Docker has a guide to get started [here](http://www.docker.io/gettingstarted/) which I've repeated below;

```
sudo apt-get install linux-image-extra-`uname -r` software-properties-common
sudo add-apt-repository ppa:dotcloud/lxc-docker
sudo apt-get update
sudo apt-get install lxc-docker
```

When the above is all done, you should be good to go!

Let's check. Running `docker ps` will list all the containers currently running, which should be none:

```
russ@storagelols:~$ docker ps
ID                  IMAGE               COMMAND             CREATED             STATUS              PORTS
```

Lets start a bash console. It should look similar to this:

```
russ@storagelols:~$ docker run -i -t ubuntu /bin/bash
Pulling repository ubuntu from https://index.docker.io/v1
Pulling image 8dbd9e392a964056420e5d58ca5cc376ef18e2de93b5cc90e868a1bbc8318c1c (precise) from ubuntu
Pulling 8dbd9e392a964056420e5d58ca5cc376ef18e2de93b5cc90e868a1bbc8318c1c metadata
Pulling 8dbd9e392a964056420e5d58ca5cc376ef18e2de93b5cc90e868a1bbc8318c1c fs layer
Downloading 58.34 MB/58.34 MB (100%)
Pulling image b750fe79269d2ec9a3c593ef05b4332b1d1a02a62b4accb2c21d589ff2f5f2dc (quantal) from ubuntu
root@34897fb610e1:/# 
```

Note, the last line says `root@34897fb610e1` - we're inside a running container. If you bring up another terminal and run `docker ps` again, you should see `34897fb610e1` listed.

```
russ@storagelols:~$ docker ps
ID                  IMAGE               COMMAND             CREATED              STATUS              PORTS
34897fb610e1        ubuntu:12.04        /bin/bash           About a minute ago   Up About a minute 
```

Cool, but not so useful.


