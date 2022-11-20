# S3

A giant distributed hashtable

* entries can be accessed in parallel at high speed
* key is a string
* value any blob of data up to 5tb
* reads have an initial delay of 20ms, then up to 90mb/s
* storage 23$/tb/month, 5$/million uploads, 0.4$/ million reads
* ideal for operations reflecting human interaction (image upload,read)
* values are immutable (cannot append, but maybe cut?)
* no https for static http hosting (Cloudfront will do that for you)
* bucket names are globally unique across customers, regions, like a domain name
    * but buckets still reside in a region. think about adding the region to the bucket name to clarify where it sits, like "images-us-east-1" and not just "images"
