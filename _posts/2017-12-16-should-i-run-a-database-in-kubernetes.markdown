---
layout: post
title: "Should I run a database in Kubernetes?"
date: 2017-12-16 00:00:00
---

A little more than a year ago I wrote a post, [Thou Shalt Not Run A Database Inside A Container](http://patrobinson.github.io/2016/11/07/thou-shalt-not-run-a-database-inside-a-container/) and since then it's become my most read blog post with more than 10,000 unique page views.

![analytics](https://i.imgur.com/PYLX86w.png =256x64){:class="img-responsive"}

In fact, if you Google the following terms and select "I'm feeling lucky" you'll likely land there:

* docker distributed database
* running databases in containers
* database inside docker
* running database in docker

So it's time to update that post to ensure it accurately reflects the state of the world.

State of the (container) world
---

At the time Kubernetes was at the 1.4 release and there was a concept of PetSets which was specifically designed to solve this problem, but it was still in Alpha. Today they have been renamed [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) and are in Beta. At the time Kubernetes supported [PersistentVolumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) with a number of storage backends, including GCE PersistentDisks and AWS EBS, however they had to be manually provisioned by an administrator. Today Kubernetes is able to automatically provision these disks as required. While there are other container orchestration platforms, I've focused on it because it's the most used system today.

So does this mean you can run Databases in a Container Orchestration platform like Kubernetes? Well yes it seems now this is a feasible solution, at least to the extent I would consider creating a Proof of Concept for it. However I would encourage anybody considering this option to think long and hard about one question:

![why](https://i.imgur.com/oofd8p3.gif)

Why
---

Consider the alternatives, many cloud providers offer managed database options that can scale impressively. In AWS you can launch a MySQL-Compatible Aurora instance with 64 CPUs and 488GB of RAM (which will cost you a whopping $6,900 per month in Nth Virginia). Some may argue the cost is prohibitive, the same EC2 instance standalone is about 55% of the cost, but that doesn't factor in storage costs and general maintenance. The Total Cost of Ownership is likely much higher once these are factored in. Replication and backups are hard problems with databases, often requiring a lot of manual intervention. AWS recently [blogged](https://aws.amazon.com/blogs/database/amazon-rds-under-the-hood-multi-az/) about how they handle replication and failover, which is available with the Multi-AZ option. This immediately doubles your running costs, but it works seemlessly; failover takes seconds and I've personally never seen it not work.

Potential Use Cases
---

Given the above the most common use case; I have a handful of databases they are relatively large and important and I just want them to work, it seems there would be no good reason to utilise either cloud based or out of the box solutions (like Percona) to manage your databases. Sometimes though companies may have hundreds or even thousands of databases and in those cases RDS costs are prohibitive. This begs the same question though; why. Why do you need that many? Some may claim they are at such a massive scale that they were forced to shard, some are just poorly designed but some [legitimately need it](https://medium.com/@Pinterest_Engineering/sharding-pinterest-how-we-scaled-our-mysql-fleet-3f341e96ca6f). While I'd love to have a conversation with someone at Pintrest about why they couldn't use other technologies such as NoSQL or Graph Databases to solve their problem I haven't had the chance, so I'll refrain from passing judgement on if this was the best approach. [Spanner](https://cloud.google.com/spanner/) is another awesome technology in this field that offers some amazing capabilities to scale while maintaining consistency.

With all the options available to you, I still can't find a great production use case to run my relational databases in containers. That does not mean they do not exist, I am not an all seeing Oracle.
