## Leveraging Unstructured Data with Cloud Dataproc on Google Cloud Platform

Structured data
* has a schema

Unstructured data
* no schema
* images, emails
* OR it could have a schema/partial schema but that schema is not useful for your analysis/query
* around 90% of data in enterprise is unstructured

Dataproc
* Spark/Hadoop on the cloud
* MapReduce
* setting up clusters manually have a lot of overhead but Google does this for you
* You will only really need to care about your custom code
* `bdutil` is an open source solution
* Create clusters for specific jobs
* If storing data not on GCS, make sure its located near the compute nodes to avoid large egress charges

Why cloud? 
* All boils down to cost. 
* Cheap storage. When data size goes to petabytes, cost soars
* You can quickly reach petabytes 200 servers logging 50 entries per second for 3 years
* Speed too, if you can parallelize, cut down job duration from days to minutes
