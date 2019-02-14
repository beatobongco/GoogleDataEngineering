# Google Data Engineering

Notes and code for the Google Data Engineering specialization on Coursera

## Google Cloud Platform Big Data and Machine Learning Fundamentals

Compute, network, storage - foundational requirements for computing

Elastic cloud
* current iteration of how we do computing - "third wave managed service"
* pay for what you use
* unlike creating and managing VMs yourself, use emphemeral resources to run jobs

Why do it this way?
* access to innovation and infrastructure of big players like Google at a pay-per-use basis
* leverage enormous amounts of compute in a short time without owning machines and justifying annual cost and depreciation 
(simply good economics)

Why use cloud services?
* change where you compute - dont need to manage your infrastructure (No Ops -- minimize systems administration overhead), pay for what you use
  * a great example is VFX rendering - massive compute when needed, nothing at rest
* improve scalability and reliability - use Google's infra
  * financial systems -- terabytes per hour
* change how you compute - leverage better tech/algorithms to do things that were not possible before. New "powers" if used
correctly can result in transformational innovation in your own business

preemptible VM - pay a discounted price but when someone who can pay full-price comes along, lose the machine.
Useful for fault-tolerant systems or for running big jobs

### Cloud Storage
* persistent blob storage, staging ground for import to other Google cloud products
* usually the first step of data lifecycle, use `gsutil cp`
* `gs://some_address` is just a key to a blob. Not a real folder structure
* accessible via REST api
* can use as a CDN -- durable, persistent, edge-cached

Projects - for payments - which billing account is going to be responsible for paying

Zones - use many in same region to reduce service distruptions. If a Google datacenter goes down you can still access

Regions - use many for a global application

`mpl_toolkits.basemap` is pretty cool for quick viz of a map 

`gsuitl acl` - controls access control lists. Do stuff like give everyone access to your cloud storage bucket

### Compute Engine
* think of as fungible - able to replace or be replaced by another identical item

### Cloud SQL
* good for gigabytes of relational information
* auto replication, backups, fast connection 
* there are random failures sometimes, if this happens try creating in another region
* you can configure network on creation

### Cloudshell 

VM with devtools installed and 5GB persistent with some [limitations](https://cloud.google.com/shell/docs/limitations)

### Dataproc

Hadoop - map reduce framework

Pig - higher level scripting language for Hadoop

Hive - SQL-like interface to query data stored in highly distributed file systems (HDFS)

Spark - analytics engine built on Hadoop

Dataproc - google-managed hadoop, pig, hive, spark programs 

Insights from the activity
* To run jobs that ingest data from a db, you have to [authorize](https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/CPB100/lab3b/authorize_dataproc.sh) dataproc to access the db
* You upload your [ML script](https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/CPB100/lab3b/sparkml/train_and_apply.py) to a GCS bucket and run it via dataproc -> jobs -> submit jobs

### Datastore

* transactional support for heirarchical data
* good for terabytes of heirarchical data
* Persistent hashmap
* filter objects on property
* returns iterable (think generators in Python) so for big queries doesn't take up all our memory

### Bigtable

* petabytes of **flat** data ex. sensor data, high frequency stock trading
* tables must look tall and narrow
* key-value HBase API
* append-only
* updates append a new row
* high throughput

### Bigquery

* serverless data warehouse
* run SQL queries on massive amounts of data via API call (GB -> petabytes)
* ~500 GB in 12 seconds
* each "instance" is a table
* need to prefix with `#standardSQL`

My favorite offering so far. There's a public HN dataset that I can use for mining for knowledge hehe

```sql
#standardSQL
SELECT id, timestamp, title, url, score FROM `bigquery-public-data.hacker_news.full` WHERE title IS NOT NULL and title != "" and timestamp > '2019-01-01' ORDER BY score DESC LIMIT 500;
```
