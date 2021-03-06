---
id: resources
title: "@serverless-stack/resources"
description: "Docs for the @serverless-stack/resources package"
---

The `@serverless-stack/resources` package provides a couple of simple AWS CDK Constructs:

- `sst.App` (used internally)
- `sst.Stack`
- `sst.Function`

## sst.Stack

The `sst.Stack` and `sst.App` constructs allow you to:

- Automatically prefix stack names with the stage
- Optionally prefix resource names with the stage
- Deploy the entire app using the same AWS profile and region

### Creating a new stack

Create a new stack by adding this in `lib/MyStack.js`.

```jsx
import * as sst from "@serverless-stack/resources";

export default class MyStack extends sst.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Define your stack
  }
}
```

Here `sst.Stack` is a simple extension of `cdk.Stack` that prefixes the stack name with the stage and enforces using the global region and AWS profile.

### Adding to an app

Add it to your app in `lib/index.js`.

```jsx
import MyStack from "./MyStack";

export default function main(app) {
  new MyStack(app, "my-stack");

  // Add more stacks
}
```

Here `app` is an instance of `sst.App`. It's a simple extension of `cdk.App`.

Note that, setting the env for an individual stack is not allowed.

```jsx
new MyStack(app, "my-stack", { env: { account: "1234", region: "us-east-1" } });
```

It will throw this error.

```
Error: Do not directly set the environment for a stack
```

This is by design. The stacks in SST are meant to be re-deployed for multiple stages (like Serverless Framework). And so they depend on the region and AWS profile that's passed in through the CLI. If a stack is hardcoded to be deployed to a specific account or region, it can break your deployment pipeline.

### Accessing app info

The stage, region, and app name can be accessed through the app object.

So in the `lib/index.js` you can access it using.

```js
app.stage;
app.region;
app.name;
```

And in your stack classes (for example, `lib/MyStack.js`) you can use.

```js
this.node.root.stage;
this.node.root.region;
this.node.root.name;
```

You can use this to conditionally add stacks or resources to your app.

### Prefixing resource names

You can optionally prefix resource names to make sure they don't thrash when deployed to different stages in the same AWS account.

You can do so in your stacks.

```jsx
this.node.root.logicalPrefixedName("MyResource"); // Returns "dev-my-sst-app-MyResource"
```

This invokes the `logicalPrefixedName` method in `sst.App` that your stack is added to. This'll return `dev-my-sst-app-MyResource`, where `dev` is the current stage and `my-sst-app` is the name of the app.

## sst.Function

A replacement for the [`cdk.lambda.NodejsFunction`](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-lambda-nodejs-readme.html) that allows you to develop your Lambda functions locally while using [`sst start`](packages/cli.md#start). Supports ES and TypeScript out-of-the-box.

Takes the following props in addition to the [`cdk.lambda.FunctionOptions`](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-lambda.FunctionOptions.html).

By default, `AWS_NODEJS_CONNECTION_REUSE_ENABLED` is turned on. Meaning that the Lambda function will automatically reuse TCP connections when working with the AWS SDK. [Read more about this here](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/node-reusing-connections.html).

Also, [enables AWS X-Ray](https://docs.aws.amazon.com/lambda/latest/dg/nodejs-tracing.html) by default so you can trace your serverless applications.

### `handler`

Path to the entry point and handler function. Of the format `/path/to/file.function`. First checks for `.ts` file and then for a `.js` file.

If the [`srcPath`](#srcpath) is set, then the path to the `handler` is relative to it.

### `bundle`

Bundles your Lambda functions with [esbuild](https://esbuild.github.io). Turn this off if you have NPM packages that cannot be bundled.

Defaults to `true`.

### `srcPath`

The source directory where the handler file is located. If the `bundle` option is turned off, SST zips up the entire `srcPath` directory and uses it as the Lambda function package.

Defaults to `""`, the project root.

### `runtime`

The runtime environment. Only runtimes of the Node.js family are supported.

Defaults to `lambda.NODEJS_12_X`.
