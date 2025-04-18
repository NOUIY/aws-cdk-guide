# Building AWS CDK applications with AWS SAM<a name="testing-locally-build-with-sam-cli"></a>

The AWS SAM CLI provides support for building Lambda functions and layers defined in your AWS CDK application with [sam build](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-build.html).

For Lambda functions that use zip artifacts, run `cdk synth` before you run `sam local` commands. `sam build` isn't required.

If your AWS CDK application uses functions with the image type, run `cdk synth` and then run `sam build` before you run `sam local` commands. When you run `sam build`, AWS SAM doesn't build Lambda functions or layers that use runtime-specific constructs, for example, [NodejsFunction](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_lambda_nodejs.NodejsFunction.html). `sam build` doesn't support [bundled assets](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.BundlingOptions.html).

## Example<a name="testing-locally-build-with-sam-cli-examples"></a>

Running the following command from the AWS CDK project root directory builds the application.

```
$ sam build -t ./cdk.out/CdkSamExampleStack.template.json
```