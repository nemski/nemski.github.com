---
layout: post
title: Stop doing shitty health checks
date: 2016-06-24 00:30:00
---

I was at Velocity Conf this week in Santa Clara and had a great time watching some brilliant talks by people in the epicenter of the revolution of Ops. It was great to chat and learn from people working in some of the biggest companies in tech (Netflix, Google, Apple). I struggled to write this blog post because there was so much awesome content that came out of it.

One thing really stuck with me though was Donny Nadolny's talk on Debugging Distributed Systems. He described an issue they had where ZooKeeper would lock up and, due to the hard dependancy they had on it, the entire application would come to a stand still (although gracefully degrade). The issue came down to how ZooKeeper attempts to "catch up" followers/slaves and do blocking IO writes while maintaining a lock, which can result in the system becoming completely deadlocked in the process. Not only this the Leader would respond fine to the "ruok" health check command, because the healthcheck thread was no deadlocked. This resonanted with me a lot as it reminded me of a bug I once spent 18 months trying to nail down, but due to lack of ability to replicate the problem and no visibility of the source code I was unable to troubleshoot effectively. The biggest problem here and in my scenario was the lack of an effective health check or heartbeat to tell someone the system was not responding. I even had a Police station complain they could not respond to emergency calls because we had not notified them their system was down so they could revert to manual systems.

Let me define something here first. A health check is a pull mechanism where another system polls for the status of another system, where as a heartbeat is something that pushes an event to another system to say everything is OK. The mechanisms serve the same purpose but are useful in different scenarios.

The emergency services incident got my thinking a lot about heartbeats and health checks. The system I was supporting had some 6 different applications, most of them third party proprietary systems, that handled an event before it was presented to a user. At the time we were aware of silent failure scenarios impacting 3 of them and had implemented a heartbeat across that system to tell us when they were down. After this particular incident I had no choice but to implement a heartbeat across all 6 paths.

When the root cause of the issue was finally found, it turned out (similar to Donny's case) there was an edge case where threads could become deadlocked but the "heartbeat" mechanism, the one instrumented inside the application not by me, was blissfully unaware of it.

When designing software, ensure your heartbeats and/or health checks do more than just say "All is OK because I'm ok", have them actively track the internal state of the application and it's threads. Are they still running? Are they deadlocked?

Finally doing end to end health monitoring is critical. It's not enough to establish the availability and status of each individual component in the critical path, we must establish the end to end state. Only then do we truly know the status of our infrastructure. Recently when I was designing a health check for a web application we instrumented checks for every hard dependency in our application and ensured proper functioning of that system where we felt the normal health check was insufficient.

Stop writing shitty health checks, these are the most important instrumentation your application has. Nobody gies a shit about how fast your caching layer is if you can't service any requests.
