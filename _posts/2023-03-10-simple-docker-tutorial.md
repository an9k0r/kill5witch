---
title: Simple Docker Tutorial
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2023-01-15 09:00:00 +0200
categories: [Blogging]
tags: [Docker]
math: true
mermaid: true
image:
  src: 
  width: 694
  height: 515
  alt: image alternative text
---
# Intro

The reason for this post is simple. I use docker here and then and tend to forget some commands and then i might not be sure if i'll want to store the data (read: create a volume with it) or not, hence writing this post.

It about the basics how to get a docker container running, about containrs and volumes to get me/you started.

# Install Docker

Things might change in the future, so check [here](https://docs.docker.com/get-docker/) on how to install docker on your platform.

# Docker Basic Terminology 

Before diving into Docker commands, it's important to understand the terminology used in Docker. 

Here are some key terms:

- **Image**: A pre-built package that contains all the dependencies and configurations needed to run an application.
- **Container**: An instance of an image that runs in isolation from the host system and other containers.
- **Dockerfile**: A text file that contains instructions to build an image.
- **Registry**: A storage location for Docker images, such as Docker Hub.

# Pull an image: 

To get started with Docker, you'll need to pull an image from a registry. For example, to pull the official Ubuntu image, you can use the following command:

```
docker pull ubuntu
```

For ARM architecture we'd use one of the following commands (we can use other image apart from `ubuntu` or version `20.04`).
```
docker pull arm64v8/ubuntu
docker pull arm64v8/ubuntu:20.04
```

We can list images using following command:

```
docker image ls
```

# Run a container 

Once you have an image, you can use it to run a container. For example, to run a container from the Ubuntu image, use the following command (add `--name` handle if you want to add a name):

```
docker run -it ubuntu bash
```

This will start a container and open a bash shell inside it. 
The `-it` flag tells Docker to run the container in interactive mode, and the bash command starts a bash shell.

> If you want to persist data you might want to add volume but more on that below [Start a container and mount the volume:](#Start-a-container-and-mount-the-volume)

We can check running conainers using following command
```
docker container ls
```

Perhaps you want to see exited containers as well and if so add `-a` handle to the command above.

If we want to exit interactive shell and want to preserve the dockers session we can use `Ctrl+P` followed by `Ctrl+Q`.

If we want to **enter** the same container we need to use its name, in my case the container ran as `frosty_noyce`...

```
docker exec -it frosty_noyce bash
```

Docker can be **STOPed** using following command
```
docker container stop <id>
```

... or **STARTed**
```
docker container stop <id>
```

**Remove** container:

```
docker container rm <id>
```

## Modify a container: 

You can modify a container by installing packages or editing files inside it. 

For example, to install the nginx package in the container, use the following command:

```
apt-get update
apt-get install nginx
```

# Volumes - Saving Changes

There are few possibilities how to persist data from a newly created container.

## Create an Image

Once you have made changes to a container, you can create a new image from it using the docker commit command. For example, to create a new image from the modified Ubuntu container, use the following command:

```
docker commit <container-id> my-ubuntu-image
```

Replace `<container-id>` with the ID of the container you want to create an image from, and my-ubuntu-image with a name for the new image.

## Create a Volume and save data to it

Docker volumes provide a way to store data outside of a container's filesystem and make it available to multiple containers. This allows you to separate data and application code, and makes it easier to manage and backup your data.

Here's an example of how to use Docker volumes:

### Create a volume:
```
docker volume create mydata
```

This will create a new volume named mydata.

### Start a container and mount the volume

```
docker run -it -v mydata:/app/data ubuntu bash
```

This will start a container from the ubuntu image, and mount the mydata volume at the /app/data directory inside the container.

Make some changes inside the container, like creating new files or modifying existing ones, and save them to the mounted volume.

### Stop and remove the container:

```
docker stop <container-id>
docker rm <container-id>
```

Replace <container-id> with the ID of the container you want to remove.

### Start a new container and mount the same volume:

```
docker run -it -v mydata:/app/data ubuntu bash
```

This will start a new container from the ubuntu image, and mount the mydata volume at the /app/data directory inside the container. The changes you made in the previous container should now be available in this new container.

Note that Docker volumes can also be used to mount data from the host system or from other containers. For more information, see the Docker documentation on volumes: https://docs.docker.com/storage/volumes/

## Dockerfile

1. Create a new directory for your project, and create a file named Dockerfile inside it.
2. In the Dockerfile, specify the base image you want to use for your new image
   For example, if you want to use Ubuntu as the base image, you can add the following line to the Dockerfile:

```
FROM ubuntu:latest
```

3. Add any additional commands you want to run in the container to the Dockerfile. For example, if you want to install the nginx web server, you can add the following line to the Dockerfile:

```
RUN apt-get update && apt-get install -y nginx
```
4. You can also specify environment variables, work directory, or add files to the container by adding ENV, WORKDIR, and COPY commands respectively.
5. Save the Dockerfile and navigate to the directory in your terminal.
6. Build the Docker image using the following command:
```
docker build -t my-image-name .
```

The `-t` option specifies a name for the image, and the `.` at the end specifies the location of the Dockerfile.

7. After building, you can start a new container from your custom image:

```
docker run -it my-image-name bash
```

This will start a new container from the my-image-name image and run the bash command inside it.

Note: It's important to keep your Dockerfile as simple and clean as possible, and to use best practices for building images. The Docker documentation provides many useful tips and best practices for Dockerfile creation: https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

# Outro

It's impossible to cover all scenarios without writing very lenghty article, but hopefully the few commands helped you with your docker endeavours and if not, just [google](https://www.google.com/) or try asking [OpenGPT](https://openai.com/blog/chatgpt)