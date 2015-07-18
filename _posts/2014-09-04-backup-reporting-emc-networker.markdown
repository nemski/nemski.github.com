---
layout: post
title: Backup reporting with EMC NetWorker
date: 2014-09-04 00:00:00
---

Today I spent 4 hours building a custom reporting solution for NetWorker manual backups and thought I'd share the process for getting there. NetWorker comes with the capability to report on successful and failed backup jobs, but what if a backup never ran? We've had one scenario where a backup was scheduled through the NetWorker GUI but the schedule didn't actually exist in the backend (obviously a bug in the product), but mostly we were concerned about our manual backup jobs.

We use manual backup jobs for unique scenarios where a scheduled backup won't suffice, such as when we have to mount a Block level or FileSystem snapshot and back that up.

For this we have a dedicated debian VM with 512mb of RAM and a single vCPU. This seems to cope with whatever we can throw at it, which I was quite impressed with considering the agent is doing dedupe on the fly for multi-TB databases. This server has dedicated folders for each backup so we can distinguish them in NetWorker.

Cron tasks backup files at various intervals, so how do we know when a backup was not run? We can generate emails from our custom scripts but this is messy and error prone.

To rip data out of NetWorker you can use a command called mminfo, which even kindly dumps the data in XML. The following command gives pretty much all the information you need about backup jobs run in the past day:

{% highlight bash %}
mminfo -v -s &lt;backup host&gt; &nbsp;-a -r \
"client,name,group,nfiles,ssid,savetime(25),level,sumsize,nsavetime,sscomp(25)" \
-q "savetime&gt;1 days ago" -otR -xml
{% endhighlight %}

Then I wrote a very simple 37 line Ruby script to parse that XML into a single database table called&nbsp;"jobs" on a daily basis.

> ![image](https://31.media.tumblr.com/a804c6ba74a29a8875e37eca72b6550a/tumblr_inline_nbdlinChPC1s54kjo.jpg)

Then we have another table describing our manual backup jobs, which has foreign keys to the job client column (backup_host) and job name column (mount_point)

Once data is loaded into the jobs table by our script and we've populated our manual jobs we can create two database views. First is to report on manual backups that haven't run recently, using our schedule field:

{% highlight sql %}
SELECT MAX(j.`end-time`) AS last_backup, m.* FROM jobs j
RIGHT JOIN manual_backups m ON m.backup_host = j.client AND m.mount_point = j.name
GROUP BY m.backup_host,m.mount_point
HAVING DATE_ADD(last_backup, INTERVAL m.schedule DAY) &lt; NOW()
{% endhighlight %}

An export of which is scheduled once a week and emailed out if there are any matches.

Second is to report on jobs in our "Manual" group which don't have entries in the manual_backups table, so we know if we've forgotten to add a manual job:

{% highlight sql %}
SELECT j.* FROM jobs j
LEFT JOIN manual_backups m ON m.backup_host = j.client AND m.mount_point = j.name
WHERE j.group = "Manual" AND m.mount_point IS NULL
GROUP BY j.name,j.client
{% endhighlight %}

If a manual job is no longer required we can then purge all of its data from the jobs table to prevent it showing in this report.

We're considering expanding this database to provide custom reporting on scheduled backups as well, but still need to decide if its worth the effort or if NetWorker can meet our needs.
