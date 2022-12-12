# SparkStructuredStream

More than a parallelism framework, Apache Spark is fully designed for data processing purposes. Probally its best function is the Spark Structured Stream, a way for realtime data processing, with capabilities of reading data from many popular tools like Apache Kafka, AWS S3 and saving again in anywhere that you want.

This repository is just a simple example of using it, I already worked with it in very interesting real case cenarios, but I cannot share any of those here for platform confidentiality reasons.

## Real time data ingestion using SparkStructuredStream

Let's say that you have a path into AWS S3 that your data arrive for your data ingestion:

```
s3://my-beauty-bucket/some/prefix/new-data/
```
