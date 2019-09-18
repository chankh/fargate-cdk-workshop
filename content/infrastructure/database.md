---
title: "Configure a database"
weight: 50
---

In the previous steps, we deployed the Spring PetClinic application on AWS
Fargate using AWS CDK. In the default configuration, PetClinic uses an in-memory
database (HSQLDB) which gets populated at startup with data, and since it is
in-memory, data will be lost when application terminates. This is definitely not
what you would want in a real-world scenario. A database is usually used so that
the application can store and retrieve data from this database, and become
stateless.

In this section, we will look at how we can use a MySQL database managed by
[Amazon RDS](https://aws.amazon.com/rds) and provision it into our
infrastructure with AWS CDK.

#### Configure our application to use a database

First thing here we need to update our application configuration so that it will
use a database for persistence. Navigate to the directory `spring-petclinic` and
open up file `src/main/resources/application.properties`. Modify the first few
lines so that it should look like this.

{{<highlight text "hl_lines=2-5 14">}}
# database init, supports mysql too
database=mysql
spring.datasource.url=${DB_CONN}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}

spring.datasource.schema=classpath*:db/${database}/schema.sql
spring.datasource.data=classpath*:db/${database}/data.sql

# Web
spring.thymeleaf.mode=HTML

# JPA
spring.jpa.hibernate.ddl-auto=update
{{</highlight>}}

We changed the `database` to `mysql` here, and also specified the datasource
parameters. Take note here we are using environment variables for the datasource
URL, username and password. We will pass these values when we define our Fargate
service later in our CDK stack.

#### Add a database in our CDK stack

Before adding code to our CDK stack, we will need to first install the required 
construct library.

```
npm install @aws-cdk/aws-secretsmanager
npm install @aws-cdk/aws-rds
```

Let's go back to our `lib/cdk-workshop-stack.ts` and update the code to add new
import and resources.

{{<highlight ts "hl_lines=5-6 21-36 52-58">}}
import cdk = require('@aws-cdk/core');
import ec2 = require('@aws-cdk/aws-ec2');
import ecs = require('@aws-cdk/aws-ecs');
import ecs_patterns = require('@aws-cdk/aws-ecs-patterns');
import asm = require('@aws-cdk/aws-secretsmanager');
import rds = require('@aws-cdk/rds');

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

    // Let's first generate a password for the database
    const dbSecretId = "petclinicDbPassword";
    const dbSecret = new asm.Secret(this, dbSecretId);
    
    const db = new rds.DatabaseInstance(this, "PetClinicDb", {
      engine: rds.DatabaseInstanceEngine.MYSQL,
      masterUsername: "root",
      masterUserPassword: dbSecret.secretValue,
      instanceIdentifier: "PetClinicDB",
      instanceClass: ec2.InstanceType.of(
          ec2.InstanceClass.BURSTABLE3, ec2.InstanceSize.MICRO // Using db.t3.micro here
        ),
      vpc: vpc,
      databaseName: "petclinic",
      multiAz: false // Default is true, for this workshop we will not use multiAZ
    });

    // Open port 3306 for MySQL RDS Security Group
    const dbSG = ec2.SecurityGroup.fromSecurityGroupId(this, "PetClinicDBSG", db.securityGroupId);
    dbSG.addIngressRule(ec2.Peer.ipv4(ec2.Vpc.DEFAULT_CIDR_RANGE), ec2.Port.tcp(3306), "Allow connection within the VPC");
    
    const dbHost = db.dbInstanceEndpointAddress;
    const dbPort = db.dbInstanceEndpointPort;
    const dbUrl = "jdbc:mysql://" + dbHost + ":" + dbPort + "/petclinic";

    // Create a load-balanced Fargate service and make it public
    new ecs_patterns.ApplicationLoadBalancedFargateService(this, "spring-petclinic", {
      cluster: cluster, // Required
      containerPort: 8080, // Port number on the container, default is 80
      cpu: 512, // Default is 256
      desiredCount: 2, // Default is 1
      environment: {
        "DB_CONN": dbUrl,
        "DB_USERNAME": "root"
      },
      secrets: {
        "DB_PASSWORD": ecs.Secret.fromSecretsManager(dbSecret)
      },
      image: ecs.ContainerImage.fromAsset("../spring-petclinic"), // Required
      memoryLimitMiB: 1024, // Default is 512
      publicLoadBalancer: true // Default is false
    });
  }
}
{{</highlight>}}

You've noticed that apart from the RDS construct library, we have also used the
Secrets Manager construct library here. This is the way how CDK have enforced
best practice patterns by storing credentials such as the database password in a
secrets store. This prevents from putting down passwords as plain text in code.
You can either pre-generate a password and store that into [AWS Secrets Manager]
(https://aws.amazon.com/secrets-manager/) then reference the ARN of your secret
in CDK, or like what we've done here, generate a new secret directly using AWS
Secrets Manager.

Further down where we update the Fargate service, we inject into
the environment the values for database connection string and username, which
are plain text values, and as for the database password, we used another
approach by referencing the secret value that we've created earlier using
secrets manager.

#### Update the stack

Let's go ahead and update our existing stack, in the terminal execute `cdk deploy`
and wait for the change to be deployed.

```
cdk deploy
```

Once the stack is updated, you can visit the application using the URL from the
stack output. That's it! You now have a Spring Boot application running on AWS
Fargate, backed by a MySQL database managed by Amazon RDS.