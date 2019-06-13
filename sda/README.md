## Serverless Data Analysis with Google BigQuery and Cloud Dataflow

Goal: deal with very large datasets in a declarative way with both BigQuery and Dataflow without servers and infrastructure

side inputs - smaller datasets you need to join with your bigger datasets

### dataflow 
* allows you reuse most of the same code from batch and stream 
* no-ops data pipelines for reliable, scalable data processing

### BigQuery
* petabyte scale
* way more effective when data is denormalized rather than in different tables
* charged by amount of data processed
* cost of bq storage is the same as cloud storage (!!)
* sharability -- just give someone access and they can run queries on it or view a view
* columnar -- no indexes, keys or partitions.
  * Each column in separate, compressed, encrypted file. 
  * Cost goes up as no. cols go up
  * Therefore better tall than wide
* if used on python, can return pandas DataFrame
* can use `IF(cond, iftrue, iffalse)`
* `LPAD` pads stuff on the left, useful
* `CONCAT` is useful too
* user-defined functions in JavaScript
* not good for transactional data
* you can use *views* to restrict access to your data (that's the only data people you share to can see), you can save this view in a new dataset as to have different access controls
* a view is live -- changes as parent table changes
* saving a table into a new table is called *materializing* (not live)
* can define Google Sheet as external table

### BigQuery reference architecture
![image](https://user-images.githubusercontent.com/3739702/59414176-a10c0680-8df3-11e9-8529-645dd12600d9.png)
