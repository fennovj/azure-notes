# Extract knowledge and insights from your data with Azure Databricks

## Spark notebooks

The simplest way to get started with databricks is with Apache Spark notebooks. You can use then for processing huge files, join them, live stream, and train machine learning models.
Notebooks have .dbc format.
Spark can readily handle petabytes of data given a sufficiently large cluster.

You can execute Python, R, Scala, SQL, bash, and databricks filesystem commands. In general, variables/memory are not shared between languages.
To create a pyspark dataframe: (basically has similar interface to pandas) but with LAZY evaluatioN!!!

```python
df = spark.range(1000).toDF("number")
display(df)
df.describe().show()
df.schema
pdf = df.toPandas()
```

`pyspark.ml` has many functions from sklearn. You can also do pagerank with `graphframes` library.

Marketing: you can use it for

- Querying, exploring, and analyzing very large files and data sets
- Joining data lakes
- Machine learning and predictive analytics
- Processing streaming data
- Graph analytics
- Overnight batch processing of very large files

But not for updating individual records, waaay too much overhead for that.

## Read and write

- query large files (RDDs)
- Joins/aggregations
- access blobs
- data lakes
- key vault secret scopes

```python
from pyspark.sql.functions import year
display(peopleDF
  .select("firstName","middleName","lastName","birthDate","gender")
  .filter("gender = 'F'")
  .filter(year("birthDate") > "1990")
)

from pyspark.sql.functions import col
dordonDF = (peopleDF
  .select(year("birthDate").alias("birthYear"), "firstName")
  .filter((col("firstName") == 'Donna') | (col("firstName") == 'Dorothy'))
  .filter("gender == 'F' ")
  .filter(year("birthDate") > 1990)
  .orderBy("birthYear")
  .groupBy("birthYear", "firstName")
  .agg(count("birthYear").alias("total"))
  .withColumnRenamed("fromName", "toName") # rename col
  .withColumn("absoluteBirthYear", abs(col("birthYear")))  # Assign
)
```

Temporary view is a spark object that you can query from SQL, but only exists during this spark session.

```python
myCustomDf = ...
myCustomDf.createOrReplaceTempView("MyView")
spark.sql("SELECT * FROM MyView where firstName = 'Donna'")
```

Aggregations and joins

```python
from pyspark.sql.functions import min, max
salaryDF = peopleDF.select(max("salary").alias("max"), min("salary").alias("min"), round(avg("salary")).alias("averageSalary"))

salaryDF.show()
joinedDF = leftDF.join(rightDF, col("a") == col("b"))  # Inner join
```

Reading data

```python
myDF = spark.read.parquet('dbfs://mnt/x.parquet')
myDF = spark.read.option("inferSchema", "true").option("header","true").csv('/mnt/x.csv')

# Mounting blob storage needs SAS token
dbutils.fs.mount(
  source = "wasbs://%s@%s.blob.core.windows.net/" % (ContainerName, StorageAccount),
  mount_point = "/mnt/datafolder",
  extra_configs = {"fs.azure.sas.%s.%s.blob.core.windows.net" % (ContainerName, StorageAccount) : "%s" % SasKey}
)
```

You can also do hierarchical data like json

```python
data = spark.read.option("inferSchema", "true").option("header","true").csv('/mnt/x.json')
data.select("dates.createdOn", "dates.publishedOn")  # hierarchial column select

## parse array columns
data.select(size("authors").alias("num_authors"))  # length
data.select(col("authors")[0].alias("primaryAuthor")))  # index
from pyspark.sql.functions import explode
data.select(explode(col("authors")).alias("author")))  # Will duplicate rows to fit all the authors into the 'author' column
```

DATA LAKE

TODO THIS IS MISSING!!!

You can also get secrets. To do this, create a secret scope in datapricks, where you name the scope, and configure which key vault it is.

```python
mysecret = dbutils.secrets.get(scope = "scopename", key = "sql-password")
print('Password=' + mysecret)
# will print: Password=[REDACTED]
```

Write to CosmosDB

```python
writeConfig = {
"Endpoint" : uri,
"Masterkey" : key,
"Database" : "demos",
"Collection" : "documents",
"Upsert" : "true"
}
YorkDF.write.format("com.microsoft.azure.cosmosdb.spark").mode("overwrite").options(**writeConfig).save()
```

## Machine learning

Basically you can just use pandas/sklearn, its auto-spark enabled

Tensorflow is not parallelized, you need sparkdl (spark-deep-learning)

```python
from sparkdl import TFTransformer
from sparkdl.graph.input import TFInputGraph
import sparkdl.graph.utils as tfx

graph = tf.Graph()
with tf.Session(graph=graph) as session, graph.as_default():
    _, _ = build_graph(session, w0)
    gin = TFInputGraph.fromGraph(session.graph, session,
                                 ["input_tensor"], ["output_tensor"])

transformer = TFTransformer(
    tfInputGraph=gin,
    inputMapping={'inputCol': tfx.tensor_name("input_tensor")},
    outputMapping={tfx.tensor_name("output_tensor"): 'outputCol'})

odf = transformer.transform(sdf)
```
