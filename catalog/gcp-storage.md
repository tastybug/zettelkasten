# GCP Storage Options

## Catalog Tree

* Object Store (binary and object data)
  * `Cloud Storage`
* File Storage (NAS)
  * `Filestore`
* Relational
  * `Cloud SQL`; flavors are MSSQL, Postgres, MySQL
  * `Cloud Spanner`; hybrid of transactional and analytical db
* NoSQL / Nonrelational
  * `Firestore`; hierarchical, mobile, web
  * `Cloud BigTable`; heavy read and write, events
* OLAP
  * `BigQuery`

![Decision Tree](./gcp-storage-decision-tree.png)

## Cloud Storage

* very fast
* HA - depends on storage class, from 99,9% to 99,95%
* extremely durable: 99,999999999%
* it's like S3: a bucket for objects
* no minimum size for objects and buckets; size is limited by the quota
* buckets cannot be nested and the name is _global_
* objects can move between buckets
* offers fine-grained access control to complete buckets or single objects
  * a maximum of 100 access control lists can exist per bucket
  * an ACL specifies a list of users (can be concrete users, `allUsers` or `allAuthenticatedUsers`) and the permission (`owner`, `writer`, `reader`)
  * it's possible to create "signed URLs" that are time-limited, are signed and specify allowed operations (GET, PUT, DELETE) - sort of like a URL with an embedded JWT (so it's a bearer URL)
* automated object lifecycle management
* object versioning supported and multipe versions can be retained
* strongly consistent
* buckets can be mounted into VMs
* storage classes:
  * `standard`; for hot data that is briefly stored
    * no minimum storage duration
    * no retrieval cost
  * `nearline`; infrequently accessed data like backups and multimedia 
    * 30 days minimum data storage 
    * 0.01$/GB retrieval cost
  * `coldline`; infrequently accessed data, read maybe once per quarter
    * 90d minimum data storage
    * 0.02$/GB retrieval cost
  * `archive`; again infrequently accessed, disaster recovery data
    * 365 days minimum data storage
    * $0.05/GB retrieval cost
