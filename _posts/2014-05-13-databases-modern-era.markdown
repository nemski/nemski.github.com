---
layout: post
title: Databases in the modern era
date: 2014-05-13 00:00:00
---

Since the 1960s various types of databases have formed that attempt to solve the most prevalent issues of that era. In the 1970s relational databases became prevalent, as they simplified the management of data by ensuring it was structured in a way that made it easier to update. No more having data inconsistencies, everything is properly mapped and all is good.

![image](https://31.media.tumblr.com/183200a8a006c092fd3d556c838611b4/tumblr_inline_n5ihut0pTh1s54kjo.jpg)

Except not all data models fit relational databases well, some data is too dynamic or complex and managing the data model within an application (in memory) as well as in a database (at rest) became cumbersome. As Object Oriented programming took off in the 1980s Object-databases spawned to allow programmers to more easily and dynamically store their application data. By allowing custom objects to be stored in their database they allowed programmers to more easily integrate their applications and update them.

![image](https://31.media.tumblr.com/8449168556bc9754f9b4832c5b8798c7/tumblr_inline_n5ihweDZGf1s54kjo.jpg)

In the 1990s new research spawned Object-relational database which combined the two approaches to produce a Database Management System capable of storing relational data as well as custom objects and types together. Many of these databases are still in wide spread use today, including Oracle, PostgreSQL and Microsoft SQL.

In the past few years we've seen the explosion of yet another database type, which seeks to solve the most prevalent issue of our era; database performance and availability. That model is NoSQL.

![image](https://31.media.tumblr.com/fa69ffd46292fe6056156e3d7a15bedc/tumblr_inline_n5ik3ybohj1s54kjo.jpg)

NoSQL stores data in a flat schema without tracking the relationships between elements. Consequently it doesn't support JOIN operations or atomic updates to multiple "tables". This greatly benefits performance and availability by allowing any number of nodes to be added to a cluster to service requests, but at the sacrifice of making updates and reporting potentially return inconsistent results.

This provides great benefits to large scale applications that are OK with taking longer to write data and having to work around data inconsistency problems (or ignore them all together), as long as the users get to view the results quickly. Also applications such as Puppet ENC's tend to use NoSQL due to the dynamic nature of the schema. This is because NoSQL provides the capacity to quickly and easily update the "columns" stored while letting the application handle the relationship side of things.
