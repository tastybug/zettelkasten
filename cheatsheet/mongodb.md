# MongoDB

Create index on the fly
```
db.getCollection('listings').createIndex(
  {
      "seller.foreignId": 1,
      "partnerName": 1
  },
  {
      name: "debuggerindex",
      unique: false,
      background: true
  }
)
```

Remove index on the fly
* `db.getCollection('listings').dropIndex( "debuggerindex" )`

Is the current node the master?
* `db.isMaster()`
