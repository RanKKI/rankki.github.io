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
Imagine that you have a computer(server) that can serve 5 users at same time,
if Alice become the 6th user, Alice has to wait someone released the server
resources.

It's not friendly to our users, so you must improve the server's performance.

### Scale-up

You decided to upgrade the hardware, using intel i9 instead i3, upgrade RAM to 64GB from 4GB
and using 1Gbps fibre broadband instead your ADSL broadband.

Now, you can serve 10k users at same time, however Alice become the 10,001st user and 
you can't upgrade your server anymore, then you will considering scale out

### Scale-out

If one server can serve 10k users, what if there are 10 servers, 
which can serve 10k * 10 users at same time.

Once there are multiple server to serve same thing, Load Balance must be considered

> In computing, load balancing refers to the process of distributing 
> a set of tasks over a set of resources (computing units), with the aim 
> of making their overall processing more efficient. 

## Availability

As a services(content) provider, I want my website (in other word, the server) 
online and provide content 7 * 24 hours, and that is high availability.

In case of Database, if a instance is down, all read/write operation will 
redirect to other instance.

## Deploy MongoDB

### Setup a standalone instance

Installing MongoDB is easy, following the official MongoDB manual from [here](https://docs.mongodb.com/manual/installation/)

After install, there are two way to start the MongoDB services.

Using `systemctl` for Linux system (you can't use this command if you're using Windows)
```bash
sudo systemctl start mongod
```

Calling mongod directly
```bash
sudo mkdir -p /srv/mongodb/db
sudo mongod --port 27017 --bind_ip localhost,0.0.0.0 --dbpath /srv/mongodb/db --oplogSize 128
```


### Replica Set

> A replica set in MongoDB is a group of mongod processes that maintain 
> the same data set. Replica sets provide redundancy and high availability

In a Replica Set, there is a primary instance, and the rest of instances 
are secondary. all secondary instances are not writable by the client. 
transaction made from client to the primary instance, will automatic sync to all secondary instances. 

> A secondary can become a primary. If the current primary becomes unavailable, 
> the replica set holds an election to choose which of the secondaries becomes the new primary.

![](https://docs.mongodb.com/manual/images/replica-set-read-write-operations-primary.bakedsvg.svg)

```bash
sudo mkdir -p /srv/mongodb/rs0-0 /srv/mongodb/rs0-1 /srv/mongodb/rs0-2
sudo mongod --replSet rs0 --port 27017 --bind_ip localhost --dbpath /srv/mongodb/rs0-0 --oplogSize 128
sudo mongod --replSet rs0 --port 27018 --bind_ip localhost --dbpath /srv/mongodb/rs0-1 --oplogSize 128
sudo mongod --replSet rs0 --port 27019 --bind_ip localhost --dbpath /srv/mongodb/rs0-2 --oplogSize 128
```

Execute commands above will create three MongoDB instances and using ports 27017, 27018, 27019 respectively.
and those three instances are assigned to a replica set `rs0` (you may set the name to anything you want).

By adding `--fork --logpath /var/log/mongod.log` on each `mongod` command if you want to run those three 
instances on the same machine (only in test environment), or opening three session and run commands separately.

Using `mongsh` to connect one of the instances (i.e. 27017), and run the command to add rest of instances to the replica set.
```bash
$ rs.initiate()
$ rs.add("ip:27018")
$ rs.add("ip:27019")
```

Now, whatever you insert, update or delete to the primary instance (:27017), it also been done to the rest two instances.

### Sharding
Since *Replica Set* is used to provide high availability to reduce the downtime. 
Sharding is solving to many requests at same time to one database instance. *limitation of I/O speed*

> Sharding is a method for distributing data across multiple machines.
> MongoDB uses sharding to support deployments with very large data
> sets and high throughput operations.

![](https://docs.mongodb.com/manual/images/sharded-cluster-production-architecture.bakedsvg.svg)

Starting a config server is similar to replica set but with `--configsvr`, (config server must be a replica set)
```bash
sudo mongod --configsvr --replSet conf --port 27020 --bind_ip localhost --dbpath /srv/mongodb/conf-0 --oplogSize 128
```

Those replica set used in sharding must run with `--shardsvr`
```bash
sudo mongod -shardsvr --replSet rs0 --port 27017 --bind_ip localhost --dbpath /srv/mongodb/rs0-0 --oplogSize 128
```

After the config server start, add 2 or more replica set to the config server.

```bash
mongosh ip:27020
sh.addShard( "rs0/ip:27017,ip:27018,ip:27019" )
sh.addShard( "rs1/ip:27017,ip:27018,ip:27019" )
sh.addShard( "rs2/ip:27017,ip:27018,ip:27019" )
```

And then using `enableSharding` to set which database for sharding, 
then determined which key of the collection as *Shard Key*

```bash
sh.enableSharding("test")
sh.shardCollection("database.collection", { uid : "hashed" } )
```

Now connect to the config server just like connecting to a single MongoDB instance,
Each document will insert to different replica set based the hashed shared key `uid`

---

 - https://en.wikipedia.org/wiki/Load_balancing_(computing)
 - https://docs.mongodb.com/manual
