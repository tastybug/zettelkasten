# SQS Simple Queue Service

Message Queue, best for single consumers. Alternative: Kinesis. Comes in 2 flavors:

Standard: 

* ordering is not guaranteed
* At least once delivery, message duplication can happen
* nearly unlimited number of transactions per second, no capacity management necessary

FIFO queue:
* up to 300 message operations per second (send, receive, delete)
* allows batch operations (up to 10 messages), so max 3000 messages per second
* one can enable "high throughput mode", which increases the limit to 30000 messages (batching) and 3000 w/o batching (currently in preview in some regions)
* exactly-once delivery
* ordering guaranteed

Traits:
* requests are billed, based on size (consume and produce)
    * try long polling times for when the queue is empty to drive down costs
    * messages up to 256kb text, each 64kb chunk is billed
    * messages can contain up to 10 headers in the form of key/value pairs; values can be string, binary, numbers
    * a batch request is billed based on the size as well; at <64kb batch request would be billed as a single request
* messages are retained max for 14d, default is 4d
* messages are locked by the consumer to avoid double-processing
* queues can be shared across accounts
* access can be restricted by IP and time-of-day
* messages have a unique id
* messages need to be programmatically deleted after consumption, otherwise they resurface again after a defined amount of time ("visibility timeout").
* Consuming with more than one consumer (as in different apps, not multiple consumer threads) will not work well due to the message resurfacing behaviour and the inability of tracking consumption over different consumers. Unlike kafka, consumers have no identity, a message is simply received or not.
* dead-letter-queues built in: when a message has exceeded the "maximum receive coun", 's moved to a DLQ associated with the original queue -> set up separate consumer processes for those DLQs. DLQ must be of same type as original queue (standard or fifo)

