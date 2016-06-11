---
layout: post
title: A model for DevOps
date: 2016-06-11 11:30:00
---

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">When you have a &quot;devops team&quot; that acts as a system owner you&#39;re not fostering shared responsibility</p>&mdash; nemski (@DrNemski) <a href="https://twitter.com/DrNemski/status/741435137594822656">June 11, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

I see a lot of people talking about a "DevOps" team and a lot of people talk about how that's an anti-pattern, "DevOps isn't a team or a role, it's a methodology" is usually the argument and I agree with that. But nobody talks about a model for how you should structure people and teams when trying to learn from DevOps. I've learnt these structures from various sources, so I thought I'd try and summarise them. But first lets lay out some possible architectures that we might want to apply these models to.

### Monolithic Application ###

We all known a monolithic application, it's one big wooly mammoth with varying degrees of technical debt. It's grown organically over the years and requires a large team of people to maintain and develop.

### Micro Services ###

Small applications, loosely coupled, each with their own team. Each team implements and manages their own systems and interface with eachother via well defined APIs. The boundaries of responsibilities are very clear.

### Micro Monolith ###

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">I see you have a poorly structured monolith. Would you like me to convert it into a poorly structured set of microservices?</p>&mdash; Architect Clippy (@architectclippy) <a href="https://twitter.com/architectclippy/status/570025079825764352">February 24, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

This is a slightly different approach, you might have many applications (small to large) that are tightly coupled. Boundaries of responsibility are not clear and there are dependencies between systems such that each cannot be changed independant of the other. Changes usually require a long process of testing and validation (often manual).


So with that out of the way lets define some possible team structures and where they might best apply.

### Site Reliability Engineering (SRE) ###

A dedicated team of Operations (possibly with Developers) that take responsibility for the day to day maintenance of the site. They are not solely responsible for system uptime, but it is their primary concern. They are not "gatekeepers" to the system, more like sheppards watching over a flock. This model is more appropriate to a monolithic application, since its likely its size and level of technical debt require more focus and resources to keep the system up while improving resiliance and reliability.

### Cross functional teams ###

A team of developers and operations staff that work together on a single system. Team members share responsibility and benefit from cross skilling. This model fits a micro-services architecture where each team has their own infrastructure and is loosely coupled to other teams and applications. The team does act as system owners because responsibility boundaries don't cross teams; but no one person has that ownership.

### Insourcing ###

This more closely resembles a traditional IT team, but with less system ownership. The "insourcing" team acts in various ways to support the micro and monolith teams. They may support common infrastructure that's used by multiple applications and they may consult or even build infrastructure for other teams. This structure can fit any model but usually complements the other two structures. The team has ownership of systems but they don't enforce that teams have to use the systems they build. This way they are accountable for building systems that enhance flexibility.
