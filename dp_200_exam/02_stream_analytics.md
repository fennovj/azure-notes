# Stream analytics

SA uses some kinda SQL-like language. Here are some of the common functionalities.

Generally, the input/output is treated as json. The input is usually a list of events. ASA takes care of putting events together. For example, you can set it to give you the events every 5 minutes, or 1000 events, whichever comes sooner.

Stream Analytics automatically makes sure you have the correct events available. For example if you aggregate over 10 minutes, it will store the last 10 minutes of events. You have to give limits where needed, for example: comparing to the last event is not possible  because the last event may be hours/days ago, so you have to give a limit.

A simple example query is:

```sql
SELECT
    City,
    Coordinates.Latitude,
    Coordinates.Longitude
INTO streamoutput
FROM streaminput
```

Note the 'INTO' and 'FROM' defining the input/output streams. Some things you can do:

- Define a timestamp on your data and order by it: `FROM streaminput TIMESTAMP BY Time`. Now in the query, you can do `Time`, `minute`, `second`, etcetera
- Filter which inputs are sent to the output with a 'WHERE' clause. For example: `WHERE City = 'Utrecht'`.
- Compare to past events, e.g. `WHERE LAG(City, 1) OVER (LIMIT DURATION(minute, 1)) <> City` will filter elements where the city is different from the previous city
- Aggregation, for example: `SELECT COUNT(*) FROM Input GROUP BY City`
- more, see below

## Window functions

- Tumbling: simplest. Example: 0-10 sec, 10-20 sec, 20-30 sec, etcetera. Each event is only in 1 window. `GROUP BY TumblingWindow(time, 10)`
- Hopping: Similar to above, but with overlap. Example: 0-10, 5-15, 10-20, 15-25, etcetera. `GROUP BY HoppingWindow(time, 10, 5)`
- Sliding: New window everytime an event enters or exits the frame. So for example, if the max duration is 10, and events are at {1: 8, 2: 12, 3: 19}, then the windows would be: [1], [1, 2], [2], [2, 3], [3]. `GROUP BY SlidingWindow(time, 10)`
- Session: groups events closer than x seconds apart, with a max duration of y seconds. That means after each event, it will check if another event happens within x seconds. If so, the window continues.
The max duration will check every y seconds if it has been reached, and will cut off the window. Example, if x = 5, y = 10, and the window has started at 16, with events at 19, 23, 26, 28, 31, etcetera: the max window will check at 20 (max duration not met), and again at 30 (max duration met), and the window will end at 28. `GROUP BY SessionWindow(time, 5, 10)`
- Snapshot: groups events with the same timestamp. Not a window function but just a normal grouping. `GROUP BY System.Timestamp()`

## Returning last element in window only

First, calculate the last event in each window.

```sql
WITH LastInWindow AS
(
    SELECT
        MAX(Time) AS LastTime
    FROM
        streaminput TIMESTAMP BY Time
    GROUP BY
        TumblingWindow(minute, 10)
)
```

This is also an example of 'iterative' programming. We first define a 'temp table' LastInWindow. Then, inner joining on this table acts as a filter.

```sql
SELECT
    City
FROM
    streaminput TIMESTAMP BY Time
    INNER JOIN LastInWindow
    ON DATETDIFF(minute, streaminput, LastInWindow) BETWEEN 0 AND 10
    AND Time = LastInWindow.LastTime
```

Here, DATEDIFF(minute, x, y) returns the difference in time between x and y. Note that above, both have `BY Time` defined in their inputs.

## Partition keys

You can partition the data, for example if you want to group separately for each user_id. It generally looks like `SELECT x FROM y GROUP BY Grouper(...) OVER (PARITITON BY user_id)`

Importantly, partitioning allows Stream Analytics to use more streaming units. In general, we can use 6 streaming units per partition. Furthermore, each nonpartitioned step runs not parallel, so uses the same 6 streaming units.

So for example, if we have 2 non-partitioned steps, and 1 step with 10 partitions, we can use 6 + 60 = 66 streaming units.

## Other Built-in functions

