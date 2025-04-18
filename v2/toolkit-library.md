# Perform programmatic actions using the CDK Toolkit Library<a name="toolkit-library"></a>

**Note**  
CDK Toolkit Library is in developer preview and is subject to change.

## What is the CDK Toolkit Library Library?<a name="toolkit-library-intro"></a>

The AWS Cloud Development Kit (AWS CDK) Toolkit Library enables you to perform AWS CDK actions requiring programmatic access on AWS. You can use the CDK Toolkit Library to implement actions such as bootstrapping, synthesizing, and deploying through code rather than CDK CLI commands. With this library, you can create custom tools, build specialized CLI applications, and integrate CDK programmatic access capabilities into your development workflows.

The following is an example, that installs the CDK Toolkit Library and uses it to deploy a cloud assembly of your CDK app:

```
import { Toolkit } from '@aws-cdk/toolkit-lib'; // Install the CDK Toolkit Library

const cdk = new Toolkit(); // Create a CDK Toolkit instance

// ...

// Implement a deployment
await cdk.deploy(cloudAssembly, { 
   deploymentMethod: { method: "direct" }, // Configure deployment options
   ...
}
```

## Get started with the CDK Toolkit Library<a name="toolkit-library-gs"></a>

To get started, see the `[ReadMe](https://www.npmjs.com/package/@aws-cdk/toolkit-lib)` in the *@aws-cdk/toolkit-lib* `npm` package.

For API reference information, see the [CDK Toolkit Library API reference](https://docs.aws.amazon.com/cdk/api/toolkit-lib/).