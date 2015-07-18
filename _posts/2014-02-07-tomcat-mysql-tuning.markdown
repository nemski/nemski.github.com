---
layout: post
title: Tomcat and MySQL tuning
date: 2014-02-07 00:00:00
---

We have an "enterprise" application stack that utilises Apache Tomcat and MySQL, a very common combination for modern web applications.

![image](https://31.media.tumblr.com/ab359c428112f7e8e8985bb1b2362aac/tumblr_inline_n0vpdrXEo81s54kjo.png)

Recently, after an upgrade, we noticed some significant performance problems, specifically at the beginning of this month when client reports were due. While the system was performing well with few users connected, with multiple users connected generating several reports each the system started to crawl.

At first all I was seeing was this error in the logs:

> SEVERE: Cannot get a connection, pool error Timeout waiting for idle object org.apache.tomcat.dbcp.dbcp.SQLNestedException: Cannot get a connection, pool error Timeout waiting for idle object

I guessed this was due to too few connections between Tomcat and the database, so I increased the maxActive setting in the [Resource Definition](http://tomcat.apache.org/tomcat-5.5-doc/config/globalresources.html#Resource_Definitions).

![image](https://31.media.tumblr.com/b8917d224d6542b39ea1f2230c8ea150/tumblr_inline_n0vpww0zsf1s54kjo.png)

Once this was increased and the server restarted, we witnessed the same issue. Logs, from the tomcat server, indicated a different issue:

> SEVERE: Could not create connection to database server. Attempted reconnect 3 times. Giving up.
> com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Could not create connection to database server. Attempted reconnect 3 times. Giving up ...
> Caused by: com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Too many connections

It took me a while to realise this error was actually coming from the database. This meant the bottleneck had moved from the frontend to the backend. I adjusted the max_connections setting but still performance issues persisted. After more digging I found errors in MySQL logs:

> 140205 4:24:48 [ERROR] mysqld: Can't open file: './database/table.frm' (errno: 24)

Which required another setting increased, open_files_limit, and another restart of MySQL.

Finally we experienced another issues where tomcat was throwing an error:

> SEVERE: Could not create connection to database server. Attempted reconnect 3 times. Giving up. ... Caused by: java.sql.SQLException: null, message from server: "Can't create a new thread (errno 11); if you are not out of available memory, you can consult the manual for a possible OS-dependent bug"

Which was resolved by increasing the maximum number of threads mysql can spawn, which is described in [this blog](http://www.mysqlperformanceblog.com/2013/02/04/cant_create_thread_errno_11/).

Something else we noticed improved database performance was mounting the databases with the option "noatime", which stops last accessed time from being written each time a table is updated, which reduced the number of write IOPS.

![image](https://31.media.tumblr.com/d27c0ca8b32a69b66b4d96ad5f3b91c1/tumblr_inline_n0vq0iVWv91s54kjo.png)

## **Lessons Learnt**

Listen to the log files, fix the problems they present first and then move on to more obscure performance issues.

If you can measure it, graph it! I plan to spend some time in the near future feeding key metrics from MySQL and Tomcat JMX into our performance monitoring application so we can keep track of metrics such as number of active database connections (at frontend and backend), number of open files, number of active threads etc.