This was added to the exams in September 2020. [List is found here](https://docs.microsoft.com/en-us/stream-analytics-query/built-in-functions-azure-stream-analytics).

### Aggregate functions

Aggregate are functions like MAX, COUNT, AVG, PERCENTILE_DISC (discrete), etcetera. You either aggregate a GROUP BY (for example with windowing function), or use it with an OVER statement. The syntax is like:

```sql
SELECT
    MAX (Colname)
FROM input TIMESTAMP by Time
GROUP BY OtherCol, TumblingWindow(minute, 3)
```

Or:

```sql
SELECT
    AVG(temperature) OVER (PARTITION BY id LIMIT DURATION (minute, 5))
FROM input TIMESTAMP by Time
```

### Analytic Functions

These functions calculate/analyse something, and are essentially scalar functions. Examples:

- IsFirst - is the event the first in a given interval. Example: `SELECT name, ISFIRST(day, 1) FROM input`, to see for each name if they were the first event that day.
- CollectTop - Returns an array of top events, so `CollectTop(2)` would be an array of two events. Requires to specify an ordering like `SELECT CollectTop(2) OVER (ORDER BY x ASC, y DESC) as result from input`

### Array functions

For parsing arrays in JSON. You can get an array element (with integer index), or get the length.

### Conversion functions

CAST, TRY_CAST (will return NULL if the cast fails), GetType. Speak for themselves.

### Datetime/math/string functions

Basically all scalars (just like the conversion functions above). Stuff like get YEAR/DAY/MONTH from a date field, or DATEDIFF two dates. POWER/SQUARE/FLOOR/SQRT/etc for the math functions, and LOWER/SUBSTRING/REPLACE/REGEXMATCH/etc for the string functions.

## User defined functions/aggregates

You can write your own functions and aggregates in Javacsript. You can also add Azure ML User defined functions, for example to apply a model to incoming data.

- User defined Functions are stateless and scalar functions. The return value can only be a scalar. You cannot call out external sources or endpoints in a UDF. You cannot aggregate multiple records. Stateless means that every record is processed independently from each other.
- User defined Aggregates are stateful, meaning you can make your own window operations. They are a class with the following functions:
    - `init` function that initializes a state
    - `accumulate` function that gets some record data and timestamp, and updates the state
    - `computeResult` function that returns a (scalar) value based on the state
    - OPTIONAL: `deaccumulate` gets some record data and timestamp, and removes that record from the state. Used with sliding/session window when an event leaves the window
    - OPTIONAL: `deaccumulateState` - Recalculate current state based on state that is leaving. Used with hopping window.

You can use them the same way as you use other aggregates in a GROUP BY statement.

Both can be used for complex business logic, but the User Defined Aggregates can do more: it can implement complex *stateful* business logic.

## Parallelization

Partitioning is the most obvious way of parallelizing Stream amalytics. If your input is partitioned (for example Event Hubs, Cosmos, IoT Hub), it is highly recommended to partition in SA as well, this makes SA read from these partitions in parallel as well.

For embarassingly parallel jobs (this means: one input partition -> one output partition), set the same PartitionKey for the input and output. `PARTITION BY PartitionId` in ALL steps of the query. The number of input/output partitions must be the same. For compatibility level 1.2 (whatever that means), you can set the Partition key in the settings, and it will be parallelized automatically, otherwize you need to add the PARTITION BY

The max number of streaming units is 6 per partition, plus 6 for all non-partitioned steps.

For high throughput in a large application, you really need some form of embarassing parallelism, for example splitting on geography or user_id. The most efficient output for a SA is Event Hub, which supports about 19 trillion events per day at the maximum of 192 streaming units.

## Reviewing bottlenecks

- Review Input/Output events for throughput and "watermark delay" or "backlogged events"
    - Watermark delay - The 'total' time, meaning: from when the event actually happens, to when the processed event is outputted by Stream analytics. This includes latency introduced by uploading the event to IoT Hubs/etc, and the actual time taken by Stream Analytics. It can therefore be a more 'complete' measure that identifies issues everywhere. It can be used to identify when SA is not keeping up with high input (among other things)
    - Backlogged events - If too many events are coming in, some will get backlogged.

For event hubs: look for 'Throttled request' to see if SA is outputting too fast. For Cosmos DB, just look at the RU to see if SA is using too many RU's. For SQL, check IO and CPU usage.
