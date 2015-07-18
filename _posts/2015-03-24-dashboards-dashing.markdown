---
layout: post
title: Dashboards with dashing
date: 2015-03-24 00:00:00
---

I was reading an [article by Dataloop.io](http://blog.dataloop.io/2015/03/18/using-monitoring-dashboards-to-change-behaviour/)&nbsp;about using dashboards to change behaviour. It was a good coincidence because I’d just stood up a Continuous Integration tool to let the team know when our puppet code fails tests, but had this niggling thought in the back of my mind&nbsp;“who, other than me, will look at this?”

The problem is we have so many dispirit systems that it’s easy to forget about one for a period of time, or the person who usually looks after it is on leave. So how do we compile all that information in one place?

Mentioned in that article (which I recommend reading btw) was something called Dashing, a Ruby framework for building dashboards. As someone who’s only done web development under duress, I dived into it and was surprised at what I managed to achieve in two days.
![image](https://41.media.tumblr.com/36e1573f09dd8361f849e2ac19a5f275/tumblr_inline_nlplieoNlt1s54kjo_500.jpg)

I was able to churn out integration with [ServiceNow](https://gist.github.com/nemski/4f3a62c052e7952c222d), [Opsview](https://github.com/nemski/dashing-opsview)&nbsp;and EMC Networker to present the three most valuable pieces of information to our team on a day to day basis. And, most importantly, I barely had to get my hands dirty with the web dev side! Most of my time was spent creating dashing jobs to collect the data.
