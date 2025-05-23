# cdk diff<a name="ref-cli-cmd-diff"></a>

Perform a diff to see infrastructure changes between AWS CDK stacks.

This command is typically used to compare differences between the current state of stacks in your local CDK app against deployed stacks. However, you can also compare a deployed stack with any local AWS CloudFormation template.

## Usage<a name="ref-cli-cmd-diff-usage"></a>

```
$ cdk diff <arguments> <options>
```

## Arguments<a name="ref-cli-cmd-diff-args"></a>

**CDK stack ID**  <a name="ref-cli-cmd-diff-args-stack-name"></a>
The construct ID of the CDK stack from your app to perform a diff.  
*Type*: String  
*Required*: No

## Options<a name="ref-cli-cmd-diff-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options).

`--change-set BOOLEAN`  <a name="ref-cli-cmd-diff-options-change-set"></a>
Specify whether to create a change set to analyze resource replacements.  
When `true`, the CDK CLI will create an AWS CloudFormation change set to display the exact changes that will be made to your stack. This output includes whether resources will be updated or replaced. The CDK CLI uses the deploy role instead of the lookup role to perform this action.  
When `false`, a quicker, but less-accurate diff is performed by comparing CloudFormation templates. Any change detected to properties that require resource replacement will be displayed as a resource replacement, even if the change is purely cosmetic, like replacing a resource reference with a hard-coded ARN.  
*Default value*: `true`

`--context-lines NUMBER`  <a name="ref-cli-cmd-diff-options-context-lines"></a>
Number of context lines to include in arbitrary JSON diff rendering.  
*Default value*: `3`

`--exclusively, -e BOOLEAN`  <a name="ref-cli-cmd-diff-options-exclusively"></a>
Only diff requested stacks and don’t include dependencies.

`--fail BOOLEAN`  <a name="ref-cli-cmd-diff-options-fail"></a>
Fail and exit with a code of `1` if differences are detected.

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-diff-options-help"></a>
Show command reference information for the `cdk diff` command.

`--processed BOOLEAN`  <a name="ref-cli-cmd-diff-options-processed"></a>
Specify whether to compare against the template with CloudFormation transforms already processed.  
*Default value*: `false`

`--quiet, -q BOOLEAN`  <a name="ref-cli-cmd-diff-options-quiet"></a>
Do not print the CDK stack name and default `cdk diff` message to `stdout` when no changes are detected.  
*Default value*: `false`

`--security-only BOOLEAN`  <a name="ref-cli-cmd-diff-options-security-only"></a>
Only diff for broadened security changes.  
*Default value*: `false`

`--strict BOOLEAN`  <a name="ref-cli-cmd-diff-options-strict"></a>
Modify `cdk diff` behavior to be more precise or stringent. When true, the CDK CLI will not filter out `AWS::CDK::Metadata` resources or unreadable non-ASCII characters.  
*Default value*: `false`

`--template STRING`  <a name="ref-cli-cmd-diff-options-template"></a>
The path to the CloudFormation template to compare a CDK stack with.

## Examples<a name="ref-cli-cmd-diff-examples"></a>

### Diff against the currently deployed stack named MyStackName<a name="ref-cli-cmd-diff-examples-1"></a>

The CDK CLI uses the following symbols in the diff output:
+ `[+]` – Identifies code or resources that will be added if you deploy your changes.
+ `[-]` – Identifies code or resources that will be removed if you deploy your changes.
+ `[~]` – Identifies a resource or property that will be modified if you deploy your changes.

The following is an example that shows a diff of local changes to a Lambda function:

```
$ cdk diff MyStackName
start: Building asset-hash:account-Region
success: Built asset-hash:account-Region
start: Publishing asset-hash:account-Region
success: Published asset-hash:account-Region
Hold on while we create a read-only change set to get a diff with accurate replacement information (use --no-change-set to use a less accurate but faster template-only diff)
Stack MyStackName
Resources
[~] AWS::Lambda::Function HelloWorldFunction resource-logical-ID
 └─ [~] Code
     └─ [~] .ZipFile:
         ├─ [-] 
        exports.handler = async function(event) {
          return {
            statusCode: 200,
            body: JSON.stringify('Hello World!'),
          };
        };
      
         └─ [+] 
        exports.handler = async function(event) {
          return {
            statusCode: 200,
            body: JSON.stringify('Hello from CDK!'),
          };
        };
      


✨  Number of stacks with differences: 1
```

A `[~]` indicator for resources that will be modified does not always mean a full resource replacement:
+ Some resource properties, like `Code`, will update the resource.
+ Some resource properties, like `FunctionName`, may cause a full resource replacement.

### Diff against a specific CloudFormation template<a name="ref-cli-cmd-diff-examples-2"></a>

```
$ cdk diff MyStackName --app='node bin/main.js' --template-path='./MyStackNameTemplate.yaml'
```

### Diff a local stack with its deployed stack. Don’t print to stdout if no changes are detected<a name="ref-cli-cmd-diff-examples-3"></a>

```
$ cdk diff MyStackName --app='node bin/main.js' --quiet
```