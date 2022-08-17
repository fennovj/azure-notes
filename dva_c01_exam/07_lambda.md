# Lambda

Lambda supports Synchonous and Asynchronous invocation. Basically, Synchronous only returns when it's done, Asynchronous will just return an acknowledgement and run in the background. You define the `InvocationType` only when you invoke the Lambda!

- RequestResponse - default, synchronous
- Event - Asynchronous. Built-in 2x retry.
- DryRun - Only verifies that parameters are correct and you have permissions

Generally, you can set this using the boto API, or with the `X-Amz-Invocation-Type` header.
Also, Lambda has its own version of a dead-letter queue for picking up failed invocations.