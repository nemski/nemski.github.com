---
layout: post
title: "The use case for time series data"
date: 2017-01-16 00:00:00
---

My favourite conference swag has to be a shirt with "Data Nerd" on the front. My first technical job was implementing and maintaining what I came to know as "Time Series Datastores" for Service Providers and large companies with millions of data points. At the time the only solutions that scaled to that size were commercial offerings that used a relational database under the hood such as Oracle and later MySQL.

I spent a lot of time educating people on how data is modeled in such datastores and how to store and retrieve data in the most efficient and accurate way.

Since then there's been an explosion of specialised, Open Source, Time Series Datastores that are fronted by tools like Graphite, Grafana and Prometheus. They have made it possible to get the scalability and features you would previously pay millions of dollars for, virtually for free.

# The use case for time series data

As [HoneyComb](https://honeycomb.io) pointed out in their recent [blog](https://honeycomb.io/blog/2016/12/the-problem-with-pre-aggregated-metrics-part-1-the-pre/) [post](https://honeycomb.io/blog/2016/12/the-problem-with-pre-aggregated-metrics-part-2-the-aggregated/) [series](https://honeycomb.io/blog/2017/01/the-problem-with-pre-aggregated-metrics-part-3-the-metrics/) time series data has it's limitations. To summarise:

- You have to define what metrics you want to collect up front
- You can't define attributes for metrics that have a high cardinality
- Time series doesn't track individual events and allow you to correlate them to trends

However it does scale very well; you can store a lot of data very efficiently and, provided you structure your queries correctly, query large datasets very quickly. Because you can store data efficiently you can keep very fine grained metrics for short periods of time to understand the impact of performance regressions in a small time window or look at the long term trends. The data can be graphed in many different ways, from simple line graphs to pie charts and heat maps.

With the maturation of commercial and open source offerings the time from asking a question to having a graph that provides the data you need can be very short.

# How it works

It's important, especially when developing instrumentation, dashboards and reports, to understand how time series datastores work.

## Push vs. Pull

The inevitable question that comes about when implementing a new monitoring tool is "do we pull the data from its source, or push it in". My preference is to push the data because, while Google advocate for a pull mechanism (and hence that's what Promtheus provides), the complexity that adds is only worth it when you scale above tens of thousands of devices. Not having to persist the state in a, potentially ephemeral, device has many benifits.

## Data types

The data we are attempting to track can be represented in many different ways. This depends on what kind of data we're trying to represent.

### Counter

If we're trying to represent how many successful requests a web server has served we can simply use a counter that continually increments. In a push mechanism we send a single transmission (usually over UDP) indicating how much to increase the counter by. In a pull mechanism we increment an internal counter that is exposed to our monitoring system.

A Counter should only ever increment, however in a pull based architecture we may hit an integer overflow and have to start at 0 again.

A downside of the pull based mechanism is we have to expose to our monitoring service more fine grained aggregates about the rate at which the counter increased. In a push based architecture the monitoring service is able to determine this itself even when data is batched up and sent in bulk. Without this we can easily miss intermittant spikes and lulls in a course grained rate calculation of the change over a minute.

### Gauge

When we want to represent how long some action takes, whether it be serving a web request or transmitting a frame, we want to provide our monitoring service with some sample of that data. In a pull based architecture we'd need to expose the avg, min, max etc of that timing, but with a push based architecture we can use Timers/Histograms.

### Timers and Histograms

These are essentially gauges but include a percentage of timings (samples). This simplifies the client implementation as we just send some data and let the monitoring system aggregate the timings.

## Aggregates

Notice all the above values are just integers or floating points. This makes maths really easy and enables some significant benefits when dealing with a large number of values. All time series based monitoring systems aggregate the data into at least six basic types:

- Average
- Sum
- Minimum
- Maximum
- Count
- Last

From there we can represent an extensive number of possible values such as percentiles (99th and 95th), standard deviation and rate of change to within a reasonable degree of accuracy. We do this by rolling data up into sample periods.

## Data Rollups

A sample period is a period of time a metric represents. A time period of 00:00 until 00:01 is a sample period of 1 minute.

By taking the raw values from a sample period we can apply the aggregate functions and stores all those aggregates in one database entry. From there we can roll our 1 minute sample periods into 5 minute sample periods without the raw data. This allows us to discard the, potentially large number of, raw values and reduce space. Not only that we make the space required to store the data a linear calculation of the number of metrics we store not the number of raw values we receive.

From my previous work I calculated that rolling raw data values up from 5 minute intervals to 1 hour intervals was accurate to between 1 and 5 percent, with the 5 percent being for percentile calculations. A lot of modern systems include 90th and 95th percentiles in the list of default aggregates, which reduces the overal accuracy to about 1 percent. This 1 percent was usually due to rounding so with high enough precision floating point values this inaccuracy can be effectively eliminated.

## Example data

So with that, lets see what the rolled up data might look like from some example raw values. If we have the following raw values (written in statsd format):

```
patrobinson.github.com.get.load_time./:320|ms|@1
patrobinson.github.com.get.load_time./:169|ms|@1
patrobinson.github.com.get.load_time./:502|ms|@1
patrobinson.github.com.get.load_time./:312|ms|@1
patrobinson.github.com.get.load_time./:290|ms|@1
```

We produce the following entry for that sample period:

| Namespace     | Count | Sum  | Avg | Min | Max | Last |
| ------------- |:-----:|:----:|:---:|:---:|:---:| ----:|
| patrobinson.github.com.get.load_time./ | 5 | 1593 | 318.6 | 169 | 502 | 290 |

# In Summary: Optimise for the most common use case first

Time series monitoring tools are extremely efficient and provide rich features for a variety of use cases. While Graphite has been a staple for many years, a lot of new tools have been released that support multi-dimensional tagging, alerting and increased scalability. 
