# Useful shell commands and server options

## CRUD 

Insert document with write concern, journal and timeout
```
db.hello.insertOne(
    {"msg": "hello"}, 
    { "writeConcern": {"w": 3, "j": true, "wtimeout": 1000} }
)
```

Find using read concern
```
db.hello.findOne().readConcern("local")
db.hello.findOne().readConcern("secondary")
db.hello.findOne().readConcern("primaryPreferred")
db.hello.findOne().readConcern("nearest")
db.hello.findOne().readConcern("secondaryPreferred")
```

## Indexes

Create compound index with french collation
```
db.myColl.createIndex(
   { score: 1, price: 1, category: 1 },
   { collation: { locale: "fr" } } )
```

Create compound text index
```
db.pojo.createIndex(
    {"stringField": "text", "nextStringField": "text"}
)
```

Search using $text
```
db.pojo.find({$text: {$search: "string 2"}})
```

Search using $text and exact phrase
```
db.pojo.find({$text: {$search: "\"string 2\""}})
```

## Aggregation

Group by state, sum population and filter by total population
```
db.zipcodes.aggregate( [
   { $group: { _id: "$state", totalPop: { $sum: "$pop" } } },
   { $match: { totalPop: { $gte: 10*1000*1000 } } }
] )
```

## Views

```
{ _id: 1, empNumber: "abc123", feedback: { management: 3, environment: 3 }, department: "A" }
{ _id: 2, empNumber: "xyz987", feedback: { management: 2, environment: 3 }, department: "B" }
{ _id: 3, empNumber: "ijk555", feedback: { management: 3, environment: 4 }, department: "A" }
```

```
db.createView(
   "managementFeedback",
   "survey",
   [ { $project: { "management": "$feedback.management", department: 1 } } ]
)
```

## Replication

### Starting replica set script
```
mongod --replSet replica-set-0 --port 27017 --bind_ip localhost --dbpath /storage/mongodb/db/nosql-training/replica-set/0/0 &
mongod --replSet replica-set-0 --port 27018 --bind_ip localhost --dbpath /storage/mongodb/db/nosql-training/replica-set/0/1 &
mongod --replSet replica-set-0 --port 27019 --bind_ip localhost --dbpath /storage/mongodb/db/nosql-training/replica-set/0/2 &
mongod --replSet replica-set-0 --port 27020 --bind_ip localhost --dbpath /storage/mongodb/db/nosql-training/replica-set/0/3 &
mongod --port 27021 --replSet replica-set-0 --bind_ip localhost --dbpath /storage/mongodb/db/nosql-training/replica-set/0/arbiter-0
```

### Shell commands
Check master info
```
rs.isMaster()
```

Replica set configuration document
```
rsconf = {
  _id: "rs0",
  members: [
    {
     _id: 0,
     host: "<hostname>:27017"
    },
    {
     _id: 1,
     host: "<hostname>:27018"
    },
    {
     _id: 2,
     host: "<hostname>:27019"
    }
   ]
}
```

Init replica set using config
```
rs.initiate(rsconf)
```

Add member to replica set
```
rs.add( { host: "mongodb3.example.net:27017", priority: 0, votes: 0 } )
```

Check replica set config
```
rs.config()
```

Check replica set status
```
rs.status()
```

## Sharding

### Starting shards
Create config server replica set (with just 1 member, repeat to add more)
```
mongod --configsvr --replSet cluster-0-config --dbpath 
/storage/mongodb/db/nosql-training/sharded-cluster/cluster-0/config/0/ 
--bind_ip localhost --port 27017
```

Create data shard server replica set (with just 1 member, repeat to add more)
```
mongod --shardsvr --replSet cluster-0-data-0 --dbpath /storage/mongodb/db/nosql-training/sharded-cluster/cluster-0/data-shards/0/0 --bind_ip localhost --port 27018
```

Start mongos routing service
```
mongos --configdb cluster-0-config/localhost:27017 --bind_ip localhost --port 27021
```

### Mongos shell commands

Invoke these command while in **mongos** shell

Add shard
```
sh.addShard("replica-set/localhost:27017")
```

Enable sharding for db
```
sh.enableSharding("shard-db")
```

Shard collection with range shard key
```
sh.shardCollection("shard-db.msg", {msg: 1})
```

Shard collection with hashed shard key
```
sh.shardCollection("user.user", {_id: "hashed"})
```

See shard status
```
sh.status()
```

# replica set config document
```
{
_id: "<replSetName>",
configsvr: true,
members: [
  { _id : 0, host : "cfg1.example.net:27019" },
  { _id : 1, host : "cfg2.example.net:27019" },
  { _id : 2, host : "cfg3.example.net:27019" }
 ]
}
```