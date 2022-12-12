# SparkStructuredStream

More than a parallelism framework, Apache Spark is fully designed for data processing purposes. Probally its best function is the Spark Structured Stream, a way for realtime data processing, with capabilities of reading data from many popular tools like Apache Kafka, AWS S3 and saving again in anywhere that you want.

This repository is just a simple example of using it, I already worked with it in very interesting real case cenarios, but I cannot share any of those here for platform confidentiality reasons.

## Real time data ingestion using SparkStructuredStream

Let's say that you have a path into AWS S3 that your data arrive for your data ingestion, tThe first step is to define the structured schema of the data, we'll do it using the StrucType class:

```python
import pyspark.sql.types as T

schema = T.StructType([
  T.StructField("id", T.StringType(), True),
  T.StructField("name", T.StringType(), True),
  T.StructField("address", T.StringType(), True),
  T.StructField("email", T.StringType(), True),
  T.StructField("birthdate", T.StringType(), True),
  T.StructField("created", T.StringType(), True),
])
```

The Spark Structured Stream will always need a pre defined schema. In this next step, we have to create the stream instance:

```python
stream = spark.readStream.format("json").schema(schema).load("s3://my-beauty-bucket/some/prefix/new-data/")
```

Spark will read all files that get into this path, but it will keep a checkpoint for wich file it aready readed and wich not, in other words, in the first run, the spark will read all existing files in the path, and for the other runs, it will only read new files, as it know that.

In this case, we'll define a function for processing and saving the data, as the Spark start many parallel jobs into the cluster, this function will be executed in microbatches. Definition:

```python
def microbatch(df, epoch):
  df.write.mode("append").format("parquet").save("s3://datalake-bucket/bronze-layer/my-table/")
  return
```

For the Spark be able to use its checkpoint resource, we need to define a path for its metadata be stored. In this case, it will be:

```
s3://datalake-bucket/configs/bronze-layer/my-table/checkpoint/
```

After that, we can start the stream:

```python
stream
    .writeStream \
    .option("checkpointLocation", "s3://datalake-bucket/configs/bronze-layer/my-table/checkpoint/") \
    .option("maxFilesPerTrigger", 100) \
    .foreachBatch(microbatch) \
    .queryName('My table stream (bronze layer)')
    .start()
```

Notice that an other option has been setted: maxFilesPerTrigger. This option is important for limit the number of files in each batch, in this case we used 100, but it need to be tuned as the files size will always be diferent from case to case.

After that, our data stream is running:
![image](https://user-images.githubusercontent.com/68759905/207071149-03e3f726-73f3-4860-8b2e-2cbfdd512fe2.png)

## Example full script:

```python
import pyspark.sql.types as T

schema = T.StructType([
  T.StructField("id", T.StringType(), True),
  T.StructField("name", T.StringType(), True),
  T.StructField("address", T.StringType(), True),
  T.StructField("email", T.StringType(), True),
  T.StructField("birthdate", T.StringType(), True),
  T.StructField("created", T.StringType(), True),
])

stream = spark.readStream.format("json").schema(schema).load("s3://my-beauty-bucket/some/prefix/new-data/")

def microbatch(df, epoch):
  df.write.mode("append").format("parquet").save("s3://datalake-bucket/bronze-layer/my-table/")
  return
  
stream
    .writeStream \
    .option("checkpointLocation", "s3://datalake-bucket/configs/bronze-layer/my-table/checkpoint/") \
    .option("maxFilesPerTrigger", 100) \
    .foreachBatch(microbatch) \
    .queryName('My table stream (bronze layer)')
    .start()
```
