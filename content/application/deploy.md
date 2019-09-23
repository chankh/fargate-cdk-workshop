---
title: "Deploy"
weight: 13
---

Now that we have successfully built our first container, we are going to use the
`fargate` command to deploy the application to AWS Fargate.

#### Download and install the latest Fargate CLI release
```
export PATH=$PATH:$HOME/go/bin
go get -u github.com/jpignata/fargate
```

#### Verify the binary
```
fargate --version
```

Your output should resemble:
![verify](/images/application/fargate_cli_verify.png)

#### Deploy to AWS Fargate

First we are going to create a load-balancer.
```
fargate lb create web --port 80
```

And this is going to create an ALB which can be use to front the services
running on AWS Fargate.

Next we need to create a service, give it a name _spring-petclinic_, attach it
to the load balancer that we have just created, and tell the load balancer that
the container listens on port 8080.
```
fargate service create spring-petclinic --lb web --port http:8080
```

This `fargate` CLI simplifies a lot of the heavylifting for us. It creates a
private Docker image repository on [Amazon Elastic Container Registry](https://aws.amazon.com/ecr/)
(Amazon ECR), builds the container image from the project directory, pushes the
image to Amazon ECR and runs the container as a service on AWS Fargate. If you
go into the [Amazon ECS console](https://console.aws.amazon.com/ecs) now, you
should see a cluster named "fargate", and there is 1 service running.

![fargate-cluster](/images/application/fargate_cluster.png)

You can also see these information using the `fargate` CLI. 
```
fargate service info spring-petclinic
```

Here you can see your service is up and running, with a bunch of information
like the load balancer.

![fargate-info](/images/application/fargate_service_info.png)

Open the load balancer URL in a new browser window and you should see the
PetClinic application.

![petclinic](/images/application/preview.png)
