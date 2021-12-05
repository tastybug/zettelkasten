# CAP Theorem

In general CAP theorem says that you have 3 opposing system qualities. These qualities are scales, they aren’t binary choices.

* Consistency:
    * BEST: every read receives the most recent write or an error
    * WORST: eventual consistency
* Partition Tolerance, a scale between
    * BEST tolerance: Longterm partition can happen: a node becomes isolated. A high number of network packages is lost.
    * MID tolerance: Shortterm partition can happen because there is lag between the syncing nodes. Some network packages are lost or delayed.
    * WORST tolerance: No partition: you either only have a single node or multiple nodes, but latency is no issue due to extremely fast, reliable network
* Availability, a scale between:
    * BEST availability: Every request receives a non-error response, without the guarantee that it contains the most recent write.
    * MID: Limited feature set in case of a partition situation.
    * WORST: the system as a whole or single nodes cease operation

Any modern distributed system comes with a network connection. If latency is really now and there is no connection problem, things are fine. But in reality you occasionally have issues. You want your system to be up, even if there are some connection delays. You cannot run a distributed system without tolerance for lag, so choice of “Partition Tolerance” is always there.
With this given, CAP Theorem is actually more about your choice of A vs. C in (short-term/long-term) partition situations.

It comes down to 2 choices:

* You want to cancel the operation and thus decrease the availability, but ensure consistency
* You proceed with the operation, thus providing availability, but risk inconsistency


## Examples for Pairs

CA: Traditional RDBMS (MySQL, Oracle, MSSQL, Postgres)

You value availability and strong consistency. Your DB runs on highly available machines to avoid partitioning (which takes the system down, as in you are not partition tolerant).

CP: Mongo, HBase, Redis, MemcacheDB

In case of serious network problems your writes will fail (depending on the write concern) to avoid inconsistency. Your DB becomes unavailable.

AP: Cassandra, Dynamo, Voldemort

Consistency is eventual, but availability is king.

_These examples above are taken from the web and are pretty shit. Mongo for example leaves it to the dev to decide between A and C thanks to write concerns._

