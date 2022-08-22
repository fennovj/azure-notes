# Lambda

Lambda supports Synchonous and Asynchronous invocation. Basically, Synchronous only returns when it's done, Asynchronous will just return an acknowledgement and run in the background. You define the `InvocationType` only when you invoke the Lambda!

- RequestResponse - default, synchronous
- Event - Asynchronous. Built-in 2x retry.
- DryRun - Only verifies that parameters are correct and you have permissions

Generally, you can set this using the boto API, or with the `X-Amz-Invocation-Type` header.
Also, Lambda can forward to dead-letter queue for picking up failed invocations, this has to be either SQS or SNS.

Maximum execution time is 15 minutes.

## Scheduling

If you want to schedule Lambdas to run e.g. every half hour, use Cloudwatch events. Cloudwatch has a Cron functionality, and can trigger lambdas.

## Concurrency

By default, the quota for Lambda is 1000 concurrent executions per account (for all functions combined). You can increase this, but AWS will only approve if you have a legitimate usecase.

You can also 'Reserve' concurrency for a single function - e.g. if a function 'reserves' 300, that means the other functions only have 700 left. *You are only allowed to reserve 900 in total*, aka you must always have 100 unreserved left per account.

There is a question of how much 'lambda instances' can you start up right away? e.g. if you have 1000 calls right away, can you answer them? The answer is yes, since there is a `burst concurrency limit` of 3000. This means you can answer 3000 right away, and after that, lambda can scale up with 500 a minute. Of course since by default the quota is 1000, for most people this burst limit is irrelevant. However:

- If you increase your quota, the burst limit stays at 3000 so you may get quota issues when usage increases suddenly
- For other regions, the burst limit is lower: e.g. in Frankfurt it's only 1000, and in some smaller regions it's only 500.

## CPU

CPU for a lambda is always proportial to memory. The pricing page is priced in `GB-Second`, meaning price for an instance with 1GB memory to run for 1 second. Of course, the price of 2 instances with 500MB memory would be the same.

Default (and minimum) is 128MB. AWS is unfortunately very opaque about how much CPU you are getting. However, you can safely assume that CPU linearly scales 1-to-1 with memory.

## Layers/Runtimes

Lambdas contain:

- Function: the main 'function' that is invocated
- Runtime: The 'base' of the image. This can be e.g. `python3.8` or `nodejs16.x`. You can also provide your custom runtime in a layer. The built in runtimes are: Python, Node, Ruby, Java, Go, .NET.
- Layer: A way to package libraries. The idea is you create a layer ahead of time, to make it faster/easier to deploy code. A layer can contain libraries, or a custom runtime.

## Lambda@Edge

You use this together with Cloudfront. The idea is to run 'closer to users', which can speed up the response and take load off the main backend, e.g. prevent 504 errors when your servers are overloaded.

The price is 3 times as much as regular lambda. So still fairly cheap, especially this is really only for functions with execution time in the milliseconds.
(e.g. 10 million requests, running 100ms each is 12 dollars)

- Viewer request: run when request received
- Origin request: run before forwarding to origin
- Origin response: run when response received
- Viewer response: run before returning reponse

## Environment variables

environment variables are set on a 'version'. The main goals are:

- Allow code changes without having to deploy an entirely new version, meaning easier deployments
- Prevent hardcoded stuff, like bucket or secret names. Makes it easier to e.g. do multitenancy.

Passing 'parameters' to a lambda is different, this is done in the 'event'. E.g. if you pass the "foo" parameter, you can access that in the lambda by running `event["foo"]` (syntax depending on language)
How you pass the parameter depends on the method request/mapping template: API gateway has a way of defining parameters directly, based on the input request.

## Execution environment

Th execution environment is essentially the VM that lambda runs in. This is mainly important if you develop your own runtimes/extensions, since the environment will send 'INIT' and 'SHUTDOWN' signals to them. Essentially due to the serverless nature, these 'INIT' and 'SHUTDOWN' can happen at any time.