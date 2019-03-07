## Leveraging Unstructured Data with Cloud Dataproc on Google Cloud Platform

Structured data
* has a schema

Unstructured data
* no schema
* images, emails
* OR it could have a schema/partial schema but that schema is not useful for your analysis/query
* around 90% of data in enterprise is unstructured

### Dataproc
* Spark/Hadoop on the cloud
* MapReduce
* setting up clusters manually have a lot of overhead but Google does this for you
* You will only really need to care about your custom code
* `bdutil` is an open source solution
* Create clusters for specific jobs
* If storing data not on GCS, make sure its located near the compute nodes to avoid large egress charges
* cluster information at port 8088
* administration UI at 9870

Why cloud? 
* All boils down to cost. 
* Cheap storage. When data size goes to petabytes, cost soars
* You can quickly reach petabytes 200 servers logging 50 entries per second for 3 years
* Speed too, if you can parallelize, cut down job duration from days to minutes

Why use cloud storage vs HDFS?
* autoscaling
* shut down cluster when not processing a job
* first-class support

Google Compute Engine
* disk performance scales with the size of the VM.

### Preemptible VMs
* almost 80% cheaper but can be pulled back at any time when someone requests a persistent VM
* 50/50 split is good for large jobs
* use it to actually get job done faster AND cheaper
* never use with HDFS, use it with Google cloud storage
* if region is busy you probably cannot get any
* handled by dataproc automatically

You can find equivalent REST or command line commands for doing most of what you do on console.cloud.google.com

### Hive
* OSS datawarehouse
* used for structured data
* Use HIVE to access the data in HDFS as if it were in a database
* Hive provides a subset of SQL (HiveQL). The way it does this is by maintaining metadata to define a schema on top of the data.
* declarative - specifies exactly how it should be done
* Hive is designed for batch jobs and not for transactions. It ingests data into a data warehouse format requiring a schema. It does not support real-time queries, row-level updates, or unstructured data. Some queries may run much slower than others due to the underlying transformations Hive has to implement to simulate SQL.

 ### Apache Pig
* used for semi-structured data
* language developed to submit MapReduce jobs
* imperative - makes plans but requests the underlying systems to determine how to process the data
* Pig provides SQL primitives similar to Hive, but in a more flexible scripting language format. Pig can also deal with semi-structured data, such as data having partial schemas, or for which the schema is not yet known. For this reason it is sometimes used for Extract Transform Load (ETL). 
* It generates Java MapReduce jobs. 
* Pig is not designed to deal with unstructured data.

### Distributed processing
* solution to the diminishing results of vertical scaling
* shard your data into pieces and process them on a cluster of nodes
* cant store data on the clusters themselves because what if one fails? Data will be lost
* MTBF - mean time between failure of computers are 3 years e.g. for 3 computers 1 will fail each year
* Google File System whitepaper led to Hadoop DFS -- replicating data across multiple nodes, recovering data when node is lost
* HDFS - 3 copies of data, if falls below 3 will replicate the data. Some systems are rack aware, copies are in different places
  * called annealing or self-healing system
* Google's petabit networking allows you to process data where it is (no more copying to where the compute lives)
* East-west communications -- bisectional bandwidth -- server-to-server communications, the speed of which is important
to serverless applications

## Spark
* cluster computing framework
* RDD - (resilient distributed datasets) hide complexity of location / replication / partitioning / recovering data in a cluster from the user while exposing it as a single entity
* think in "transformations", requests to Spark that describe what you want done but Spark does it in its own way
* transformations: input: RDD -> transform -> output: RDD
* transformations are lambda/anonymous functions (evaluated lazily)
* when Spark receives transformations stores them in a DAG (directed acyclic graph) but doesn't perform them at that time.
Performs when it receives an action, when it will create efficient pipelines to do the work

Spark chains stuff together
```py
from operator import add
lines = sc.textFile("/sampledata/sherlock-holmes.txt")

words =  lines.flatMap(lambda x: x.split(' '))
pairs = words.map(lambda x: (len(x),1))
wordsize = pairs.reduceByKey(add)
output = wordsize.sortByKey().collect()
```

```py
output2 =  lines.flatMap(lambda x: x.split(' ')).map(lambda x: (len(x),1)).reduceByKey(add).sortByKey().collect()

for (size, count) in output2: print(size, count)

```

The gist of what the lab taught
* load your data into HDFS `hadoop fs -copyFromLocal road-not-taken.txt /sampledata/.`
* you can use the `pyspark` `REPL` to play around
* you can make a python file to do transformations and run `spark-submit <name of file>`

```py
from pyspark.sql import SparkSession
from operator import add
import re

print("Okay Google.")

spark = SparkSession\
        .builder\
        .appName("CountUniqueWords")\
        .getOrCreate()

lines = spark.read.text("/sampledata/road-not-taken.txt").rdd.map(lambda x: x[0])
counts = lines.flatMap(lambda x: x.split(' ')) \
                  .filter(lambda x: re.sub('[^a-zA-Z]+', '', x)) \
                  .filter(lambda x: len(x)>1 ) \
                  .map(lambda x: x.upper()) \
                  .map(lambda x: (x, 1)) \
                  .reduceByKey(add) \
                  .sortByKey()
output = counts.collect()
for (word, count) in output:
  print("%s = %i" % (word, count))

spark.stop()
```

* Spark allows you to deal with huge datasets. For small datasets you could just use pandas, but if it cant fit into memory
use Spark
* pandas dataframes can be converted into RDDs

