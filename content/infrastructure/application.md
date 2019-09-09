---
title: "Add our application"
weight: 30
---

Here, we will finally write some CDK code. From the empty stack that was
generated, we will add a Fargate service using one of the provided ECS patterns.

#### Install the construct library

The AWS CDK is shipped with an extensive libirary of constructs called **AWS
Construct Library**. The construct library is divided into **modules**, one for
each AWS service. For example, if you want to define an Amazon EC2 instance, we
will need to use the `aws-ec2` construct library.

To discover and learn about AWS constructs, you can browse the [AWS Construct
Library reference](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html).

Okay, let's use `npm install` to install the required modules and all it's
dependencies into our project.

```
npm install @aws-cdk/aws-ec2
npm install @aws-cdk/aws-ecs
npm install @aws-cdk/aws-ecs-patterns
```

{{% notice note %}}
You can safely ignore any warnings from npm about your package.json file.
{{% /notice %}}

#### Create our Fargate service

Let's look at the power of the AWS CDK. Open the `lib/cdk-workshop-stack.ts`
file, add a few `import` statements at the beginning of the file, and define our
resources in the stack.

{{<highlight ts "hl_lines=2-4 11-28">}}
import cdk = require('@aws-cdk/core');
import ec2 = require('@aws-cdk/aws-ec2');
import ecs = require('@aws-cdk/aws-ecs');
import ecs_patterns = require('@aws-cdk/aws-ecs-patterns');

export class CdkWorkshopStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // The code that defines your stack goes here
    const vpc = new ec2.Vpc(this, "workshop", {
      maxAzs: 3 // Default is all AZs in region
    });

    const cluster = new ecs.Cluster(this, "fargate", {
      vpc: vpc
    });

    // Create a load-balanced Fargate service and make it public
    new ecs_patterns.LoadBalancedFargateService(this, "spring-petclinic", {
      cluster: cluster, // Required
      containerPort: 8080, // Port number on the container, default is 80
      cpu: 512, // Default is 256
      desiredCount: 2, // Default is 1
      image: ecs.ContainerImage.fromAsset("../spring-petclinic"), // Required
      memoryLimitMiB: 1024, // Default is 512
      publicLoadBalancer: true // Default is false
    });
  }
}
{{</highlight>}}

In less than 30 lines of TypeScript code, we are able to create an AWS Fargate
service with the following benefits:

- Automatically configures a load balancer.
- Automatically opens a security group for load balancers. This enables load
balancers to communicate with instances without you explicitly creating a
security group.
- Automatically orders dependency between the service and the load balancer
attaching to a target group, where the AWS CDK enforces the correct order of
creating the listener before an instance is created.
- Automatically configures user data on automatically scaling groups. This
creates the correct configuration to associate a cluster to AMIs.
- Validates parameter combinations early. This exposes AWS CloudFormation issues
earlier, thus saving you deployment time. For example, depending on the task,
it's easy to misconfigure the memory settings. Previously, you would not
encounter an error until you deployed your app. But now the AWS CDK can detect
a misconfiguration and emit an error when you synthesize your app.
- Automatically adds permissions for Amazon Elastic Container Registry (Amazon
ECR) if you use an image from Amazon ECR.
- Automatically scales. The AWS CDK supplies a method so you can autoscaling
instances when you use an Amazon EC2 cluster. This happens automatically when
you use an instance in a Fargate cluster.
- In addition, the AWS CDK prevents an instance from being deleted when
automatic scaling tries to kill an instance, but either a task is running or is
scheduled on that instance.
Previously, you had to create a Lambda function to have this functionality.
- Provides asset support, so that you can deploy a source from your machine to
Amazon ECS in one step. Previously, to use an application source you had to
perform several manual steps, such as uploading to Amazon ECR and creating a
Docker image.

#### Checking the diff

Save your code, and let's take a quick look at the diff before we deploy.

```
cdk diff
```

Output would look like this

