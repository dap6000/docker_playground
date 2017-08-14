# LAMP Stack Local Development Container
I've had a _good enough_ process for local development for many years now. But once I started paying attention I noticed I was losing several hours of  productivity in troubleshooting and debugging my local dev environment when  something went wrong. This didn't happen often. But when it did happen it was  frustrating. And it seems like codifying the state of my dev environment in a  container would be a long term win.

I'm a developer. Not an ops person. Ops work is slow. Especially for those of us not terribly familiar with Unix and server admin stuff anyway. The good news is I'm going to try to document my process here in hopes of reinforcing some synaptic connections in my own brain as well as possibly providing useful info for others new to Docker.

# How do I use this damn thing?
Install Docker for Mac: https://www.docker.com/docker-mac (In theory any platform should work. In practice I've only tested this on a Mac.)
Download this repo as a zip file. Or clone it if you prefer. 
The `docker_playground/lamp/` directory contains the dockerfile and build context for our image.
I put mine in my home directory. Adjust the commands as needed for your location.
Change to the relevant directory. If you are paranoid like me do a sanity check with `ls`. 
```
$ cd ~/Documents/docker_playground/lamp
$ ls
dockerfile	etc		opt		readme.md	var
$ 
```
First we need to build an image. Relevant documentation here: https://docs.docker.com/engine/reference/commandline/image_build/
It makes life easier if we `tag` our image too.
```
$ docker image build --tag localdev:v1.0 .
Sending build context to Docker daemon  24.06kB
Step 1/14 : FROM ubuntu:16.04
 ---> d355ed3537e9
Step 2/14 : MAINTAINER Derek Pennycuff version 0.8
 ---> Using cache
 ---> dd7d5dc63b4e
Step 3/14 : ENV DEBIAN_FRONTEND noninteractive
...
Successfully built ab1c04df6adb
Successfully tagged localdev:v1.0
```
List your available images.
```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
localdev            v1.0                ab1c04df6adb        11 days ago         512MB
...
ubuntu              16.04               d355ed3537e9        7 weeks ago         119MB
```
Your image IDs and create date will be different of course.
Now that we have an image, we need to create a container from it.
Relevant docs: https://docs.docker.com/engine/reference/commandline/container_create/
We'll want to make use of the `--name`, `--publish`, and `--volume` options.
Naming our container makes it easier to work with without having to recreate it from scratch.
Publishing a port from the container to the host machine lets us surf to our project using a localhost:[some_port] style url.
Mounting a project as a volume lets us make edits to our source code and track changes in Git on our host machine while also making the source files available to PHP and Apache running inside the container.
```
$ docker container create --name my_craft_project --publish 9001:80 --volume ~/dev_server/Editorial-Craft:/var/www/ localdev:v1.0
e39c9200c99d4d3b70c6bf3ccccfa20cfabd788920b40f693676c25d1270a017
$
```
You can list all avialable containers. We need the `--all` flag because our new container is not running yet.
```
$ docker container ps --all
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
e39c9200c99d        localdev:v1.0       "/usr/bin/supervis..."   59 seconds ago      Created                                        my_craft_project
```
Since this is a Craft project I mounted the project root to the container's `/var/www/` directory. This is to allow the `craft` directory to exist a level up from the public `html` directory. For other projects without this requirement we may need to mount our project directory to `/var/www/html/` instead.
We can launch our new container using `docker container start…` https://docs.docker.com/engine/reference/commandline/container_start/
Sometimes it is useful to start a container with the `--attach` option so we can keep an eye on it's `STDOUT`/`STDERR`
```
$ docker container start --attach my_craft_project
2017-08-14 21:28:56,973 CRIT Supervisor running as root (no user in config file)
2017-08-14 21:28:56,978 INFO supervisord started with pid 1
2017-08-14 21:28:57,982 INFO spawned: 'apache' with pid 9
2017-08-14 21:28:59,156 INFO success: apache entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
...
```
I can now surf to the project using http://localhost:9001/