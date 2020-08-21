---
name: AWS Lambda
doc_link: https://docs.sentry.io/platforms/node/aws_lambda/
support_level: production
type: framework
---

Create a deployment package on your local machine and install the required dependencies in the deployment package.
For more information, see [AWS Lambda deployment package in Node](https://docs.aws.amazon.com/lambda/latest/dg/nodejs-package.html).

Install our Node.js SDK using `npm`:

```bash
$ npm install @sentry/node
```

You can use the AWS Lambda integration for the Node SDK as described below. Make sure the Sentry Node SDK is initialized inside the handler function.

```javascript
"use strict";

const Sentry = require("@sentry/serverless");
const AwsLambda = Sentry.Integrations.AWSLambda;

exports.handler = async (event, context) => {
     Sentry.init({
        dsn: "___PUBLIC_DSN___",
        integrations: [new AwsLambda({ context })],
     });
}
```
Checkout Sentry's [aws sample apps](https://github.com/getsentry/examples/tree/master/aws-lambda/node) for detailed examples.

### Enable Timeout Warning
Update the sentry initialization to set `timeoutWarning` to `true`

```javascript
Sentry.init({
        dsn: "___PUBLIC_DSN___",
        integrations: [new AwsLambda({ context, timeoutWarning: true })],
});
```

The timeout warning is sent only if the "timeout" in the Lambda Funtion configuration is set to a value greater than one second.
