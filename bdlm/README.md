## Google Cloud Platform Big Data and Machine Learning Fundamentals

Compute, network, storage - foundational requirements for computing

Elastic cloud

- current iteration of how we do computing - "third wave managed service"
- pay for what you use
- unlike creating and managing VMs yourself, use emphemeral resources to run jobs

Why do it this way?

- access to innovation and infrastructure of big players like Google at a pay-per-use basis
- leverage enormous amounts of compute in a short time without owning machines and justifying annual cost and depreciation
  (simply good economics)

Why use cloud services?

- change where you compute - dont need to manage your infrastructure (No Ops -- minimize systems administration overhead), pay for what you use
  - a great example is VFX rendering - massive compute when needed, nothing at rest
- improve scalability and reliability - use Google's infra
  - financial systems -- terabytes per hour
- change how you compute - leverage better tech/algorithms to do things that were not possible before. New "powers" if used
  correctly can result in transformational innovation in your own business

preemptible VM - pay a discounted price but when someone who can pay full-price comes along, lose the machine.
Useful for fault-tolerant systems or for running big jobs

### Cloud Storage

- persistent blob storage, staging ground for import to other Google cloud products
- usually the first step of data lifecycle, use `gsutil cp`
- `gs://some_address` is just a key to a blob. Not a real folder structure
- accessible via REST api
- can use as a CDN -- durable, persistent, edge-cached

Projects - for payments - which billing account is going to be responsible for paying

Zones - use many in same region to reduce service distruptions. If a Google datacenter goes down you can still access

Regions - use many for a global application

`mpl_toolkits.basemap` is pretty cool for quick viz of a map

`gsuitl acl` - controls access control lists. Do stuff like give everyone access to your cloud storage bucket

### Compute Engine

- think of as fungible - able to replace or be replaced by another identical item

### Cloud SQL

- good for gigabytes of relational information
- auto replication, backups, fast connection
- there are random failures sometimes, if this happens try creating in another region
- you can configure network on creation

### Cloudshell

