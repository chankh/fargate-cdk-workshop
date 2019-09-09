---
title: "Infrastructure"
chapter: true
weight: 32
---

# Overview

In the previous section, we have deployed the Spring PetClinic application to
AWS Fargate using the `fargate` CLI. The `fargate` CLI is a simple tool that
helps with some very basic tasks when run a container on AWS, such as scaling
and inspect logs. However eventually you would want to proceed on to better
best practices.

At AWS, we talked about infrastructure as code, one option could be to take this
and deploy the same using AWS CloudFormation, writing templates using YAML or
JSON. A nicer option would be using AWS Cloud Development Kit or in short, the
AWS CDK.

The AWS CDK is a new software development framework from AWS with the sole
purpose of making it fun and easy to define cloud infrastructure in your
favorite programming language and deploy it using AWS CloudFormation.

{{% notice info %}}

AWS CDK is currently supported in JavaScript, TypeScript, Python, Java, and .NET.
However, at this point the workshop is available in TypeScript and Python only.
Workshops tailored to more languages will be coming soon.

{{% /notice %}}
