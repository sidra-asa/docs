> Author: sidra-asa

# Deploy sharding cluster in MongoDB

## Introduction

The definition of each role are described in [offcial document](https://docs.mongodb.org/manual/core/sharded-cluster-architectures-production/).<p>
We will setup a cluster with one mongos, one config replica set, two shard sets

* mongos: mongos<p>
* config1: replica set of config server.<p>
* config2: replica set of config server.<p>
* shardI1: a node in shardI shard set.<p>
* shardI2: a node in shardI shard set.<p>
* shardII1: a node in shardII shard set.<p>
* shardII2: a node in shardII shard set.<p>

## Prehand works

Add following line to /etc/hosts in every server:

    192.168.0.1      mongos
    192.168.0.2      config1
    192.168.0.3      config2
    192.168.0.4      shardI1
    192.168.0.5      shardI2
    192.168.0.6      shardII1
    192.168.0.7      shardII2

Then complete [other prehand works](./prehand_works.md).

## shard sets 

Modify /etc/mongod.conf in shardI1 as below:

    net:
      port: 27017
      bindIp: 127.0.0.1,192.168.0.4
  
    replication:
      replSetName: shardI

    sharding:
      clusterRole: shardsvr

Restart mongod then initial replica:

    $ sudo service mongod restart

    $mongo 192.168.0.4
    MongoDB shell version: 3.2.4
    connecting to: test
    > rs.initiate()
    {
        "info2" : "no configuration specified. Using a default configuration for the set",
        "me" : "192.168.0.4:27017",
        "ok" : 1
    } 
    shardI:PRIMARY>

Add member to replica set.

    shardI:PRIMARY> rs.add("shardI2") 

Redo above works to shardI2, shardII1 and shardII2.<p>
Don't use the same IP or replSetName to other node.

## config server

Add following lines to /etc/mongod.conf in config1:

    net:
      port: 27017
      bindIp: 127.0.0.1, 192.168.0.2


    replication:
      replSetName: configReplSet
    sharding:
      clusterRole: configsvr

Then restart mongo:

    $ service mongod restart

Redo above works to config2.

## mongos

excute following command:


    mongos --configdb configReplSet/config2:27017,config3:27017


Add shards to mongos:

    $ mongo 192.168.0.1
    MongoDB shell version: 3.2.4
    connecting to: test
    mongos> sh.addShard( "shardI/shardI1:27017,shardI2:27017" )
    { ok : 1}
    mongos> sh.addShard( "shardII/shardII1:27017,shardII2:27017" )
    { ok : 1 }

Enable sharding in db named by test:

    mongos> sh.enableSharding( "test" )

Enable sharding in collection named by testshard, using '_id' as sharding key:

    mongos> use test
    mongos> db.testshard.createIndex({_id : 1}, {background : true})
    mongos> sh.enableSharding( "test.testshard", {"_id" : 1} )
    { "collectionsharded" : "test.testshard", "ok" : 1 }

Enjoy MongoDB !
