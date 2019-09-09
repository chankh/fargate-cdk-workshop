---
title: "Cluster"
weight: 11
---

A [cluster](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_clusters.html) is a grouping of containerized tasks in Fargate that exist within a single [AWS Region](https://aws.amazon.com/about-aws/global-infrastructure/), but can span multiple [Availability Zones](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html). In Fargate, a cluster can be thought of as simply a namespace that also manages the underlying compute and container orchestration.

{{% notice tip %}}
Creating separate clusters for your environments is a general best practice (e.g., dev, stage, prod). The Fargate control plane/cluster is free, you only pay for the compute.
{{% /notice %}}
