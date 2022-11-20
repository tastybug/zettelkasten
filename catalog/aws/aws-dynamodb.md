# DynamoDB

* is a document based database (schemaless)
* Table-basiertes Speichern, keine Relationen
* guenstig in Zugriff und Speicherung
* queries innerhalb eines Tables moeglich, 
* for aggregate/sort/filter data has to be pulled and done in app
* immediately consistent wenn ohne indices
* indices come in 2 flavors: local, global. local is immediately consistent, but all data with the same partition key  must fit into 10gb, otherwise write will fail. Global is not limited in that regard, but it turns into eventual consistency. With global, all writes go into a queue (which connects the table and the index) that cannot be observed. Once that queue is full, writes will fail. For stress situations this can be a killer.
* there is partitioning which is managed for you (maybe you can have control over it). You define a key property in your document (animal.pet = dog|cat|bird) that is used for partitioning. You can e.g. have typed partitions that way.