VM with devtools installed and 5GB persistent with some [limitations](https://cloud.google.com/shell/docs/limitations)

### Dataproc

Hadoop - map reduce framework

Pig - higher level scripting language for Hadoop

Hive - SQL-like interface to query data stored in highly distributed file systems (HDFS)

Spark - analytics engine built on Hadoop

Dataproc - google-managed hadoop, pig, hive, spark programs

Insights from the activity

- To run jobs that ingest data from a db, you have to [authorize](https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/CPB100/lab3b/authorize_dataproc.sh) dataproc to access the db
- You upload your [ML script](https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/CPB100/lab3b/sparkml/train_and_apply.py) to a GCS bucket and run it via dataproc -> jobs -> submit jobs

### Datastore

- transactional support for heirarchical data
- good for terabytes of heirarchical data
- Persistent hashmap
- filter objects on attributes/property
- returns iterable (think generators in Python) so for big queries doesn't take up all our memory

### Bigtable

- petabytes of **flat** data ex. sensor data, high frequency stock trading
- tables must look tall and narrow
- key-value HBase API
- append-only
- updates append a new row
- high throughput
- only get by key

### Bigquery

- serverless data warehouse
- run SQL queries on massive amounts of data via API call (GB -> petabytes)
- ~500 GB in 12 seconds
- each "instance" is a table
- need to prefix with `#standardSQL`
- can do `JOIN`s with Google Sheets

My favorite offering so far. There's a public HN dataset that I can use for mining for knowledge hehe there are also many nice public datasets, like picture in the [Met](https://bigquery.cloud.google.com/table/bigquery-public-data:the_met.images?pli=1&tab=preview)

```sql
#standardSQL
SELECT id, timestamp, title, url, score FROM `bigquery-public-data.hacker_news.full`
WHERE title IS NOT NULL and title != "" and timestamp > '2019-01-01'
ORDER BY score DESC LIMIT 100;
```

Three ways of loading data

- Files on disk on Cloud Storage
- Stream data w/ dataflow -- run queries while data is streaming in
- Federated (aggregate data fro disparate sources) data source (CSV, JSON, AVRO, Google Sheets)

### Datalab

- Jupyter notebook that handles auth to GCP
- `datalab create <name> --machine-type n1-highmem-8 us-central1-a`
- has HTML support!
- [Different](https://stackoverflow.com/questions/50340602/google-colaboratory-vs-google-datalab-how-are-they-different) from colab
- quite powerful when used together with BigQuery

```sql
%bq query -n taxiquery
WITH trips AS (
  SELECT EXTRACT (DAYOFYEAR from pickup_datetime) AS daynumber
  FROM `bigquery-public-data.new_york.tlc_yellow_trips_*`
  where _TABLE_SUFFIX = @YEAR
)
SELECT daynumber, COUNT(1) AS numtrips FROM trips
GROUP BY daynumber ORDER BY daynumber
```

```py
query_parameters = [
  {
    'name': 'YEAR',
    'parameterType': {'type': 'STRING'},
    'parameterValue': {'value': 2015}
  }
]
trips = taxiquery.execute(query_params=query_parameters).result().to_dataframe()
```

### Tensorflow

- ML lib w/ C++ backend for GPU
- ML advice: use the [simplest model](https://playground.tensorflow.org/#activation=tanh&batchSize=10&dataset=circle&regDataset=reg-plane&learningRate=0.03&regularizationRate=0&noise=0&networkShape=3&seed=0.60261&showTestData=false&discretize=false&percTrainData=50&x=true&y=true&xTimesY=false&xSquared=false&ySquared=false&cosX=false&sinX=false&cosY=false&sinY=false&collectStats=false&problem=classification&initZero=false&hideText=false) that solves the problem.
- Note that we don't need [feature engineering](https://playground.tensorflow.org/#activation=tanh&batchSize=10&dataset=circle&regDataset=reg-plane&learningRate=0.03&regularizationRate=0&noise=0&networkShape=&seed=0.60261&showTestData=false&discretize=false&percTrainData=50&x=false&y=false&xTimesY=false&xSquared=true&ySquared=true&cosX=false&sinX=false&cosY=false&sinY=false&collectStats=false&problem=classification&initZero=false&hideText=false)

General ML advice

- Make sure your data is numerical
- One-hot encode categorical data
- Do I have 5-10 examples of this data? If not, drop it. Very specific data causes overfitting
  ![image](https://user-images.githubusercontent.com/3739702/52988875-fafb5180-343b-11e9-95ca-8a4e058d5bca.png)
- the more columns you use as input features, the more rows you need to have in your training dataset
- for smaller data, it's better to use simple ML like linear regression. It's more explainable too

### CloudML Engine

NoOps, distributed tensorflow

### Google ML APIs

Use APIs Google has provided (and models trained on Google's data) for solved problems. This can be a part of (or simplify) your pipeline.

- label detection - classify things into broad categories (transportation, animal, flowers)
- face detection (+emotional state, headware)
- OCR + language support
- explicit content detection
- landmark detection (popular natural or man-made landmarks)
- logo detection
- can do image matching (API returns URLs of sites and of similar images)
- translation API
- natural language API (entity and sentiment)
- speech-to-text
- vision API - detects labels and gives you the time when they showed up

Oh my god, JSON API
![image](https://user-images.githubusercontent.com/3739702/52990023-216fbb80-3441-11e9-9d93-c23e9e02f223.png)

Super easy to use https://github.com/beatobongco/GoogleDataEngineering/blob/master/bdlm/notebooks/mlapis.ipynb

### Pub/Sub

No-ops serverless global message queue

Async processing

- common design pattern for highly available systems, load balancing, reduce coupling
- Instead of immediately processing, send these to a queue so it doesn't crash the system

### Dataflow

- Apache beam
- no-ops data pipelines in Java and Python
- you can use it over dataproc if your needs fit
  ![image](https://user-images.githubusercontent.com/3739702/53143331-7858ca80-35d2-11e9-9667-16be697e718e.png)
