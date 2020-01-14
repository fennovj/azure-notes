# Connect your services together

## Messages vs events

A message contains either of the following two:

- A message contains data, e.g. a photo, a text input from user, etc. Metadata such as timestamp and url etc don't count.
- A message expects that the receiver does something for the basic integrity of the system. e.g. data that must be saved. For example, a 'delete user x' message does not contain data, but must be acted on by the receiver for the system to function.

An event does not contain either of the two. For example, server sends to client 'person x has come online'. The client/receiver may act on that knowledge (by showing a notification), but they don't have to, for the system to function.

## Why use queues

- Increased reliability - if system is in high demand, messages are simply stored instead of 'we sure hope it will eventually get processed'
- Message delivery guarantees - at least once/at most once/ FIFO
- Transactional support - we either want ALL or NONE of the messages to get processed at the same time - e.g. don't send the payment processor message, but drop the shipping message. (Reason for the name is because this is similar to database transactions)

### Messages: Queues

For messages, you want to use a queue, to ensure all data is kept before the message is processed, and no messages are 'forgotten'. There are two options:

- Azure Queue Storage - Save messages in storage account that can be accessed with REST api
- Azure Service Bus - Message Broker for enterprise applications. More 'Kafka-like'. Multiple communication protocols, have different data contracts, higher security requirements, and can include both cloud and on-premises services.

Choose Service Bus if you need

- At-Most-Once delivery guarantee, FIFO guarantee, transactional support
- Role-based-access to messages
- publish/consume batches of messages
- receive messages without polling the queue
- Smaller than 80GB is fine

Chooise queue storage if you need

- Basically something simple that doesn't need the above stuff
- easier to code
- A queue larger than 80GB (only depends on storage account limit)
- audit of all messages while in/out of the queue

IMPORTANT NOTE: service bus messages can be 256 kb in size, or 1MB if you have premium tier pricing. In storage queue, they are 64 kb maximum

Other note: a single VM is fine for very large workloads, as long as you don't have strict time requirements for how quick a message/event should get processed. e.g. if it can take a few hours/days for a message to get processed, peaks can be processed at night without autoscaling.

### Messages: Topics

A topic is similar to a queue but each message can have multiple subscriptions. Subscriptions can also filter the messages they are interested in. This is only supported in Service bus

Choose Service Bus (with Topics) if you need

- Multiple receivers for each message
- Receivers can filter which messages they want to get

### Messages: Relays

Relays are two-way communication. It's not temporary storage for messages, and also is not loosely coupled. However, it is part of Service Bus

## Events: Azure Event Grid

Use Event grid when

- You need something simple
- Receivers can filter events
- Many receivers for a single message
- Reliable - automatic retry policy
- Pay-per-event - no events no pay

General setup: We have one 'sender', and many possible receivers who might be interested in the event. The sender doesn't care how many receivers there are. The events can come from many sources such as Blob to handlers such as Functions.

It goes:  Event --> Topic (the endpoint where the event is sent) --> Event Grid (underlying serverless architecture) --> Event subscription (endpoint where you route events to) --> Event handlers

Events are jsons like:

```json
{
    "topic": string,
    "eventTime": string,
    "data": { custominfo }
    etcetera
}
```

### Event sources

- Azure subscriptions - e.g. when VM is created
- Container registry - when images are added/removed
- Storage accounts- when blob is added
- IoT Hub
- Custom eents- REST api in web app
- More: event hub, service bus, media services

### Event handlers

- Azure functions
- Webhook - basically then you can do anything else. e.g. call api.
- Logic apps
- Microsoft flow

## Events: Azure Event Hubs

Unlike Event Grid, however, it is optimized for extremely high throughput, a large number of publishers, security, and resiliency. It does stuff in addition as what event grid does.

- Partitions - buffers where events are saved, they stay there for 24 hours
- Capture - events can be saved to blob storage / data lake
- Authentication - only publisher with token can send events. Especially when you have untrusted publishers. (e.g. client apps)

If you don't need any of those three, just use Event Grid.

Partitions: Event hubs have 4 partitions by default. Partitions are like buckets within an Event hub, and act as a logical grouping.

## Service bus code

Example code for sending a message:

```csharp
const string ServiceBusConnectionString = "Endpoint=sb://xxx";
const string QueueName = "salesmessages";
queueClient = new QueueClient(ServiceBusConnectionString, QueueName);
var message = new Message(Encoding.UTF8.GetBytes("Hello world"));
await queueClient.SendAsync(message);
await queueClient.CloseAsync();
```

Example code for receiving a message:

```csharp
static async Task ProcessMessagesAsync(Message message, CancellationToken token)
{
    Console.WriteLine($"Received message! Body:{Encoding.UTF8.GetString(message.Body)}")
    // Remove the message from the queue
    await queueClient.CompleteAsync(message.SystemProperties.LockToken);
}

queueClient = new QueueClient("same as above...");
queueClient.RegisterMessageHandler(ProcessMessagesAsync, messageHandlerOptions);
await queueClient.CloseAsync();
```

You can also use Topics, just replace 'queueClient' with 'TopicClient' basically

## Queue storage code

Keep in mind you have to first create a storage account.

### Sending a message

```csharp
// Initialization
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();
CloudQueue queue = queueClient.GetQueueReference("newsqueue");
bool createdQueue = await queue.CreateIfNotExistsAsync();
// Send the message
CloudQueueMessage articleMessage = new CloudQueueMessage("Hello world");
await queue.AddMessageAsync(articleMessage);
```

### Receiving a message

```csharp
CloudQueue queue = "..." // Same as above
CloudQueueMessage retrievedArticle = await queue.GetMessageAsync();
string newsMessage = retrievedArticle.AsString;
await queue.DeleteMessageAsync(retrievedArticle);
```

## Azure Event Hubs

big data! millions of events per second!

Keep in mind you have to create a namespace first: `az eventhubs namespace create`. Get the access key with `az eventhubs namespace authorization-rule keys list --name RootManageSharedAccessKey` (wtf?)

Create an eventhub with `az eventhubs eventhub create --name $HUB_NAME --namespace-name $NS_NAME`.

Also create a storage account: `az storage account create --name $STORAGE_NAME --sku Standard_RAGRS --encryption blob`
