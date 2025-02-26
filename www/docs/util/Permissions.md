---
description: "Docs for how permissions are handled in the @serverless-stack/resources"
---

import config from "../../config";

SST makes it easy to select the level of access you want to grant while attaching permissions to your application.

The `Permissions` type is used in:

1. The various `attachPermissions` style functions. For example, [`attachPermissions`](../constructs/Function.md#attachpermissions) in the `Function` construct.
2. The [`attachPermissionsForAuthUsers`](../constructs/Auth.md#attachpermissionsforauthusers) and [`attachPermissionsForUnauthUsers`](../constructs/Auth.md#attachpermissionsforunauthusers) in the `Auth` construct.

## Examples

Let's look at the various ways to attach permissions. Starting with the most permissive option.

Take a simple function.

```js
const fun = new Function(this, "Function", { handler: "src/lambda.main" });
```

### Giving full permissions

```js
fun.attachPermissions(PermissionType.ALL);
```

This allows the function admin access to all resources.

### Access to a list of services

```js
fun.attachPermissions(["s3", "dynamodb"]);
```

Specify a list of AWS resource types that this function has complete access to. Takes a list of strings.

### Access to a list of constructs

```js
import * as sns from "@aws-cdk/aws-sns";

const sns = new sns.Topic(this, "Topic");
const table = new Table(this, "Table");

fun.attachPermissions([sns, table]);
```

Specify which SST or CDK constructs you want to give complete access to. [Check out the list of supported constructs](#supported-constructs).

### Access to a list of specific permissions in a construct

```js
import * as dynamodb from "@aws-cdk/aws-dynamodb";

const sns = new sns.Topic(this, "Topic");
const table = new dynamodb.Table(this, "Table");

fun.attachPermissions([
 [topic, "grantPublish"],
 [table, "grantReadData"],
]);
```

Specify which permission in the construct you want to give access to. Specified as a tuple of construct and a grant permission function.

CDK constructs have methods of the format _grantX_ that allow you to grant specific permissions. So in the example above, the grant functions are: [`Topic.grantPublish`](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-sns.Topic.html#grantwbrpublishgrantee) and [`Table.grantReadData`](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-dynamodb.Table.html#grantwbrreadwbrdatagrantee). The `attachPermissions` method, takes the construct and calls the grant permission function specified.

Unlike the previous option, this supports all the CDK constructs.

### List of IAM policies

```js
import * as iam from "@aws-cdk/aws-iam";

fun.attachPermissions([
 new iam.PolicyStatement({
   actions: ["s3:*"],
   effect: iam.Effect.ALLOW,
   resources: [
     bucket.bucketArn + "/private/${cognito-identity.amazonaws.com:sub}/*",
   ],
 }),
 new iam.PolicyStatement({
   actions: ["execute-api:Invoke"],
   effect: iam.Effect.ALLOW,
   resources: [
     `arn:aws:execute-api:${region}:${account}:${api.httpApiId}/*`,
   ],
 }),
]);
```

The [`cdk.aws-iam.PolicyStatement`](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.PolicyStatement.html) allows you to craft granular IAM policies that you can attach to the function.

## Types

Below are the types and enums used to support permissions in SST.

### Permissions

_Type_ : `PermissionType | Permission[]`

Takes a [`PermissionType`](#permissiontype) or an array of [`Permission`](#permission).

On a high level, you can either give admin access to all the resources in your account or a specific list of services.

### PermissionType

An enum with the following option(s).

| Member | Description                                   |
| ------ | --------------------------------------------- |
| ALL    | Gives complete admin access to all resources. |

In a `Function` construct this would look like.

Set using `sst.PermissionType.ALL`.

### Permission

_Type_ : `string | cdk.Construct | [cdk.Construct, string] | cdk.aws-iam.PolicyStatement`

Allows you to define the permission in a few different ways to control the level of access.

The name of the AWS resource as referenced in an IAM policy.

```
"s3"
"dynamodb"
...
```

A CDK or SST construct. [Check out the list of supported constructs](#supported-constructs).

```
new cdk.aws-sns.Topic(this, "Topic")
new sst.Table(this, "Table")
...
```

A CDK construct with their specific grant permission method. Many CDK constructs have a method of the format _grantX_ that allows you to grant specific permissions. Pass in the consutrct and grant method as a tuple.

```
// const sns = new cdk.aws-sns.Topic(this, "Topic");
// const table = new sst.Table(this, "Table");

[topic, "grantPublish"]
[table, "grantReadData"]
```

Or, pass in a policy statement.

```
new cdk.aws-iam.PolicyStatement({
  actions: ["s3:*"],
  effect: cdk.aws-iam.Effect.ALLOW,
  resources: [
    bucket.bucketArn + "/private/${cognito-identity.amazonaws.com:sub}/*",
  ],
})
```

#### Supported Constructs

You can grant access to an SST or CDK construct.

``` js
fun.attachPermissions([sns, table]);
```

Currently the following SST and CDK constructs are supported.

- [Api](../constructs/Api.md)
- [Topic](../constructs/Topic.md)
- [Table](../constructs/Table.md)
- [Queue](../constructs/Queue.md)
- [Bucket](../constructs/Bucket.md)
- [Function](../constructs/Function.md)
- [EventBus](../constructs/EventBus.md)
- [ApolloApi](../constructs/ApolloApi.md)
- [AppSyncApi](../constructs/AppSyncApi.md)
- [KinesisStream](../constructs/KinesisStream.md)
- [WebSocketApi](../constructs/WebSocketApi.md)
- [ApiGatewayV1Api](../constructs/ApiGatewayV1Api.md)
- [cdk.aws-sns.Topic](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-sns.Topic.html)
- [cdk.aws-s3.Bucket](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html)
- [cdk.aws-sqs.Queue](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-sqs.Queue.html)
- [cdk.aws-dynamodb.Table](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-dynamodb.Table.html)
- [cdk.aws-rds.ServerlessCluster](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-rds.ServerlessCluster.html)

To add to this list, please <a href={ `${config.github}/issues/new` }>open a new issue</a>.
