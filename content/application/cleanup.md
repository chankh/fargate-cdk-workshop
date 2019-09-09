---
title: "Cleanup"
weight: 15
---

If you want to keep your application running, you can do so, but this is the end
of this section. In order to clean up and tear down the Fargate cluster, we need
to delete the service.

```
fargate service scale spring-petclinic 0
fargate service destroy spring-petclinic
```

{{% notice tip %}}
If the operation times out, run the command again.
{{% /notice %}}

Next, delete the load balancer.

```
fargate lb destroy web
```

Finally delete the Amazon ECS cluster and Amazon ECR repository.
```
aws ecs delete-cluster --cluster fargate
aws ecr delete-repository --force --repository-name spring-petclinic
```
