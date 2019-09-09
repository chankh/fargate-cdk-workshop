---
title: "Create a new project"
weight: 20
---

Once we've installed the AWS CDK Toolkit, we will use `cdk init` to create a new
project. We will learn how to use the CDK Toolkit to synthesize an AWS
CloudFormation template for our application and how to deploy it into your
account.

#### Create the project directory
Create an empty directory in your Cloud9 environment.

```
mkdir ~/environment/cdk-workshop
cd ~/environment/cdk-workshop
```

#### Initialize the project
We will use `cdk init` to create a new TypeScript CDK project
```
cdk init app --language typescript
```

Output should look like this

```
Applying project template app for typescript
Initializing a new git repository...
Executing npm install...
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN cdk@0.1.0 No repository field.
npm WARN cdk@0.1.0 No license field.

# Useful commands

 * `npm run build`   compile typescript to js
 * `npm run watch`   watch for changes and compile
 * `cdk deploy`      deploy this stack to your default AWS account/region
 * `cdk diff`        compare deployed stack with current state
 * `cdk synth`       emits the synthesized CloudFormation template
```

As you can see, it shows us a bunch of useful commands to get started.

Since TypeScript sources need to be compiled to JavaScript, everytime we make a
modification to our source files, we would want them to be compiled to `.js`.

#### Project directory

The `cdk init` command has created a project structure for us in the directory.
Let's explore the project directory.

![project-structure](/images/infrastructure/folder_structure.png)

* __`lib/cdk-workshop-stack.ts`__ is where the your CDK application's main stack is defined.
  This is the file we'll be spending most of our time in.
* `bin/cdk-workshop.ts` is the entrypoint of the CDK application. It will load
  the stack defined in `lib/cdk-workshop-stack.ts`.
* `package.json` is your npm module manifest. It includes information like the
  name of your app, version, dependencies and build scripts like "watch" and
  "build" (`package-lock.json` is maintained by npm)
* `cdk.json` tells the toolkit how to run your app. In our case it will be
  `"node bin/cdk-workshop.js"`
* `tsconfig.json` your project's [typescript
  configuration](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)
* `.gitignore` and `.npmignore` tell git and npm which files to include/exclude
  from source control and when publishing this module to the package manager.
* `node_modules` is maintained by npm and includes all your project's
  dependencies.

##### Your app's entry point

Let's have a quick look at `bin/cdk-workshop.ts`:

```ts
#!/usr/bin/env node
import 'source-map-support/register';
import cdk = require('@aws-cdk/core');
import { CdkWorkshopStack } from '../lib/cdk-workshop-stack';

const app = new cdk.App();
new CdkWorkshopStack(app, 'CdkWorkshopStack');
```

This code loads and instantiate the `CdkWorkshopStack` class from the
`lib/cdk-workshop-stack.ts` file. We won't need to look at this file anymore.

##### The main stack

Open up `lib/cdk-workshop-stack.ts`. This is where the meat of our application
is:

```ts
import cdk = require('@aws-cdk/core');

export class CdkWorkshopStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // The code that defines your stack goes here
  }
}
```

As you can see, there is a CDK stack (`CdkWorkshopStack`) created with nothing
in it. Next, we will add our Spring PetClinic application into this stack.
