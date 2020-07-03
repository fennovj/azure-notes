# Azure Stream Analytics

Stream analytics' main goal is to integrate IoT and gain insights in real time.

Firstly, a data stream is event data that is analysed by some other technology. We often want to detect issues, or trigger specific actions when a threshold is met.
Also, there is a difference between on-demand and live streaming data. On-demand is persisted in storage, and queries when convenient. Live data is not stored, but it requires constant computation power to generate insights.

Streaming analytics is the 'middle man' in the typical event pipeline: source -> ingestion -> processor -> destination

- Source is something like a sensor or application.
- Ingestion is an event hub, iot hub, or blob storage (blob storage mainly for static, not streaming data)
- Analytics is done in Stream Analytics, in 'SAQL' stream analytics query language. SAQL is a subset of T-SQL.
- Destination is something like Azure SQL, Data Lake, or PowerBI

Also, stream analytics guarantees exactly-once processing, and at-least-once delivery, so events are never lost. (including recovery capabilities in case of failure)

## Demo: transforming data with ASA

Generally, an ASA job has a source, an SQL query (that transforms it), and a sink. The source is like the ingestion above, can be event hub, iot hub, or blob storage.

You can define this inputs/output/query in the 'job topology'.
Specifically for a blob input, you can define a pattern like logs/{date}/{time}/{partition}, and a partition for performance (optional)

For the output, there are many options, like event hub, sql, datra lake, synapse, service bus, cosmosdb, powerbi, azure function, blob storage.
You can either stream to the output, or batch. So for example, write all new rows to output every x minutes, or write every row individually (or in batches of 100 rows or something like that). For obvious reasons, there are performance tradeoffs.

A Query looks like:

```sql
SELECT
    City,
    Coordinates.Latitude,
    Coordinates.Longitude
INTO streamoutput
FROM streaminput
```

In this example, streamoutput and streaminput are aliases for the input/output streams. In this case, the event is a JSON file, so City, and the other two, are JSON elements. You can test the query in the portal, which shows the table output (and you can download it as a JSON)

Finally, start the job! (it will run constantly in the background). In this case, because we have a blob output, it will create JSON files by default.
