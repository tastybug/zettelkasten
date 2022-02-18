# Redis

* NoSQL
* built-in replication
* eventual consistent
* key value store, schema-less
* supports querying
* supports types (string, int, lists, sets, sorted sets, hashes (key/value pairs), bitmaps, hyperlogs, geospatial indexes)
* powerful indexing options on single and multiple fields
* not to be exposed publically: simple auth only, no data encryption


## redis-cli

The standard cli client is `redis-cli`. A short testdrive:

```
docker run --name redis-server --rm -d redis
docker exec -ti redis-server redis-cli
```

## Few Examples

```
$set foo bar
OK
$get foo
"bar"
$set foo 100
OK
$incr foo
(integer) 101
$get foo
"101"
```

## Commands

* `exists key`
* `set key value`
* `get key`
* `incr key`
* `decr key`
* `incrby key 20`
* `decrby key 20`
* `del key`
* `flushall`: removes everything
* `expire key 50` key will be deleted in 50s
* `ttl key` remaining time for key
* `setex greeting 30 "Hello"` sets a value and ttl
