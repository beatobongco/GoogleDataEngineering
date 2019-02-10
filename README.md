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

Cloud Storage
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

Compute Engine
* think of as fungible - able to replace or be replaced by another identical item
