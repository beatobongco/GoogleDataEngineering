## Serverless Data Analysis with Google BigQuery and Cloud Dataflow

side inputs - smaller datasets you need to join with your bigger datasets

dataflow - allows you reuse most of the same code from batch and stream 

bigquery
* way more effective when data is denormalized rather than in different tables
* charged by amount of data processed
* cost of bq storage is the same as cloud storage (!!)
* sharability -- just give someone access and they can run queries on it or view a view
* columnar -- no indexes, keys or partitions. Each column in separate, compressed, encrypted file
* if used on python, can return pandas DataFrame
* can use `IF(cond, iftrue, iffalse)`
* `LPAD` pads stuff on the left, useful
* `CONCAT` is useful too
