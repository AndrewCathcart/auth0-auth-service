## auth0-auth-service

Adapted from [Serverless example](https://github.com/serverless/examples/tree/master/aws-node-auth0-custom-authorizers-api)

Protect API endpoints with Auth0, JWT & Custom Authorizer Lambda

Exposes a public and private endpoint for testing.

## Usage

### 1. Prerequisites

- [Node](https://nodejs.org/en/) is required. This can be installed via the website or by using [nvm](https://github.com/nvm-sh/nvm) (see their documentation).
- [AWS CLI Version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) (with config setup)
- Serverless `$ npm install -g serverless`
- Create an [auth0](https://auth0.com/) account & an [application](https://auth0.com/docs/applications). Make sure to add in allowed callback urls, allowed logout urls and allowed web origins (e.g. http://localhost:3000 for our test front-end).
- Enable Password Grant Type in the Advanced Settings of your Application.
- Go to your Auth0 Tenant Settings and make sure API Authorization Settings Default Directory is `Username-Password-Authentication`.

Then install dependencies

`$ npm install`

### 2. Create `secret.pem` file

This file contains the Auth0 public certificate, used to verify tokens.

- Create a `secret.pem` file in the root folder of this project.
- Paste the public certificate in there.

### 3. Deploy the stack

Deploy the stack using Serverless in order to consume the private/public testing endpoints.

```bash
$ sls deploy -v
```

### 4. Final test

To make sure everything works, send a POST request (curl / Postman) to the private endpoint.

- Obtain a test token from Auth0.
- Provide your token in the headers like so:

```
"Authorization": "Bearer YOUR_TOKEN"
```

## Cross-stack authorization

This is very useful in a microservices setup. For example, you have an Auth Service (this service) which owns anything auth/user-related, and a bunch of other services that require user authorization.

Easy to make your authorizer work anywhere else in your AWS account.

When defining Lambdas in other services, simply define the `authorizer` as well and provide the ARN of your `auth` function (found in the AWS Console or via `sls info`).

#### Example:

```yaml
functions:
  someFunction:
    handler: src/handlers/someFunction.handler
    events:
      - http:
          method: POST
          path: /something
          authorizer: arn:aws:lambda:#{AWS::Region}:#{AWS::AccountId}:function:auth0-auth-service-dev-auth
```

If everything was set up correctly, all incoming requests to your `someFunction` Lambda will first be authorized. You can find the JWT claims at `event.requestContext.authorizer`.
