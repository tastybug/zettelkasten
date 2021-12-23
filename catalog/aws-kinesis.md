# Kinesis (Kafka like)

Message Queue with Sharding, multiple consumers.

* Replication across regions is build in, there is no MM2 setup to be done
* Offers 4 different strategies: 
    * Data Streams Service (closest to what we used Kafka in Seller Platform)
    * Video Streams Service
    * Kinesis Data Firehose: this writes stream records directly into S3, Amazon RedShift, Amazon Elastic Search (Amazon ES), Splunk, Kinesis Data Analytics
* Consumer Implementation Guide: https://docs.aws.amazon.com/streams/latest/dev/building-consumers.html
* Distributed consent making around which consumer takes which shard seems to be none existent, maybe one has to implement that manually or go with static amount of consumers and hard assignments
* Throughput is reported as lower compared to Kafka
* Max message size is 1Mb
* Ordering can be guaranteed for PUTs from the same client and with the same target partition, but it requires the client to calculate the offset itself. You can provide the desired sequence number (a.k.a. offset) based on the offset of the last message you sent. In absence of that ordering is dependent on arrival at Kinesis.
* TTL 7d, by default messages are kept for 24h
* Topics are called “streams” and have a stream name
* Record structure:
    *  sequence number
    * partition key (like message id in Kafka as it seems)
    * immutable data blob
* A partition key is a unicode string, max 256 chars. A hashing function decides into which shard this will end up.
* Producer and client libraries for various Langs
* Like Kafka, a topic is sharded/partitioned
* there is a shard quota of 500 per account for some regions, 200 for others. Increases can be requested.
* There is a max of 20 consumers per topic (see RegisterStreamConsumer in https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html)
* A shard can support writes up to 1,000 records per second or 1MiB per second, what is reached first
    * Picking an amount of shards is subject to account level quotas: https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html
    * Shard size can be increased and decreased later on, messages will be moved automatically, sometimes using temporary internal shards WHICH COUNT AGAINST YOUR QUOTA!
* You pay per shard
* Apparently no log compaction
