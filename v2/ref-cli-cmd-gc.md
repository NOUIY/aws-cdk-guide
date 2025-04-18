# cdk gc<a name="ref-cli-cmd-gc"></a>

Use the AWS Cloud Development Kit (AWS CDK) command line interface (CLI) `cdk gc` command to perform garbage collection on unused assets stored in the resources of your bootstrap stack. Use this command to view, manage, and delete assets that you no longer need.

For Amazon Simple Storage Service (Amazon S3) assets, the CDK CLI will check existing AWS CloudFormation templates in the same environment to see if they are referenced. If not referenced, they will be considered unused and eligible for garbage collection actions.

**Warning**  
The `cdk gc` command is in development for the AWS CDK. The current features of this command are considered production ready and safe to use. However, the scope of this command and its features are subject to change. Therefore, you must opt in by providing the `unstable=gc` option to use this command.

## Usage<a name="ref-cli-cmd-gc-usage"></a>

```
$ cdk gc <arguments> <options>
```

## Arguments<a name="ref-cli-cmd-gc-args"></a>

**AWS environment**  <a name="ref-cli-cmd-gc-args-env"></a>
The target AWS environment to perform garbage collection actions on.  
When providing an environment, use the following format: `aws://<account-id>/<region>`. For example, `aws://123456789012/us-east-1`.  
This argument can be provided multiple times in a single command to perform garbage collection actions on multiple environments.  
By default, the CDK CLI will perform garbage collection actions on all environments that you reference in your CDK app or provide as arguments. If you don’t provide an environment, the CDK CLI will determine the environment from default sources. These sources include environments that you specify using the `--profile` option, environment variables, or default AWS CLI sources.

## Options<a name="ref-cli-cmd-gc-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options).

`--action STRING`  <a name="ref-cli-cmd-gc-options-action"></a>
The action that `cdk gc` performs on your assets during a garbage collection.  
+ `delete-tagged` – Deletes assets that have been tagged with a date within the range of buffer days that you provide, but does not tag newly identified unused assets.
+ `full` – Perform all garbage collection actions. This includes deleting assets within the range of buffer days that you provide and tagging newly identified unused assets.
+ `print` – Outputs the number of unused assets at the command prompt but doesn’t make any actual changes within your AWS environment.
+ `tag` – Tags any newly identified unused assets, but doesn’t delete any assets within the range of buffer days that you provide.
*Accepted values*: `delete-tagged`, `full`, `print`, `tag`  
*Default value*: `full`

`--bootstrap-stack-name STRING`  <a name="ref-cli-cmd-gc-options-bootstrap-stack-name"></a>
The name of the CDK bootstrap stack in your AWS environment. Provide this option if you customized your bootstrap stack name. If you are using the default `CDKToolkit` stack name, you don’t have to provide this option.  
*Default value*: `CDKToolkit`

`--confirm BOOLEAN`  <a name="ref-cli-cmd-gc-options-confirm"></a>
Specify whether the CDK CLI will request manual confirmation from you before deleting any assets.  
Specify `false` to automatically delete assets without prompting you for manual confirmation.  
*Default value*: `true`

`--created-buffer-days NUMBER`  <a name="ref-cli-cmd-gc-options-created-buffer-days"></a>
The number of days an asset must exist before it is eligible for garbage collection actions.  
When you provide a number, assets that have not existed beyond your specified number of days are filtered out from garbage collection actions.  
*Default value*: `1`

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-gc-options-help"></a>
Show command reference information for the `cdk gc` command.

`--rollback-buffer-days NUMBER`  <a name="ref-cli-cmd-gc-options-rollback-buffer-days"></a>
The number of days an asset must be tagged as isolated before it is eligible for garbage collection actions.  
When you provide a number, the CDK CLI will tag unused assets with the current date instead of deleting them. The CDK CLI will also check if any assets have been tagged during previous runs of the `cdk gc` command. Previously tagged assets that fall within the range of buffer days you provide will be deleted.  
*Default value*: `0`

`--type STRING`  <a name="ref-cli-cmd-gc-options-type"></a>
The bootstrap resource type within your bootstrap stack to perform garbage collection actions on.  
+ `all` – Perform garbage collection actions on all bootstrapped resources.
+ `ecr` – Perform garbage collection actions on assets in the Amazon Elastic Container Registry (Amazon ECR) repository of your bootstrap stack.
+ `s3` – Perform garbage collection actions on assets in the Amazon S3 bucket of your bootstrap stack.
*Accepted values*: `all`, `ecr`, `s3`  
*Default value*: `all`

`--unstable STRING`  <a name="ref-cli-cmd-gc-options-unstable"></a>
Allow the usage of CDK CLI commands that are still in development.  
This option is required to use any CDK CLI command that is still in development and subject to change.  
This option can be provided multiple times in a single command.  
To use `cdk gc`, provide `--unstable=gc`.

## Examples<a name="ref-cli-cmd-gc-examples"></a>

### Basic examples<a name="ref-cli-cmd-gc-examples-basic"></a>

The following example prompts you for manual confirmation to perform default garbage collection actions on assets in the Amazon S3 bucket of your bootstrap stack:

```
$ cdk gc --unstable=gc --type=s3

 ⏳  Garbage Collecting environment aws://<account-id>/<region>...
Found 99 assets to delete based off of the following criteria:
- assets have been isolated for > 0 days
- assets were created > 1 days ago

Delete this batch (yes/no/delete-all)?
```

The following example performs garbage collection actions on a range of assets in the Amazon S3 bucket of your bootstrap stack. This range includes assets that have been previously tagged by `cdk gc` for over 30 days and have been created 10 days or older. This command will prompt for manual confirmation before deleting any assets:

```
$ cdk gc --unstable=gc --type=s3 --rollback-buffer-days=30 --created-buffer-days=10
```

The following example performs the action of deleting previously tagged assets in the Amazon S3 bucket of your bootstrap stack that have been unused for longer than 30 days:

```
$ cdk gc --unstable=gc --type=s3 --action=delete-tagged --rollback-buffer-days=30
```