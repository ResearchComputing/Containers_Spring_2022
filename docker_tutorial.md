# Docker Tutorial

## What is Docker?

Docker is an open source containerization platform for developing, shipping, and running applications. Docker enables you to separate applications from infrastructure so you can use software quickly on any platform. 

## A case for Docker in Research

Scientific Software is often challenging to work with:
- Difficult installations
- Low support from the developers
- Often *very* outdated
- Complex dependency trees

Because of this it's often desired for a software to be repeatable and accurate. __*But (you may say) installs are only done once, why should I care about reproducible applications?*__

Research is collaborative, team members work together to get projects done, reproducibility ensures all members of a team can provide productivity towards a project. Research is correcting over time (Academic reviews are common), and is inevitably *hard*. Research is continuous and someone may wish to reproduce your work on a different system. What happens when you move on from a project? Do you take the work with you to a different system or leave it for someone else to figure out? You get the idea - there are *many* reasons to make software repeatable and accurate in research.

There are other options to help keep research software repeatable and accurate: virtual environments (python, anaconda), other HPC managements systems (spack) - but do they *really* enable accurate reproducibility? What about incorrect installs, different hardware or OS, performance? There are a whole host of issues that plague developers and researchers. Containers are one answer to these issues.


## So what is a containers?

A container is a sandboxed process on your machine that is isolated from all other processes on the host machine. That isolation leverages kernel namespaces and cgroups, features that have been in Linux for a long time. Docker has worked to make these capabilities approachable and easy to use. To summarize, a container:

- Is a runnable instance of an image. You can create, start, stop, move, or delete a container
- Can be run on local machines, virtual machines or deployed to the cloud
- Is portable (can be run on any OS)
- Containers are isolated from each other and run their own software, binaries, and configurations

## Installation

