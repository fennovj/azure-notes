# other

## Logic apps

The main strength of Logic Apps is the hundreds of pre-built components, such as Dropbox, Salesforce, SAP, Youtube, Azure Functions, Office 365, Slack, etcetera

Example: New tweet about your product: detect sentiment, if positive -> store link in database, if negative -> email customer service.

You can also write custom connectors, as long as the service has a REST api. You create an OPEN API file to describe the API, then upload and create a custom connector.

Use Logic apps when:

- You need to integrate multiple applications and systems
- You want scalability, but not neccecarily subsecond response time
- You don't need highly complex conditional logic, Azure Functions is better for that
- You have connectors, or are prepared to take a few hours to write a custom connector

Example: Social Media monitor is a perfect fit, integrates email/database/twitter.

Another example is checking daily if there are old videos to archive. Bit more complex, and not as much integration, but still logic apps will work nicely

Online sales process that alerts paymentprocessor/suppliers/shippers - will likely involve complex business logic based on which item/amount of item/location, and different shippers might all ave different methods of being alerted, meaning writing custom APIs for all of them. Logic apps could work, if we had the resources to write all the custom connectors.