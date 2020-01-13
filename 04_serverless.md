# Create serverless applications

## Introduction

Serverless is FaaS - Function as a Service, aka microservice.

Advantages

- If function never runs, you don't pay (except for WebJobs)
- Stateless logic - instances are created/destroyed on demand
- Event driven, autoscales if event happens more than normal
- You can always take the code and deploy it in non-serverless if needed

Disadvantages

- Runtime is 5 minutes or maximum 10 minutes. If the function needs to give a HTTP response, it is restricted to 2.5 minutes. (Durable functions have no timeout)
- If the function gets called many times, a VM will probably be cheaper.

There are 4 general options:

- Logic Apps
- Microsoft Flow
- WebJobs
- Azure Functions

### Design-first

Logic Apps and Microsoft Flow have a drag/drop interface, but Flow is even more for 'non-programmers'. Flow is built on Logic Apps, but Flow is for 'business analysts'. Logic apps also has code editing, and can be included in Azure DevOps.

### Code first

WebJobs are hosted in Azure App service, and only support certain languages (including Python, Bash, Node.js, Powershell, .NET). However, the SDK only supports C#. You pay for the entire VM or App Service Plan.

Azure Functions supports many languages, has automatic scaling and Pay-per-use pricing. They are more flexible and simple. (also it's easier to call a Azure Function from Logic Apps)

However, WebJobs have two advantages: They can be inplemented in an existing App Service app, and you can more closely control the 'JobHost' object that listens for events that trigger the code. E.G. Retry policies when a failure occurs.

![Overview image](https://docs.microsoft.com/en-us/learn/modules/choose-azure-service-to-integrate-and-automate-business-processes/media/3-service-choice-flow-diagram.png)

In general, Azure reccommends a design-first approach when people want to understand the workflow without looking at code. Also, design-first is more 'self-documenting' - the design describes the process in a picture.

In general, code-first is better if tthere is no design-first requirement, especially if you need to communicate with custom databases/API's.

The rest of the section is mostly about Azure Functions! Jay!

## Serverless logic with Azure Functions

Functions are hosted in a **function app**, which logically group functions.

An azure function metadata is defined in `function.json`.

You will often choose the *consumption plan* instead of the *app service plan*, which is pay-per-second.

There are several trigger types, including:

- Blob storage - when new or updated blob is detected
- CosmosDB - insert or update
- HTTP - start with HTTP request
- Timer - start on a schedule
- Event Grid, Graph events, Queue storage, Service Bus

### Function key

You can make a function key, and pass it in the x-functions-key header for HTTP requests. Obviously this isn't always relevant, for example timer functions don't need authentication to run.

## Other triggers

### Timer trigger

They can either be defined as **timestamp parameter name**, or as **CRON expression**. Example CRON expression: `0 */5 * * * *` means every 5 minutes.

### Blob Trigger

You enter info about the blob trigger. You can also define stuff like `BLOBNAME/{name}.png`. That function will only trigger when a png is uploaded or updated.

## Bindings

Binds connect a function with other data/services. a function has an input binding and an output binding.

Note that an azure function can only have one trigger!! A *trigger* is a secial input binding that causes a function to execute. But it can have multiple input bindings to read from.

There are many bindings like blob storage, CosmosDB, Event Hubs, HTTP endpoints, etc.

A binding has a:

- name
- type
- direction: in or out
- connection: the name of the key/secret to use
- path: type-specific configuration

Example:

```javascript
    {
      "name": "headshotBlob",
      "type": "blob",
      "path": "thumbnail-images/{filename}",
      "connection": "HeadshotStorageConnection",
      "direction": "in"
    },
```

Specifically, when making a function in Javascript, your javascript code will look like:

```javascript
module.exports = async function (context, req) {
    // req contains the trigger binding
    // context.res contains the output binding
    context.res = {
        status: 200
        body: "hello " + req.query.name
    };
}
```

### Input bindings

There are input bindings such as: CosmosDB, Blob storage, Table Storage, SignalR.

There are output bindings such as: Blob storage, Table storage, Event Hubs, SendGrid, CosmosDB, SignalR, HTTP, Queue storage, Service Bus

In input bindings, an expression like `{name}` or `{id}` will appear, the *binding expression*. These will refer to GET/POST parameters etcetera. For example in Cosmos, this is what you filter on, and your partition key, etcetera.

### Output bindings

CosmosDB bindings such as creating new documents in Cosmos: you create a parametername such as `context.bindings.newdocument`, then go `context.bindings.newdocument = JSON.stringify({...}); context.done()`, and voila!

## Durable functions

Copied from website: "Durable Functions is an extension of Azure Functions that enables you to perform long-lasting, stateful operations in Azure."

Durable functions can retain state between function calls. This is to simplify complex stateful executions that would otherwise have to read/write to storage every 10 minutes to keep a state.

- They enable event-driven code (like normal functions)
- You can chain functions together
- You can orchestrate and specify the order of functions

You can call functions synchronously or asynchonously. Output is saved locally and can be used in future function calls.

### Function types

There are three:

- client - entrypoints in response to event.
- orchestrator - defines order and how actions are executed. C# or javascript
- activity - basic units that do actual work

Workflow patterns

- Function chaining - a sequence of functions in order
- Fan out/fan in - multiple functions in parallel, and wait for all to finish
- Async HTTP APIs - API with long-running operation, the API redirects to endpoint where client can poll status
- Monitor - recurring check, for example for a change in state
- Human interation - wait for some human interaction using timeouts and compensation if human fails to act in time. For example, approval process This is smart because human interaction can take a while, so durable functions can wait for a while for them.

### Make a durable function

First, you make a client function, then an orchestrator function, then some activity functions.

An orchestration function calls an activity with `callActivity`. You can also create timers with `createTimer`.

Here is an example of a timeout:

```javascript
const activityTask = context.df.callActivity("GetQuote");
const timeoutTask = context.df.createTimer(deadline.toDate());

const winner = yield context.df.Task.any([activityTask, timeoutTask]);
if (winner === activityTask) {
    // All pending timers must be complete or canceled before the function exits.
    timeoutTask.cancel()
    return true;
}
```

## Azure functions core tools

You can install azure core tools, then do `func init` to create a local azure function template, `func new` to create a function.

You can also run a function locally, by `func start`, then use curl or something to start the function.

You can publish with `func azure functionapp publish <app_name>`. Here, 'app_name' is the 'function app' in azure, not the local folder.

## Github webhooks

Webhooks are user-defined HTTP callbacks, called after some event like pushing code or updating a wiki page.

A webhook consists of:

- events. [reference here](https://developer.github.com/webhooks/#events)
- payload url, with content type: json or form
- secret: so not anyone with the payload url can trigger the event

## SignalR

Usually, 'detecting changes' means a polling-based design - fetching changes from server every x minutes

Drawback is that there is a delay between the change and when app becomes away of change. Obvious solution: smaller polling interval, but that means more overhead.

SignalR can be used with CosmosDB and Functions to Push changes to an app. We create a function that triggers on changes, with a persistent connection (thanks to SignalR)

## Publishig to static website

Azure Storage allows a feature where you can statically serve files from a storage container. `https://<ACCOUNT_NAME>.<ZONE_NAME>.web.core.windows.net/<FILE_NAME>`

Then, using an Azure Function to update those static files, you get an ever-changing website!

## Visual studio

I skipped this part. But basically, Visual Studio has a GUI to create functions (like core tools but gui), and then you can run them locally and test in the browser or with curl. You can also publish.

### Continuous deployment

Azure Functions integrates with BitBucket, Dropbox, GitHub, and Azure DevOps. This enables a workflow where function code updates made by using one of these integrated services triggers deployment to Azure.

```bash
az functionapp deployment source config-zip \
-g <resource-group> \
-n <function-app-name> \
--src <zip-file>
```

### Unit test functions

It's possible, visual studio has some integration.
