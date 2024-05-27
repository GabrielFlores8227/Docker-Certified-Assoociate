## Chapter 1 - Image Creation, Management, and Registry

### Docker Images

A Docker image is a file containing the code and components needed to run software in a container.
Containers and images use a layered file system. Each layer contains only the differences from the 
previous layer. The image consists of one or more read-only layers, while the container adds one 
additionall writable layer.

The layered file system allows multiple images and containers to share the same layers. This 
results in:

  - Smaller overall storage footprint.

  - Faster image transfers.

  - Faster image build.

### Dockerfiles

If you want to create your own images. you can do so with a Dockerfile. A Dockerfile is a set of instructions
which are used to construct a Docker image. These instructions are called directives.

- `ADD`: Add local or remote files and directories.
  
- `ARG`: Use build-time variables.
  
- `CMD`: Specify default commands.
  
- `COPY`: Copy files and directories.
  
- `ENTRYPOINT`: Specify default executable.
  
- `ENV`: Set environment variables.
  
- `EXPOSE`: Describe which ports your application is listening on.
  
- `FROM`: Create a new build stage from a base image.
  
- `HEALTHCHECK`: Check a container's health on startup.
  
- `LABEL`: Add metadata to an image.
  
- `MAINTAINER`: Specify the author of an image.
  
- `ONBUILD`: Specify instructions for when the image is used in a build.
  
- `RUN`: Execute build commands.
  
- `SHELL`: Set the default shell of an image.
  
- `STOPSIGNAL`: Specify the system call signal for exiting a container.
  
- `USER`: Set user and group ID.
  
- `VOLUME`: Create volume mounts.
  
- `WORKDIR`: Change working directory.

### Efficient Docker Images

When working with Docker in real world, it is important to create Docker images that are as
efficient as possible. This means that they are as small as possible and result in ephemeral 
containers that can be started, stopped, and destroyed easily.

Docker supports the ability to perform multi-stage builds. Multi-stage builds have more than
one `FROM` diirective in the Dockerfile, with each `FROM` directive starting a new stage.
Each stage begins a completely new set of file system layers, allowing you to
selectively copy only the files you need from previous layers.

Here is an example of an efficient Dockerfile utilizing multi-stage builds:

```Dockerfile
# Use the official Golang image as the base image for the compiler stage
FROM golang:latest AS compiler

# Set the working directory within the container for the compiler stage
WORKDIR /compiler

# Copy the source code (helloworld.go) to the working directory
COPY ./helloworld.go .

# Initialize Go module to manage dependencies
RUN go mod init helloworld

# Build the Go application for Linux (GOOS=linux) with CGO disabled (-installsuffix cgo)
# and output the binary as 'helloworld' in the working directory
RUN GOOS=linux go build -a -installsuffix cgo -o helloworld .

# Start a new stage using the lightweight Alpine Linux as the base image
FROM alpine:latest

# Set the working directory within the container for the runtime stage
WORKDIR /root

# Copy the built executable 'helloworld' from the compiler stage to the working directory of the runtime stage
COPY --from=compiler /compiler/helloworld ./

# Define the command to run the executable when the container starts
CMD ["./helloworld"]
```

### Managing Images

Download an image from a remote registry to the local machine:

```bash
docker image pull <IMAGE:TAG>
```

List the layers used to build an image:

```bash
docker image history <IMAGE:TAG>
```

List images: 

```bash
docker image ls
```

Add the -a flag to include intermediate images:

```bash
docker image ls -a
```

Get detailed information about an image:

```bash
docker image inspect <IMAGE>
```

Use `--format` flag to get only a subset of the information (use Go templates):

```bash
docker image inspect <IMAGE> --format TEMPLATE
```

These commands can both be used to delete an image. Note that if an image has other
tags, they must be deleted first:

```bash
docker image rm <IMAGE_ID>
```

```bash
docker rmi <IMAGE_ID>
```

Delete unused images from the system:

```bash
docker image prune
```

### Flattening an Image

Sometimes, images with fewer layers can perform better. In a few cases, you may want to take an image
with many layers and flatten them into a single layer.

Docker does not provide an official method for doing this, but you can accomplish it by doing the 
following:

```bash
docker run --name <CONTAINER_NAME> <IMAGE>
```

```bash
docker export <CONTAINER_NAME> > <CONTAINER_NAME>.tar
```

```bash
cat <CONTAINER_NAME>.tar | docker import - flat:latest
```

### Docker Registries

A Docker Registry is responsible for storing and distributing Docker images.

You can also create your own registries using Docker's open source registry software, or Docker Trusted Registry,
the non-free enterprise solution.

To create a basic registry, simply run a container using registry image and publish port 5000.

```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

## Chapter 2 - Orchestration

### Docker Swarm

Docker includesa feature called swarm mode, which allows you to build a distributed cluster of docker machines to 
run your containers.

Docker swarm provides many useful features, and can help facilitate orchestration, high-availability, and scaling.

### Configuring a Swarm Manager

Setting up a new swarm is relatively simple. All we have to do is create the first swarm manager:

- Install Docker CE on the Swarm Manager server.

- Initialize the swarm with `docker swarm init --advertise-addr <IP>`.

- If necessary, leave the swarm with `docker swarm leave`.

Once the swarm is initialized, you can see some info about the swarm with `docker info`.

You can list the current node in the swarm with `docker node ls`
