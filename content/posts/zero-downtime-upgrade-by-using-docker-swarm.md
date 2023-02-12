---
title: "Zero-downtime upgrade by using Docker Swarm"
date: 2023-02-11T13:52:28+08:00
draft: false
tags:
 - Docker
categories:
---

We want users to be able to continue using the service during the upgrade process, the first thing we have to do is kill the process when we want to upgrade a running application, and then restart it.

Here is the problem, there is a gap between killing and restarting a process, is there a way to solve this?

And what if the new changes have bugs? How do we restart the old application in a quick way?

## Concepts

### Rolling update

The technical name of this is **Rolling Update**, we can continuously deliver our new changes without any downtime.

The concept is simple, stop the old process after the new process is started and then move all the traffic from the old process to the new one.

### Load balancing

To do this, we may need a service call **Load Balance**, it will navigate all requests to different nodes.

By decreasing the workload weight of the outdated process, no new requests will be directed towards it. This will allow us to safely shut down the old node.


### Rollback

We can **Rollback** the service if there's a bug in the new node by replacing the current application with its previous version.

But, manually performing the task and keeping track of previous releases is challenging.

## Docker Swarm

> Swarm provides many tools for scaling, networking, securing and maintaining your containerized applications, above and beyond the abilities of containers themselves.

### Init Swarm

```
docker swarm init
```

By using this command, we had initiated the swarm environment and treat current machine as a Leader

### Join Swarm

On other machines, we can use command to join a swarm as a worker node

```
docker swarm join --token <TOKEN> <IP>:<PORT>
```

## Docker Service

Docker service works with the Swarm orchestrator.

### Create

```
docker service create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

### Update

```
docker service update [OPTIONS] SERVICE
```

### Zero-Downtime Update

by default, updating a service will stop current node first, then run the latest image, by adding `--update-order start-first`, we can start the new image first, then stop the old node.

```
docker service update --update-order start-first [OPTIONS]  SERVICE
```


### Scale

if the resources usage are too high of a single node, we can scale-out this service, let other nodes run the same service to balance the workloads

```
docker service scale SERVICE=<NUMBER>
```

But remember, you need more than one node in your Swarm orchestrator, otherwise these services will still run on a single machine.