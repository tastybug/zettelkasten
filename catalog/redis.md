# Redis

* NoSQL
* built-in replication
* eventual consistent
* key value store, schema-less
* supports querying
* supports types (string, int, lists, sets, sorted sets, hashes (key/value pairs), bitmaps, hyperlogs, geospatial indexes)
* powerful indexing options on single and multiple fields
* not to be exposed publically: simple auth only, no data encryption

## Some More Complex Types

### List

A list of strings, ordered by insertion. You can treat a list like a queue because you add by pushing left (`LPUSH`) or right (`RPUSH`) and you can `LPOP` and `RPOP` to pop. You can however also use `LINSERT` to add values at a specific index. 

You access with `LRANGE`, which is 0 based:
```
LPUSH people john
LPUSH people jane
LPUSH people bob
LRANGE people 0 -1 # this gives all starting from 0
1) "bob"
2) "jane"
3) "john"
LRANGE people 1 2 # everything between index 1 and 2
1) "jane"
2) "john"
```

### Sets

A set of strings, no repetition of values as per usual.

A few commands:

* `SADD cars "Ford"`
* `SISMEMBER cars "Chevy"`
* `SMEMBERS cars` lists all elements
* `SCARD cars` set length
* `SMOVE cars mycars "Ford"` remove `Ford` from `cars` and add it to `mycars`
* `SREM cars "BMW"`
* there's a few more ..

### Sorted Sets

Like a regular set, but values come with a "score" which defines the order of things. Value must not repeat, but scores _can_ repeat.

```
ZADD users 1981 "Brad Bradison"
ZADD users 1975 "John Doe"
ZADD users 1975 "Jane Jannison"
```

## redis-cli

The standard cli client is `redis-cli`. A short testdrive:

```
docker run --name redis-server --rm -d redis
docker exec -ti redis-server redis-cli
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
* `rename currentkey newkey`
* `append key ' world'` appends ` world` to key's value
