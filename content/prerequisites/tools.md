---
title: "Install CLI Tools"
chapter: false
weight: 22
---

The [AWS Command Line Interface](https://aws.amazon.com/cli/) (CLI) is already installed in Cloud9.

For this workshop, we are also going to use the [Fargate CLI](https://github.com/jpignata/fargate).
The Fargate CLI is a little tool that abstracts the complexity of using AWS CLI
to launch container tasks on Fargate.

{{% notice tip %}}
In this workshop we will give you the commands to download the Linux
binaries. If you are running macOS / Windows, please [see the official docs
for the download links.](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html)
{{% /notice %}}

#### Install CLI utilities
```
sudo yum -y install jq
```

#### Configure the AWS account ID and region
```
export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query 'Account')
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r .region)
aws configure set default.region $AWS_REGION
```

