---
path: "/docs/deploying/docker"
title: Deploy to the Docker platform
---

#Deploy to the Docker platform

Docker provides tooling and a platform to manage the lifecycle of lightweight virtualization packages, known as containers. In this guide, we will cover the steps necessary to Dockerize a Kitura application. This includes how to build the Docker image and how to create the container instance from that image.

A full description of the Docker platform can be found on the Docker [website](https://docs.docker.com/get-started/), including installation instructions. This guide is based on the two Dockerfiles generated by the Kitura CLI's [kitura init](../getting-started/create-server-cli) command.

---

##Step 1: Build compile image

As the Swift compiler is large and not required for running compiled Kitura applications, the Docker images for application compilation and runtime are separated into a build image and a run image. The `kitura init` command supports this two step build process by generating two Dockerfiles for your Kitura project:

- `Dockerfile-tools`: building this Dockerfile produces a Docker image containing everything needed to compile your Kitura application. Running the image will compile your Kitura application.
- `Dockerfile`: building this Dockerfile produces a Docker image containing your compiled application. Running the image will run your application.

We first need to build the compile image:

```
docker build -t myapp-build -f Dockerfile-tools .
```

The above command builds a Docker image called `"myapp-build"` using the generated `Dockerfile-tools` file. We will use this image to compile our Kitura application.

---

##Step 2: Compile application code

Using the image we've just built, we can compile the application code with the following command:

```
docker run -v $PWD:/swift-project -w /swift-project myapp-build /swift-utils/tools-utils.sh build release
```

The Docker run command takes the `'-v'` option which maps the current working directory on the host to the `'/swift-project'` directory on the Docker container. The `-w` option sets working directory inside the container. We then tell Docker to create a container using the `myapp-build` tag name from the image we created earlier. Finally we invoke the utility script that is located in the container at `/swift-utils/tools-utils.sh` to build in release mode. The `tools-utils.sh` file can be viewed [here](https://github.com/IBM-Swift/swift-ubuntu-docker/blob/master/utils/tools-utils.sh).

---

##Step 3: Build the run image

Now that we have compiled our Kitura application we can now build the Docker runtime image, with the following command.

```
docker build -t myapp-run .
```

This command builds a Docker image which is tagged `"myapp-run"` using the generated `Dockerfile` file. The newly built Docker image contains your compiled Kitura application and the necessary Swift runtime libraries.

---

##Step 4: Check the image is available

Docker provides a command to list your local images:

```
docker image ls
```

This command lists the Docker images available in your local repository, which will include your recently built image tagged `"myapp-run"`.

---

##Step 5: Start the Docker container

Now that we have an image that contains our Kitura application, we just need to run an instance of the image:

```
docker run -p 8080:8080 -it myapp-run
```

The above Docker run command uses the `-p` flag to publish port 8080 to the host interfaces. It then runs a container from the `myapp-run` image in interactive mode. You can see your container is running with the following command:

```
docker container ps -a
```

Since we exposed our ports, we should then be able to navigate to: http://localhost:8080 to view our server.