---
title: "Deploy our application"
weight: 40
---

Before deploying our stack, we need to bootstrap CDK. This will create a staging
S3 bucket for CDK.

```
cdk bootstrap aws://$ACCOUNT_ID/$AWS_REGION
```

Let's go ahead and deploy our stack.

```
cdk deploy
```

You will notice that `cdk deploy` not only deployed your CloudFormation stack,
but also build the docker image from your disk and pushes that to Amazon ECR.
During the deployment, all the CloudFormation events show in the console so you
can monitor the progress and inspect any failures.

Once the deployment is completed, you should see an output like this.

![output](/images/infrastructure/outputs.png)

Copy the DNS name of the load balancer from the outputs and paste that into a
new browser tab or window. That's it, your application is now running on Fargate
using an infrastructure created using AWS CDK.