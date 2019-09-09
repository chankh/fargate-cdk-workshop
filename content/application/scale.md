---
title: "Scale"
weight: 14
---

At this point, our cluster and service are up and running. Our application
turned out to be incredibly popular and we need to scale the number of
containers we are running in the cluster.

We are going to going to use the `fargate` CLI to accomplish the scaling:

```
fargate service scale spring-petclinic 2
```

If you run the **service info** command again, you will see that we now have two
containers running.

```
fargate service info spring-petclinic
```

The output should now resemble:

![scale out](/images/application/fargate_service_info_2.png)

You can see that we have scaled the service to two and there are two containers
running.