## BigQuery + Spark
* some overlap with Hadoop and in certain cases can be used instead of Hadoop
* Spark jobs can read from BigQuery but go through a temporary GCS storage -- good for periodic or infrequent transfers
* you can also use a connector from Spark to read from BQ (as an RDD) and write back to BQ -- better for frequent transfers 

## Installations scripts

Install custom software on master + worker nodes

* use shebang (#!) to specify what language interpreter to use
* can check role of the VM by checking `/usr/share/google/get_metadata_value attributes/dataproc-role` `Master`
* pre-built initialization actions: `gs://dataproc-initialization-actions`

Example of an initialization script

```bash
#!/bin/bash

# install Google Python client on all nodes
apt-get update
apt-get install -y python-pip
pip install --upgrade google-api-python-client

ROLE=$(/usr/share/google/get_metadata_value attributes/dataproc-role)
if [[ "${ROLE}" == 'Master' ]]; then
   git clone https://github.com/GoogleCloudPlatform/training-data-analyst
fi
```

* Also note that mostly everything here can be done via CLI

## Tensorflow
* nice explanation of ML: assume a solution exists, use exhaustation method to generate solutions then uses error to refine guess

## ML APIs
* nice way to put it: reduces data into countable things 
* TIL: Google has good APIs for chatbots https://dialogflow.com/
* Cool story: Meeting Nanny: Google used Vision API to check if meetings were taking place. Detects motion every 30 seconds
for the first 8 minutes, sends a PubSub message to booker if no motion. Also sends meeting IDs/link as PubSub message.
* Ocado: grocery supermarket: route emails based on NLP
![image](https://user-images.githubusercontent.com/3739702/53931157-018df800-40cf-11e9-9363-4b9de881e438.png)

Read a whole book in Spark and use Sentiment Analysis on it. You'd copy this file to a GCS bucket and do
Dataproc > submit job

```py
#!/usr/bin/env python
# Copyright 2018 Google Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

'''
  This program reads a text file and passes to a Natural Language Processing
  service, sentiment analysis, and processes the results in Spark.
  
'''

import logging
import argparse
import json

import os
from googleapiclient.discovery import build

from pyspark import SparkContext
sc = SparkContext("local", "Simple App")

'''
You must set these values for the job to run.
'''
APIKEY=
PROJECT_ID=
BUCKET=

## Wrappers around the NLP REST interface

def SentimentAnalysis(text):
    from googleapiclient.discovery import build
    lservice = build('language', 'v1beta1', developerKey=APIKEY)

    response = lservice.documents().analyzeSentiment(
        body={
            'document': {
                'type': 'PLAIN_TEXT',
                'content': text
            }
        }).execute()
    
    return response

## main

# We could use sc.textFiles(...)
#
#   However, that will read each line of text as a separate object.
#   And using the REST API to NLP for each line will rapidly exhaust the rate-limit quota 
#   producing HTTP 429 errors
#
#   Instead, it is more efficient to pass an entire document to NLP in a single call.
#
#   So we are using sc.wholeTextFiles(...)
#
#      This provides a file as a tuple.
#      The first element is the file pathname, and second element is the content of the file.
#
sample = sc.wholeTextFiles("gs://{0}/sampledata/time-machine.txt".format(BUCKET))

# Calling the Natural Language Processing REST interface
#
# results = SentimentAnalysis(sampleline)
rdd1 = sample.map(lambda x: SentimentAnalysis(x[1]))

# The RDD contains a dictionary, using the key 'sentences' picks up each individual sentence
# The value that is returned is a list. And inside the list is another dictionary
# The key 'sentiment' produces a value of another list.
# And the keys magnitude and score produce values of floating numbers. 
#

rdd2 =  rdd1.flatMap(lambda x: x['sentences'] )\
            .flatMap(lambda x: [(x['sentiment']['magnitude'], x['sentiment']['score'], [x['text']['content']])] )

# First item in the list tuple is magnitude
# Filter on only the statements with the most intense sentiments
#
rdd3 =  rdd2.filter(lambda x: x[0]>.75)


results = sorted(rdd3.take(50))


print('\n\n')
for item in results:
  print('Magnitude= ',item[0],' | Score= ',item[1], ' | Text= ',item[2],'\n')
```

Example output (not the book but a poem)

```py
('Magnitude= ', 0.8, ' | Score= ', -0.8, ' | Text= ', [u'roads diverged in a yellow wood,\nAnd sorry I could not travel both\nAnd be one traveler, long I stood\nAnd looked down one as far as I could\nTo where it bent in the undergrowth;'])
('Magnitude= ', 0.6, ' | Score= ', 0.6, ' | Text= ', [u'Then took the other, as just as fair,\nAnd having perhaps the better claim,\nBecause it was grassy and wanted wear;\nThough as for that the passing there\nHad worn them really about the same,'])
('Magnitude= ', 0.1, ' | Score= ', -0.1, ' | Text= ', [u'And both that morning equally lay\nIn leaves no step had trodden black.'])
('Magnitude= ', 0, ' | Score= ', 0, ' | Text= ', [u'Oh, I kept the first for another day!'])
('Magnitude= ', 0.1, ' | Score= ', -0.1, ' | Text= ', [u'Yet knowing how way leads on to way,\nI doubted if I should ever come back.'])
('Magnitude= ', 0.3, ' | Score= ', -0.3, ' | Text= ', [u'I shall be telling this with a sigh\nSomewhere ages and ages hence:\nTwo roads diverged in a wood, and I-\nI took the one less traveled by,\nAnd that has made all the difference.'])
```

Could usue this on customer feedback of social media commentary
