# Docker 

* `docker build -t friendlyhello .`  # Create image using this directory's Dockerfile
* `docker run -p 4000:80 friendlyhello`  # Run "friendlyname" mapping port 4000 to 80
* `docker run -d -p 4000:80 friendlyhello`         # Same thing, but in detached mode
* `docker container ls`                                # List all running containers
* `docker container ls -a`            # List all containers, even those not running
* `docker container stop <hash>`           # Gracefully stop the specified container
* `docker container kill <hash>`         # Force shutdown of the specified container
* `docker container rm <hash>`        # Remove specified container from this machine
* `docker container rm $(docker container ls -a -q)`         # Remove all containers
* `docker image ls -a`                             # List all images on this machine
* `docker image rm <image id>`            # Remove specified image from this machine
* `docker image rm $(docker image ls -a -q)`   # Remove all images from this machine
* `docker login`             # Log in this CLI session using your Docker credentials
* `docker tag <image> username/repository:tag`  # Tag <image> for upload to registry
* `docker push username/repository:tag`            # Upload tagged image to registry
* `docker run username/repository:tag`                   # Run image from a registry


## 1 - Orientation

Commands in this section:
* `docker run hello-world` - runs a docker image
* `docker image ls` - lists all the docker images on your computer
* `docker container ls` - lists the running containers (`-a` lists all containers, running or stopped)


An *image is an executable package* that includes everything needed to run an application - the code, a runtime, libraries, environment variables, and configuration files.

A *container is a runtime instance of an image* - what the image becomes in memory when executed (that is, an image with state, or a user process). You can see a list of your running containers with the command, docker ps, just as you would in Linux.

You can run a simple docker image like this:

* `$ docker run hello-world`

This will first look locally and then at the docker registry for an image called hello world. It will then run this image (i.e. make a container).

## 2 - Containers

Commands in this section:
* `docker build -t <name> .` - builds a docker image and gives it a name
* `docker run -p 4000:80 <name>` - runs the image (as a container), and exposes the container's port 80. This is then mapped to port 4000 of our localhost, so we have to visit 'http://localhost:4000'
* `docker run -d -p 4000:80 <name>` - runs the container in detached state
* `docker container stop <container-id>` stops a currently running container
* `docker login` login to the docker registry
* `docker tag image <username>/<repository>:<tag>` used like this: `docker tag <image> johnptmcdonald/get_started:part_2`
* `docker push <username>/<repository>:<tag>` pushes up an image to the docker registry. It's used like this: `docker push johnptmcdonald/get_started:part_2`

Docker apps are built in a certain way. We start with *containers*. Above this level is *services*, which defines how containers behave in production. Above that we have the *stack*, which defines the interactions of all the services.

The images that make a runtime are defined in the *Dockerfile*, which says exactly what should go on inside the environment in the container. Access to resources like networking interfaces or disk drives is virtualized and isolated from the rest of your system, so you need to map ports to the outside world. You also need to be specific about what files from your system you would like copied into the container.

We can build a new image from a base image with the docker build command like this (`-t` gives the image a name tag, and the `.` means use the Dockerfile. The version tag is automatically set to `latest`):

`$ docker build -t friendlyhello .`

We can now check this image with our trusty docker image command:
`$ docker image ls`

and then run it as a container with:
`$ docker run -p 4000:80 friendlyhello` 
`$ docker run -d -p 4000:80 friendlyhello` (in detached state)

We can tag it and then push it up to docker with:
`$ docker tag friendlyhello johnptmcdonald/get_started:part_2` 
`$ docker push johnptmcdonald/get_started:part_2` 

From now on, anyone can always run this container simply by runnign this:
`$ docker run -p 4000:80 johnptmcdonald/get_started:part_2`


## 3 - Services
In this part we scale our application and enable load-balancing. Services are really just “containers in production.” A service only runs one image, but it codifies the way that image runs — what ports it should use, how many replicas of the container should run so the service has the capacity it needs, and so on. Scaling a service changes the number of container instances running that piece of software, assigning more computing resources to the service in the process.

A *docker-compose.yml* file is a YAML file that defines how Docker containers should behave in production. It looks like this:

```
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: johnptmcdonald/get_started:part_2
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:
```

This basically means we will pull the image from `johnptmcdonald/get_started:part_2`, and run 5 containers as a web service with each one using max 10% of a single core of CPU time. We map 4000 on the host to 80 on the web service, and these containers will share port 80 via a load-balanced network called `webnet`.

We then run 
`$ docker swarm init` 
`$ docker stack deploy -c docker-compose.yml getstartedlab`, where getstartedlab is the nae of the app






