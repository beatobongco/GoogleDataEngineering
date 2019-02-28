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
* program w/ requests, lazy evaluation, directed acyclic graph (DAG)
* think in "transformations" input: RDD -> transform -> output: RDD
* transformations are lambda/anonymous functions
