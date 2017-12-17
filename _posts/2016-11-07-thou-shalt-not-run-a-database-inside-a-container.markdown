---
layout: post
title: "Thou shalt not run a database inside a container"
date: 2016-11-07 00:00:00
---

So my last blog post, [Docker in Production: A retort](/2016/11/05/docker-in-production/), generated a lot of comments. One of them was "Why shouldn't you run a Database in Docker?" I've seen this questions a few times, but rarely reply because it's such a long answer. But here goes.

EDIT: I've been made aware I'm not being clear that I am referring to running a database in a container management solution, like Kubernetes. The issues laid out below don't relate to just running your database in docker with no shared storage or ability to start it automatically on a different node.

## Kelsey Hightower told you not to

He's a smart guy, he's worked at CoreOS and now works at Google. He has done some fantastic talks on Docker, Kubernetes and System Administration in general. [Here's him talking about it](https://youtu.be/Nosa5-xcATw?t=1080)

## Distributed systems are hard

Like really hard. But that's ok when your application is stateless, because when it all goes to shit you just reboot things and they magically get better. But when your database gets fubar you can end up with permanent corruption. The most complex part of a distributed system is the [Byzantine Generals problem](https://en.wikipedia.org/wiki/Byzantine_fault_tolerance#The_Byzantine_Generals.27_Problem), which describes the problem of how we determine what is the source of truth and how do we know for certain it is the only source of truth. There are various solutions to this problem, usually implemented in the storage engine of databases. Some databases do this well, mostly NoSQL databases where it's possible to shard data across multiple nodes by not supporting a JOIN operation. Traditional relational databases do not do this well, they generally only work  in a master-slave configuration. There are ways to run them in master-master configuration but you need a very specific skill type to execute this. If you're doing this, well why are you reading my blog post? You know better than I do.

If you cannot determine what is the master node, which involves some kind of conensus algorithm and an odd number of nodes to vote on it, then you will end up with data loss and/or corruption.

## Volume management for Docker is still in it's infancy

There isn't a well respected solution for how to do volume management for Docker clusters right now. Some people use NFS, but it lacks any kind of locking support on files (see [this blog](http://0pointer.de/blog/projects/locking.html) for more details on linux file lock support). So you cannot guarentee that another system is not writing to your files.

GlusterFS is a recent successor to NFS, that offers much better performance, clustering and scalability. However it has several known issues that render locking unreliable.[1](https://bugzilla.redhat.com/show_bug.cgi?id=1194546) [2](https://bugzilla.redhat.com/show_bug.cgi?id=1287099)

## You lied to us! I can total run my NoSQL database in Docker

The answer here is it really depends on the underlying technology. As stated, there is no solution for filesystem level replication of data (except in a few proprietary solutions like AWS's flagship database Aurora, VMware File System and Google File System, none of which tackle this specific use case). So instead we must rely on the storage engine of the database to do clustering and replication for us. Some are well known for their support of this (etcd, ElasticSearch and Riak KV to name a few), but it needs to be considered on a case by case basis to evaluate how well the particular database deals with this problem, because it's so easy to get wrong.

## With great knowledge comes great responsibility

Now you know why there is significant anxiety to running databases in Docker, you have the power to make the decision for yourself.

If you find yourself wanting to learn more I highly recommend Pintrest's blog on [how they sharded their MySQL Cluster](https://engineering.pinterest.com/blog/sharding-pinterest-how-we-scaled-our-mysql-fleet). They use Zookeeper, similar to etcd, to keep the configuration of which data lives in which MySQL database. This provides a highly available, consistent store. But they still recommend you "only read/write to the master... It simplifies everything and avoids lagged replication bugs."

I've yet to see a blog post about somebody containerising their MySQL server and how it went really well.

## EDIT: Some solutions to the problems I have presented here

Some people have pointed out a couple of solutions to the problem presented here. One is the Kubernetes [PetSet](http://kubernetes.io/docs/user-guide/petset/) concept. While still in Alpha (so obviuosly not fit for Production), this looks to be solving the exact problem of ensuring one and only one master exists at any one time.

Uber also [blogged about their dockerizing of MySQL](https://eng.uber.com/dockerizing-mysql/) which is a very similar solution to the Kubernetes approach. They are defining the cluster topology up front, which guarentees a single master. They don't talk about how they generate the master, but since that obviously comes from another source it's likely highly consistent.

So today we still can't run Docker inside a container cluster, but there are some good solutions on the roadmap.

Check out my [follow up post](http://patrobinson.github.io/2017/12/16/should-i-run-a-database-in-kubernetes/) on this ∂subject.
