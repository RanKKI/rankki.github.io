---
title: "Deploy MongoDB replica set and sharding"
date: 2021-12-03T11:33:25+08:00
draft: false
tags:
 - mongodb
categories:
 - 技术
lang: en
---

## Scale-up or Scale-out

### Why scale?
Imagine you have a computer (server) that can serve 5 users at the same time,
When Alice becomes the 6th user, Alice has to wait for someone to free up the server resources.

It's not user friendly, so you need to improve the performance of the server.

### Scale-up

You decide to upgrade the hardware, using Intel i9 instead of i3, upgrading RAM from 4GB to 64GB and using 1Gbps fibre broadband instead of your ADSL broadband.

Now you can serve 10,000 users at the same time, but when Alice becomes the 10001st user and you can't upgrade your server anymore, then you can scale-out your services.

### Scale-out

If one server can serve 10k users, what if there are 10 servers which can serve 10k * 10 users at the same time.

Once there are multiple servers serving the same thing, load balancing needs to be considered, as we need a service to route requests to different servers.

> In computing, load balancing refers to the process of distributing
> a set of tasks across a set of resources (computing units) in order to
> of making their overall processing more efficient.

## Availability

As a service (content) provider, I want my website (in other words, the server) to be online and provide content 7/24, and that is high availability.

In the case of the database, if one instance goes down, all read/write operations are redirected to another instance.

## Deploy MongoDB

### Standalone instance

Installing MongoDB is easy, follow the official MongoDB manual from [here] (https://docs.mongodb.com/manual/installation/)

After installation, run these commands to start the MongoDB service

```bash
sudo mkdir -p /srv/mongodb/db
sudo mongod --port 27017 --bind_ip localhost,0.0.0.0 --dbpath /srv/mongodb/db --oplogSize 128
```

### Replica Set

> A replica set in MongoDB is a group of mongod processes that maintain the same
> the same dataset. Replica Sets Provide Redundancy and High Availability

In a replica set, there is one primary instance, and all other instances are secondary.
All secondary instances are not writable by the client.
Transactions from the client to the primary instance are automatically synchronised to all secondary instances.

> A secondary can become a primary. If the current Primary becomes unavailable,
> The replica set holds an election to choose which of the secondaries will become the new primary.

![](https://docs.mongodb.com/manual/images/replica-set-read-write-operations-primary.bakedsvg.svg)

```bash
sudo mkdir -p /srv/mongodb/rs0-0 /srv/mongodb/rs0-1 /srv/mongodb/rs0-2
sudo mongod --replSet rs0 --port 27017 --bind_ip localhost --dbpath /srv/mongodb/rs0-0 --oplogSize 128
sudo mongod --replSet rs0 --port 27018 --bind_ip localhost --dbpath /srv/mongodb/rs0-1 --oplogSize 128
sudo mongod --replSet rs0 --port 27019 --bind_ip localhost --dbpath /srv/mongodb/rs0-2 --oplogSize 128
```

Running the commands above will create three MongoDB instances and use ports 27017, 27018, 27019 respectively. and these three instances will be assigned to a replica set `rs0` (you can set the name to anything you want).

If you want to run these instances on the same machine for testing purposes, you need to add `--fork` to each command, or open three sessions and run the commands separately.

Use `mongsh` to connect one of the instances (i.e. 27017) and run the command to add the rest of the instances to the replica set.

```bash
rs.initiate()
$ rs.add("ip:27018")
$ rs.add("ip:27019")
```

Now, whatever you add, update or delete in the primary instance (:27017), it has also been done in the other two instances.

### Sharding

Because *Replica Set* is used to provide high availability to reduce downtime.
Sharding is solving too many requests to a database instance at the same time. ***Limitation of the I/O speed***

> Sharding is a method of distributing data across multiple machines.
MongoDB uses sharding to support deployments with very large > data sets and high throughput operations.
> and high throughput operations.

![](https://docs.mongodb.com/manual/images/sharded-cluster-production-architecture.bakedsvg.svg)

Starting a config server is similar to a replica set, but with `--configsvr', (config server must be a replica set)
```bash
sudo mongod --configsvr --replSet conf --port 27020 --bind_ip localhost --dbpath /srv/mongodb/conf-0 --oplogSize 128
```

The replica set used in sharding must be run with `--shardsvr'.
```bash
sudo mongod -shardsvr --replSet rs0 --port 27017 --bind_ip localhost --dbpath /srv/mongodb/rs0-0 --oplogSize 128
```

After starting the config server, add 2 or more replica sets to the config server.

```bash
mongosh ip:27020
sh.addShard( "rs0/ip:27017,ip:27018,ip:27019" )
sh.addShard( "rs1/ip:27017,ip:27018,ip:27019" )
sh.addShard( "rs2/ip:27017,ip:27018,ip:27019" )
```

And then use `enableSharding` to determine which database to shard, then specify which key of the collection to use as the *shard key*.

```bash
sh.enableSharding("test")
sh.shardCollection("database.collection", { uid : "hashed" } )
```

Now connect to the config server as if you were connecting to a single MongoDB instance, Each document will be inserted into a different replica set based on the hashed shared key `uid`.

---

 - https://en.wikipedia.org/wiki/Load_balancing_(computing)
 - https://docs.mongodb.com/manual
