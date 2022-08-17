# API Gateway

To start, a REST API is a collection of endpoints connected to resources. These resources can be custom HTTP redirects, Lambdas, or other services. API Gateway adds a bunch on top like Access Control, and ___

## Development

### Define resources

### Access control

### Integration request/response

Integration request is a request that the API Gateway submits to the backend. E.g. you can transform incoming requests. Example: User requests `apigateway.com/q=123`, you sent a POST request to `domain.com` with body `{id: 123}`. API gateway can do those transformations. You can:

- Set up data mappings
- Define an IAM role with which to call backend services (e.g. for lambda)
- Set up a proxy resource (group multiple endpoints into one, and the endpoint simply becomes another parameter)

You can also set up an integration response, which means you take in the output, and you:
- define HTTP Code
- define response body, using data mappings or regular expressions
- Define headers

There are multiple integration types:

- AWS - Set up an AWS Service Action. Most commonly used for invoking Lambdas. You still set up data mappings yourself.
- AWS_PROXY - invoke AWS (especially lambda), but do all the data mapping automatically. Lambda receives a JSON with all the HTTP Info it could need (headers, query string, etc) <https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-input-format>, and the response is automatically returned.
- HTTP - Call a custom HTTP endpoint, you must set up all data mappings yourself
- HTTP_PROXY - Custom HTTP endpoint, but this straight up forwards the requests/responses. Best if you are just using API Gateway for authentication/staging.
- MOCK - Returns a response without actually sending the response to the backend. Used e.g. in a test environment where you don't want to set up a full backend.

Another note: for lambda proxies specifically, lambda *must* return a response of the form:

```
{
    "isBase64Encoded": true|false,
    "statusCode": httpStatusCode,
    "headers": { "headerName": "headerValue", ... },
    "body": "..."
}
```

Otherwise, you cannot use `AWS_PROXY` with Lambda, and you will get 502 (bad gateway) errors.

## Deployment

After creating a REST Api (which means it has an ID), you need to deploy it, this goes in two steps: Create a deployment, then set the stage. imagine the ID is `123`:

`aws apigateway create-deployment --rest-api-id 123 --region eu-west-1`

`aws apigateway update-stage --rest-api-id 123 --region eu-west-1 --stage-name production` (this will also auto-create the stage if it doesn't exist yet)

Stages also have certain stage-specific settings:
- Cloudwatch logging
- X-ray tracing
- Variables (that can be referred to in the rest-api definition)
- API Cache, Throttling, Web ACL, Client certificate (used by API Gateway to call downstream services)
- Canary settings

### API Cache

This is like a built-in cache, that's 500MB by default, that keeps the latest responses. Main goal is to reduce backend calls (this is not like cloudfront where it's geographical caching) You can define:

- TTL: 300 seconds by default, maximum is 3600
- Capacity: 500MB by default, only thing that affects the price
- Per-key cache invalidation - This allows client to use the `Cache-Control: max-age=0` header. This is done on an IAM Role basis (`execute-api:InvalidateCache` action), so user must be authorized with AWS credentials. Unauthorized request can either get a 403, or you can just ignore the header.

## Canary deployment

When you update a stage to add a Canary, you add a new release, and there will be a `substage` named `{stage-name}/Canary` that traffic will be redirected to. Settings:

- Percentage - How much traffic is redirected to Canary
- Stage variables - you can override variables
- Stage Cache - if you want to disable the stage cache or not for the canary (if you don't want canary users to get old cached responses)

Generally, when the canary succeeds, you would then 'Promote' the canary, which will create a deployment that shifts the remaining traffic. You can also 'delete' the canary if you are getting errors.