You can download and install Docker on multiple platforms. Refer to the following section and choose the best installation path for you:
- Various [Linux](https://docs.docker.com/engine/install/) distributions
> Note: If you don’t want to preface the docker command with `sudo` follow steps for your installation to add your user to a docker group (e.g. for [Linux](https://docs.docker.com/engine/install/linux-postinstall/)).

For Mac and windows you have 2 options:

1. Install a Linux VM with virtual box (or other virtualization platform) then install Docker on Linux (above) or
2. Install Docker Desktop (Native Applications) 

- [Mac](https://docs.docker.com/desktop/mac/install/) (both Apple and Intel chips)
- [Windows](https://docs.docker.com/desktop/windows/install/)


If you installed Docker correctly, you should be able to check the version like so:
```bash
$ docker --version
```

## Basic Components of Docker

Docker most simply can be broken down into 3 main components:

1. Docker File
	- A Docker file is just that, a file. The Docker file acts like DNA, code that tells docker how to build an image.
2. Image
	- Snapshot of your software along with all of its dependencies (down to OS level)
	- Immutable and can be used to spin up multiple containers
3. Container
	- Running instances of images that are isolated and have their own sets of environments and processes
	- Actual software running in the real world

## Hello World

To test our installation and go over a simple docker example run the following command:

```bash
$ sudo docker run hello-world
```

You should see a message that starts with "Hello from Docker!". But what actually happened here?


1. We used the `docker run <image-id>` command to tell docker what image to run. The `docker run` command is used to run a continer from an image.
2. Docker returned a message telling us that it was `Unable to find image 'hello-world:latest' locally.` This is because we run it for the first time. 
> The ":latest" part here is the image tag. We can tag an image with whatever we want (dev, v0.1.1, my-tag, etc...), but the default tag is latest.
3. Since the image was not found locally, docker had to get it from somewhere, and we got it from Docker Hub. Docker Hub in concept is similar to GitHub, but instead of hosting repositories, it hosts images, and often those images are open sourced. But there are also private images.

## Docker Commands

There are many *MANY* Docker commands, too many to cover here, but they generally come in the following form:
```bash
$ docker <sub-command> <flags> <target/command>
```
Examples:
```bash
$ docker run -it myimage
$ docker container ls
$ docker image prune
```

| Container Commands| Description |
| --- | ----------- |
| `docker container ls` | List docker containers currently running |
| `docker container rm <container>` | Remove (an) container(s) |
| `docker rm <container>` | |
| `docker container prune` | Remove all stopped containers |


| Image Commands| Description |
| --- | ----------- |
| `docker image ls` | List docker images stored in cache |
| `docker image rm <image>` | Remove (an) image(s) |
| `docker rmi <image>` | |
| `docker image prune` | Remove unused images |

| Other Commands| Description |
| --- | ----------- |
| `docker info` | Shows Docker system-wide information |
| `docker inspect <docker-object>` | Shows low-level information about an object |
| `docker config <sub-command>` | Manage docker configurations |
| `docker stats <container>` | Shows container resource usage |
| `docker top <container>` | Shows running processes of a container |
| `docker version` | Shows docker version information |

## Launching a Docker Container 

Next, we'll launch a linux ubuntu base container interactively by invoking a shell with the `bash` command which will allow us to poke around *inside* of the container:

```bash
$ docker run -it ubuntu bash
```

The `-it` allows us to run the container interactively and connect it to a pseudoterminal:
```bash
root@<container-id>:/$ ls
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

We can run Ubuntu commands here, updating the apt repo and installing libs/dependencies:
```bash
root@<container-id>:/$ apt-get update
...
root@<container-id>:/$ apt-get install vim -y
...
```

You will notice that if you exit the container and try to re-run the container from the image, the changes you made are gone. This is because containers are ephemeral, they run in host memory so those changes are gone once the container is exited. So how do we persist dat in Docker containers? We'll explore this in the next demo briefly, then go into more depth in demo 2. 


## Demo 1: Gromacs

Next we'll take a look at a popular reserch program which has been recompiled and reinstalled locally on CURC resources inumerable times. GROMACS is a molecular dynamics application that can often be a complex and challenging installation for the average user, and run only on Linux and Mac. GROMACS has *very* dense documentation, and it requires compilation which some users may not be comfortable doing. An alternative is using the official GROMACS Docker container:

From Dockerhub (which we'll talk about more below) we can pull and run the GROMACS container, invoking help and running the container interactively:

```bash
$ docker run gromacs/gromacs gmx help commands
$ docker run -it gromacs/gromacs
```

If you're intersted in GROMACS, you can check out the following Docker GROMACS example. The following highlights a GROMACS workflow using an example using pdb2gmx from the [tutorial KALP15 in DPPC](http://www.mdtutorials.com/gmx/membrane_protein/01_pdb2gmx.html):
```bash
$ mkdir $HOME/data ; cd $HOME/data
$ wget http://www.mdtutorials.com/gmx/membrane_protein/Files/KALP-15_princ.pdb
$ docker run -v $HOME/data:/data -w /data -it gromacs/gromacs gmx pdb2gmx -f KALP-15_princ.pdb -o KALP-15_processed.gro -ignh -ter -water spc
```
There's a lot going on here, but trust me, we'll explore all of these concepts more in later Demos. Here we create a directory on our local machine called `data`, download data to it, then run GROMACS from the container by binding the directory within our local machine to the container (we'll explore this more below). See the [KALP15 in DPPC tutorial](http://www.mdtutorials.com/gmx/membrane_protein/01_pdb2gmx.html) for more information on what options to choose from GROMACS when the container runs.

## Building a Docker image from a Dockerfile

You can pull Docker images like a pro! But what if there isn't a Docker image that suits your needs? Or what if your team wants to port their application to a container for easy shipping/running? The answer is to build a Docker image from scratch. As mentioned above, you can build a Docker image from a Dockerfile (remember: Dockerfile = Docker DNA).

A Dockerfile is simply a text file that contains instructions to build and setup a default image, it contains:
- A template or "starting" image (required)
- Commands to install dependencies and libraries
- Other configs (ports, volumes, environment variables)
- The command the container will execute

These come in the form of __KEY__ __value__ pairs (notice the key is capitalized):
- __FROM__
	- start __FROM__ a “template” image
	- base image gets pulled down from cloud
- __RUN__
	- to __RUN__ terminal commands
	- install dependencies
- __WORKDIR__, __ENV__, etc…
- __CMD__ (or __ENTRYPOINT__)
	- execute a default __CMD__


Example Dockerfile (must be named `Dockerfile` with no extensions):
```bash
FROM ubuntu:18.04

RUN apt-get update && apt-get install -y \
	gcc \
	nano \
	vim

WORKDIR /target

CMD bash
```



Once we set up our Dockerfile we can use the `docker build` command to build the image from the Dockerfile: 
```bash
docker build –t <image-name> .
```
> Note: The `.` in the command points to the location of the `Dockerfile`, "this directory" in this case. The `-t` flag allows you to name or "tag" your image.

## Demo 2: Ubuntu with GCC

For this first example we will build a custom Ubuntu image that will provide a location to run the GNU Compiler Collection.

Dockerfile provided in the course GitHub repo at: `$CONTAINER_ROOT/dockerdemo/ubuntu-gcc`

We need to build the container:
1. Navigate to the directory:
```bash
$ cd $CONTAINER_ROOT/dockerdemo/ubuntu-gcc
```
2. Build the image with:
```bash
$ docker build -t test-gcc .
```
Run image as container:
```bash
$ docker run -it test-gcc
```

Once we're in the container we see that we are put into `/target` which is the `WORKDIR` that was set in the Dockerfile. We can even create a test file here:
```bash
$ touch test.txt && echo "test contents" > test.txt
```

But if we exit and try to re-run the container we will again notice those changes are gone. In Demo 1 we had you run a couple of commands which created a directory on your host machine, download some data, and then run a *very* long `docker run` command. In this `docker run` command there was a `-v` flag which allows you to mount a filesystem to your container.

There are 2 kinds of mounts:
1. __Bind Mounts__: Allows the docker container to access files on the host OS. You choose the host's source directory, files in the directory will be moved to the container's working directory.
	- Source Directory: Directory on the host system. Never within a container.
	- Target Directory: Directory in the Docker Container. Never on the host system.
	- You specify these directories with a flag set within the docker run command:
	```bash
	docker run -v <source-dir>:<target-dir> <image>
	```
	- Bind mounts are often easiest to conceptualize because we're working with filesystems on our own machine we can explore. Bind mounts do come with their own issues however, i.e. if you move the directories or change permissions. These problems can be overcome using volume mounts.
<br></br>
2. __Volume Mounts__: Generally the same concept, but instead of pointing the "source directory" to an existing directory on the host, volumes are stored within docker cache. This location still exists within your local host but is burried in the Docker installation location (abstracted from us).
	- You create Docker volumes in your terminal and link your volume directory (similarly linked through the docker run command):
	```bash
	 $ docker volume create <volume-name>
	 $ docker run -v <volume-name>:<target-dir> <image> 
	```

	- Volumes are great options if you only intend to access this data within Docker, but can be difficult to use if you want the data accessable outside.

	- You may ask “where is Docker actually storing my data when I use a named volume?” If you want to know, you can use the docker volume inspect command.
	```bash
	$ docker volume create test
	test
	$ docker volume inspect test
	[
			{
					"CreatedAt": "2022-04-06T12:01:41-06:00",
					"Driver": "local",
					"Labels": {},
					"Mountpoint": "/var/lib/docker/volumes/test/_data",
					"Name": "test",
					"Options": {},
					"Scope": "local"
			}
	]
	```
	- The Mountpoint is the actual location on the disk where the data is stored. Note that on most machines, you will need to have root access to access this directory from the host. But, that’s where it is!

We're going to use bind mounts so we can access the data directly. In the directory where our Dockerfile lives, use this command (all on one line):
```bash
$ docker run -it -v $(pwd)/source:/target test-gcc
```

Now that we have mounted the local `/source` directory to the `/target` in our container we can list out the contents, then perform the compilation:
```bash
$ ls
hello.c
$ gcc hello.c -o hello
$ ./hello
Hello, world!
```

We can even exit the container and find the new executable file (`hello`) in the `/source` directory on our host machine. Next time we run this container with the `-v` flag to bind mout this same `/source` directory we will find all of the same files.

## Demo 3: NCL Container

This next example we will be building a Docker image that will run the NCAR Command Language (NCL), this is a fairly niche software that is difficult to install, it is provided by the NCAR Docker account, the Dockerfile is located in the Course Repo sourcecode:

- Dockerfile provided at: `$CONTAINER_ROOT/dockerdemo/ncl`
- Build the Dockerfile and name the image: "ncl-demo"
```bash
$ docker build -t ncl-demo .
```
- Run and test "ncl-demo":
```bash
$ docker run -v $(pwd)/source:/target ncl-demo ncl test.ncl
```

This command runs the ncl container, uses the `ncl` executable on the `test.ncl` file within our `/source` directory and produces a PDF file which we can open on our local machine. You can open up the PDF in your file explorer or in a browser from your terminal with (firefox example):
```bash
$ firefox <filename>.pdf
```

## Demo 4: Docker Compose

The final concept we'll cover is Docker Compose. Docker Compose is a tool that was developed to help define and share multi-container applications. With Compose, we can create a YAML file to define the services and with a single command, can spin everything up or tear it all down quickly and easily.

The big advantage of using Compose is you can define your application stack in a file, keep it at the root of your project repo (it’s now version controlled), and easily enable someone else to contribute to your project. Someone would only need to clone your repo and start the compose app.

Docker compose comes bundled with Docker Desktop, but if you are running non-desktop (i.e. on Linux) follow the [official docker-compose install instructions.](https://docs.docker.com/compose/install/) 

There are (currently) 3 versions of docker compose which have their own syntax and features. V3 is the newest and the version we will be using. The general structure of a `docker-compose.yml` file follows:

```bash
version: '3'

services:
  <image-name>:
    image: <image>
    volumes:
      - <host-directory>:<container-directory>
  <image2-name>:
    build: .
    volumes:
      - <host-directory>:<container-directory>
```

Note you can add many more options including ports mapping, application links, networks, etc to the docker compose file, but we keep it simple here. You have the options to either use pre-existing images (i.e. from Dockerhub or if you have images built in your Docker cache) or build from a Dockerfile. In the following example we have a simple python script in our source directory which we copy into the image (using the COPY command in the Dockerfile). We could build this by hand each time we want to make changes, or by writing the build commands in the `docker-compose.yml` once we can just run the command:
```bash
$ docker-compose up 
```
To build and run all containers specified in the docker compose file. To run the docker compose:

1. Navigate to the directory:
```bash
$ cd $CONTAINER_ROOT/dockerdemo/docker-compose
```
2. Run the docker compose file:
```bash
$ docker-compose up
```
Done! Simple as that. Docker compose makes it extremely easy to develop and play around with containers as you can run then with `docker-compose up` and tear everything back down with `docker-compose down` which takes care of container and image cleanup. Docker compose is even more useful for creating application stacks which can be connected easily; think of a python application that pushes data to a database. In this case you could define both your own python application (build it in the docker-compose file) *and* an official database image ([mysql](https://hub.docker.com/_/mysql) for example) in a single docker compose file and bring them up with `docker-compose up` simultaneously. This is beyond the scope of this particular tutorial, but RC does have another tutorial on just this (check it out [here](https://github.com/ResearchComputing/CUmulus_tutorials/tree/main/tutorial2) if you're intersted).

There are many *__MANY__* other topics in Docker and Docker compose to explore, but this is a good starting point for Docker in research. 

### More resources

- [Official Docker Docs](https://docs.docker.com/get-docker/)
- [Official Docker Tutorial](https://docs.docker.com/get-started/)
- [Docker Hub](https://hub.docker.com/)
- [Docker compose docs](https://docs.docker.com/compose/)
- [Massive curated list of Docker resources](https://project-awesome.org/veggiemonk/awesome-docker)


