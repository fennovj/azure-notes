# Misc

## Architecture

<https://docs.microsoft.com/en-us/azure/architecture/data-guide/big-data/>

- Lambda architecture:
    - Speed layer --> Analytics client
    - Cold layer: Master data --> Serving layer --> Analytics client

The advantage is you combine both real-time (less accurate) insights with long-term insights. Downside is processing logic appears in two places, meaning duplicate logic. Kappa architecture is designed as an alternative where all data follows a single path

- Kappa architecture:
    - Long term layer ( An append-only immutable log, such as Synapse )
    - Speed layer ( Where data can be easily accessed from, such as CosmosDB )

Real-time logs go immediately into the speed layer (example: cosmosdb). For batch insights, you can replay the log in parallel to some batch processing. Batch results are eventually stored in the Speed layer for the analytics client to access.