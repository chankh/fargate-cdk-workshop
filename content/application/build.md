---
title: "Build"
date: 2018-08-07T12:37:34-07:00
weight: 11
---

In this step, we are going to build the Docker image for the PetClinic
application and test it locally in our Cloud9 environment.

##### Create a tag

A [Docker tag](https://docs.docker.com/engine/reference/commandline/tag/)
represents a specific version of a container image. In this workshop, we are
going to use a combination of the date and the current Git SHA.

```
cd ~/environment/spring-petclinic
export TAG=$(date +%Y-%m-%d.%H.%M.%S).$(git rev-parse HEAD | head -c 8)
```

##### Build Docker Image

To build the docker image, we need to define a `Dockerfile`. We have prepared 
one for you, copy this to the project directory and start building.

```
cp ~/environment/fargate-cicd-workshop/static/assets/Dockerfile .
docker build --tag spring-petclinic:$TAG .
```

##### Run it locally

Test run your docker image locally.

```
docker run -it spring-petclinic:$TAG
```

Once it is started, you can preview the application using **Preview** > 
**Preview Running Application** from the menu.

When you are ready to proceed, press `ctrl-c` to stop the application and move 
on to the next step.