```
Stack CdkWorkshopStack
The CdkWorkshopStack stack uses assets, which are currently not accounted for in the diff output! See https://github.com/aws/aws-cdk/issues/395
IAM Statement Changes
┌───┬────────────────────────────────────────────────┬────────┬────────────────────────────────────────────────┬─────────────────────────────────────────────────┬───────────┐
│   │ Resource                                       │ Effect │ Action                                         │ Principal                                       │ Condition │
├───┼────────────────────────────────────────────────┼────────┼────────────────────────────────────────────────┼─────────────────────────────────────────────────┼───────────┤
│ + │ ${AdoptEcrRepositorydbc60defc59544bcaa5c28c95d │ Allow  │ sts:AssumeRole                                 │ Service:lambda.amazonaws.com                    │           │
│   │ 68f62c/ServiceRole.Arn}                        │        │                                                │                                                 │           │
├───┼────────────────────────────────────────────────┼────────┼────────────────────────────────────────────────┼─────────────────────────────────────────────────┼───────────┤
│ + │ ${spring-petclinic/TaskDef/ExecutionRole.Arn}  │ Allow  │ sts:AssumeRole                                 │ Service:ecs-tasks.amazonaws.com                 │           │
├───┼────────────────────────────────────────────────┼────────┼────────────────────────────────────────────────┼─────────────────────────────────────────────────┼───────────┤
│ + │ ${spring-petclinic/TaskDef/TaskRole.Arn}       │ Allow  │ sts:AssumeRole                                 │ Service:ecs-tasks.amazonaws.com                 │           │
├───┼────────────────────────────────────────────────┼────────┼────────────────────────────────────────────────┼─────────────────────────────────────────────────┼───────────┤
│ + │ ${spring-petclinic/TaskDef/web/LogGroup.Arn}   │ Allow  │ logs:CreateLogStream                           │ AWS:${spring-petclinic/TaskDef/ExecutionRole}   │           │
│   │                                                │        │ logs:PutLogEvents                              │                                                 │           │
├───┼────────────────────────────────────────────────┼────────┼────────────────────────────────────────────────┼─────────────────────────────────────────────────┼───────────┤
│ + │ *                                              │ Allow  │ ecr:GetAuthorizationToken                      │ AWS:${spring-petclinic/TaskDef/ExecutionRole}   │           │
├───┼────────────────────────────────────────────────┼────────┼────────────────────────────────────────────────┼─────────────────────────────────────────────────┼───────────┤
│ + │ arn:${AWS::Partition}:ecr:${AWS::Region}:${AWS │ Allow  │ ecr:BatchCheckLayerAvailability                │ AWS:${spring-petclinic/TaskDef/ExecutionRole}   │           │
│   │ ::AccountId}:repository/${springpetclinicTaskD │        │ ecr:BatchGetImage                              │                                                 │           │
│   │ efwebAssetImageAdoptRepository17144F4B.Reposit │        │ ecr:GetDownloadUrlForLayer                     │                                                 │           │
│   │ oryName}                                       │        │                                                │                                                 │           │
├───┼────────────────────────────────────────────────┼────────┼────────────────────────────────────────────────┼─────────────────────────────────────────────────┼───────────┤
│ + │ arn:${AWS::Partition}:ecr:${AWS::Region}:${AWS │ Allow  │ ecr:BatchDeleteImage                           │ AWS:${AdoptEcrRepositorydbc60defc59544bcaa5c28c │           │
│   │ ::AccountId}:repository/{"Fn::Select":[0,"{\"F │        │ ecr:DeleteRepository                           │ 95d68f62c/ServiceRole}                          │           │
│   │ n::Split\":[\"@sha256:\",\"${springpetclinicTa │        │ ecr:GetRepositoryPolicy                        │                                                 │           │
│   │ skDefwebAssetImageImageName40F1DFAD}\"]}"]}    │        │ ecr:ListImages                                 │                                                 │           │
│   │                                                │        │ ecr:SetRepositoryPolicy                        │                                                 │           │
└───┴────────────────────────────────────────────────┴────────┴────────────────────────────────────────────────┴─────────────────────────────────────────────────┴───────────┘
IAM Policy Changes
┌───┬───────────────────────────────────────────────────────────────────┬────────────────────────────────────────────────────────────────────────────────┐
│   │ Resource                                                          │ Managed Policy ARN                                                             │
├───┼───────────────────────────────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────┤
│ + │ ${AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62c/ServiceRole} │ arn:${AWS::Partition}:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole │
└───┴───────────────────────────────────────────────────────────────────┴────────────────────────────────────────────────────────────────────────────────┘
Security Group Changes
┌───┬───────────────────────────────────────────────────┬─────┬────────────┬───────────────────────────────────────────────────┐
│   │ Group                                             │ Dir │ Protocol   │ Peer                                              │
├───┼───────────────────────────────────────────────────┼─────┼────────────┼───────────────────────────────────────────────────┤
│ + │ ${spring-petclinic/LB/SecurityGroup.GroupId}      │ In  │ TCP 80     │ Everyone (IPv4)                                   │
│ + │ ${spring-petclinic/LB/SecurityGroup.GroupId}      │ Out │ TCP 80     │ ${spring-petclinic/Service/SecurityGroup.GroupId} │
├───┼───────────────────────────────────────────────────┼─────┼────────────┼───────────────────────────────────────────────────┤
│ + │ ${spring-petclinic/Service/SecurityGroup.GroupId} │ In  │ TCP 80     │ ${spring-petclinic/LB/SecurityGroup.GroupId}      │
│ + │ ${spring-petclinic/Service/SecurityGroup.GroupId} │ Out │ Everything │ Everyone (IPv4)                                   │
└───┴───────────────────────────────────────────────────┴─────┴────────────┴───────────────────────────────────────────────────┘
(NOTE: There may be security-related changes not in this list. See http://bit.ly/cdk-2EhF7Np)

Parameters
[+] Parameter spring-petclinic/TaskDef/web/AssetImage/ImageName springpetclinicTaskDefwebAssetImageImageName40F1DFAD: {"Type":"String","Description":"ECR repository name and tag asset \"CdkWorkshopStack/spring-petclinic/TaskDef/web/AssetImage\""}
[+] Parameter AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62c/Code/S3Bucket AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62cCodeS3Bucket92AB06B6: {"Type":"String","Description":"S3 bucket for asset \"CdkWorkshopStack/AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62c/Code\""}
[+] Parameter AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62c/Code/S3VersionKey AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62cCodeS3VersionKey393B7276: {"Type":"String","Description":"S3 key for asset version \"CdkWorkshopStack/AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62c/Code\""}
[+] Parameter AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62c/Code/ArtifactHash AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62cCodeArtifactHash8BCBAA49: {"Type":"String","Description":"Artifact hash for asset \"CdkWorkshopStack/AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62c/Code\""}

Resources
[+] AWS::EC2::VPC workshop workshop774D4CF0 
[+] AWS::EC2::Subnet workshop/PublicSubnet1/Subnet workshopPublicSubnet1Subnet87990C87 
[+] AWS::EC2::RouteTable workshop/PublicSubnet1/RouteTable workshopPublicSubnet1RouteTable0C7CE44C 
[+] AWS::EC2::SubnetRouteTableAssociation workshop/PublicSubnet1/RouteTableAssociation workshopPublicSubnet1RouteTableAssociation4E1D7DFD 
[+] AWS::EC2::Route workshop/PublicSubnet1/DefaultRoute workshopPublicSubnet1DefaultRouteE5DA3DB8 
[+] AWS::EC2::EIP workshop/PublicSubnet1/EIP workshopPublicSubnet1EIP32B6E54E 
[+] AWS::EC2::NatGateway workshop/PublicSubnet1/NATGateway workshopPublicSubnet1NATGateway5EC38F2B 
[+] AWS::EC2::Subnet workshop/PublicSubnet2/Subnet workshopPublicSubnet2Subnet43D6D347 
[+] AWS::EC2::RouteTable workshop/PublicSubnet2/RouteTable workshopPublicSubnet2RouteTableEB105B89 
[+] AWS::EC2::SubnetRouteTableAssociation workshop/PublicSubnet2/RouteTableAssociation workshopPublicSubnet2RouteTableAssociationAFEB2B26 
[+] AWS::EC2::Route workshop/PublicSubnet2/DefaultRoute workshopPublicSubnet2DefaultRouteC15E1A3A 
[+] AWS::EC2::EIP workshop/PublicSubnet2/EIP workshopPublicSubnet2EIP9D3589CB 
[+] AWS::EC2::NatGateway workshop/PublicSubnet2/NATGateway workshopPublicSubnet2NATGateway4B6CA47C 
[+] AWS::EC2::Subnet workshop/PrivateSubnet1/Subnet workshopPrivateSubnet1Subnet0C43F3C1 
[+] AWS::EC2::RouteTable workshop/PrivateSubnet1/RouteTable workshopPrivateSubnet1RouteTableA22DC8AD 
[+] AWS::EC2::SubnetRouteTableAssociation workshop/PrivateSubnet1/RouteTableAssociation workshopPrivateSubnet1RouteTableAssociationBEA87B8D 
[+] AWS::EC2::Route workshop/PrivateSubnet1/DefaultRoute workshopPrivateSubnet1DefaultRouteC1164F50 
[+] AWS::EC2::Subnet workshop/PrivateSubnet2/Subnet workshopPrivateSubnet2SubnetEC3F2437 
[+] AWS::EC2::RouteTable workshop/PrivateSubnet2/RouteTable workshopPrivateSubnet2RouteTable91F8D528 
[+] AWS::EC2::SubnetRouteTableAssociation workshop/PrivateSubnet2/RouteTableAssociation workshopPrivateSubnet2RouteTableAssociation5604EA35 
[+] AWS::EC2::Route workshop/PrivateSubnet2/DefaultRoute workshopPrivateSubnet2DefaultRoute415D85ED 
[+] AWS::EC2::InternetGateway workshop/IGW workshopIGWE45F4CE9 
[+] AWS::EC2::VPCGatewayAttachment workshop/VPCGW workshopVPCGW69842950 
[+] AWS::ECS::Cluster fargate fargate92BC2AF7 
[+] AWS::ElasticLoadBalancingV2::LoadBalancer spring-petclinic/LB springpetclinicLB0555AA24 
[+] AWS::EC2::SecurityGroup spring-petclinic/LB/SecurityGroup springpetclinicLBSecurityGroupE7AD236C 
[+] AWS::EC2::SecurityGroupEgress spring-petclinic/LB/SecurityGroup/to CdkWorkshopStackspringpetclinicServiceSecurityGroupFD8E91D2:80 springpetclinicLBSecurityGrouptoCdkWorkshopStackspringpetclinicServiceSecurityGroupFD8E91D2800D2BEBB3 
[+] AWS::ElasticLoadBalancingV2::Listener spring-petclinic/LB/PublicListener springpetclinicLBPublicListener8C3EB78B 
[+] AWS::ElasticLoadBalancingV2::TargetGroup spring-petclinic/LB/PublicListener/ECSGroup springpetclinicLBPublicListenerECSGroup1A7F3F24 
[+] AWS::IAM::Role spring-petclinic/TaskDef/TaskRole springpetclinicTaskDefTaskRole4F06F33E 
[+] AWS::ECS::TaskDefinition spring-petclinic/TaskDef springpetclinicTaskDefC1DF9D5F 
[+] Custom::ECRAdoptedRepository spring-petclinic/TaskDef/web/AssetImage/AdoptRepository/Resource springpetclinicTaskDefwebAssetImageAdoptRepository17144F4B 
[+] AWS::Logs::LogGroup spring-petclinic/TaskDef/web/LogGroup springpetclinicTaskDefwebLogGroupFA9C756C 
[+] AWS::IAM::Role spring-petclinic/TaskDef/ExecutionRole springpetclinicTaskDefExecutionRole2EA1B442 
[+] AWS::IAM::Policy spring-petclinic/TaskDef/ExecutionRole/DefaultPolicy springpetclinicTaskDefExecutionRoleDefaultPolicy5F55F587 
[+] AWS::ECS::Service spring-petclinic/Service/Service springpetclinicService71C976E2 
[+] AWS::EC2::SecurityGroup spring-petclinic/Service/SecurityGroup springpetclinicServiceSecurityGroupEE77DD29 
[+] AWS::EC2::SecurityGroupIngress spring-petclinic/Service/SecurityGroup/from CdkWorkshopStackspringpetclinicLBSecurityGroup998846D8:80 springpetclinicServiceSecurityGroupfromCdkWorkshopStackspringpetclinicLBSecurityGroup998846D8807D7A3918 
[+] AWS::IAM::Role AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62c/ServiceRole AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62cServiceRoleD788AA17 
[+] AWS::IAM::Policy AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62c/ServiceRole/DefaultPolicy AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62cServiceRoleDefaultPolicy6BC8737C 
[+] AWS::Lambda::Function AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62c AdoptEcrRepositorydbc60defc59544bcaa5c28c95d68f62c52BE89E9 

Outputs
[+] Output spring-petclinic/LoadBalancerDNS springpetclinicLoadBalancerDNS31C6BB59: {"Value":{"Fn::GetAtt":["springpetclinicLB0555AA24","DNSName"]}}
```

As you can see, this code synthesizes a couple of CloudFormation resources and
parameters for you.
