---
layout: post
title: "Docker in Production: A retort"
date: 2016-11-05 00:00:00
---

The internet has been awash with a well written article about the dangers of running Docker in production today. [Docker in Production: A History of Failure](https://thehftguy.wordpress.com/2016/11/01/docker-in-production-an-history-of-failure/).

The piece was well written and touched on the many challenges of running Docker in production today. However towards the end it tailed off into a rant filled with rhetoric that was a knee jerk reaction to a problem many IT folks have experienced: new and shiny does not bring home the bacon, plain and boring does.

So let me try to retort the claims in this article from my experience of running Docker in production.

## Docker Issue: Breaking changes and regressions

Docker maintains API versioning to support backwards compatibility. However there is a lack of long term support for docker engine[1].

There is also much debate about how Docker maintains the runtime environment and image formats. Since the engine is rather monolithic, the runtime and image formats change at a great rate. This is something Redhat and Google are trying to conteract with the Open Container Initiative (OCI). There is even talk they may fork Docker.
It's good that there is open debate about moving Docker towards a standardised stable format and away from a monopoly driven by the goals of one company.

So the point is valid but misleading, the only breaking changes that exist in Docker are in the internal implementation and there are some big names invested in splitting those internal implementations into open standards.

## Docker Issue: Can’t clean old images

This is a well known issue, I was surprised the author seemed unaware of [docker-gc](https://github.com/spotify/docker-gc) by Spotify. This is a fairly trivial solution that is similar to the cron solution offered, but which you don't need to develop or maintain yourself.

## Docker Issue: Kernel support (or lack thereof)

I was surprised I'd never experienced this issue before. That being said I've been building Docker recently on Ubuntu Xenial (LTS), which is running kernel version 4.4.x. Other teams in our organisation using Docker are running Amazon Linux, which has been running version 4.x of the kernel for some time.

## Linux 4.x: The kernel officially dropped Docker support

This is where the article goes from factual to a bit of a flame of Docker. Yes aufs was not accepted into the mainline linux kernel because it's code was described as "dense, unreadable, and uncommented"[2], but it's still supported by Docker and available for every kernel version I've tried to install it on. It's also one of only two filesystems listed as "production ready" on Docker's website[3]. Therefore the statement:

> There is no unofficial patch to support it, there is no optional module, there is no backport whatsoever, nothing. AUFS is entirely gone.

is entirely untrue.

## Bonus: The worldwide docker outage

I remember this outage and seeing the github post describing that the Docker repository was not supported 24x7. But honestly it wasn't a big deal. Depending on package repositories to be available to run your CI pipelines isn't ideal, but sometimes necessary. If it's of such a concern you should really be running your own mirrors.

## Docker Registry

This is what I really came to this article to read. Docker Hub is very unreliable and something we discourage using internally. But again I was surprised the author hadn't come to a solution similar to ours, using AWSs EC2 Container Registry (ECR). Since they mentioned they ran in AWS, this seems like an obvious choice. It's S3 backed and provides timestamps for when images are created to enable easier cleanup.

## Banned from the core

As a lot of people say in this industry, boring tech is what makes money. This is a well accepted principal in many operations teams.

## Banned from the DBA

Again, well accepted principle that "thou shalt not run a database inside a container". Don't do it, end of story.

## A Personal Opinion

This article is a mixture of facts and personal opinions. Some are justified, but it seems like the person writing it is looking at the problem a bit too emotionally. Obviously they've been bitten badly by a rush to implement Docker without proper assessment. From my personal experience the measured approach of putting smaller services in Docker has worked well for us. Some production critical services do run in Docker and we've had issues with them, but the blast radius has been contained (pun intended) and we've learned some valuable lessons along the way that help us as an organisation transition.

> Docker applications should be run in auto-scaling groups. (Note: We’re not fully there yet).
> Whenever an instance is crashed, it’s automatically replaced within 5 minutes. No manual action required. Self healing.

Ok now I understand why the author is upset. This is a good lesson to learn, something to be trialled out not used in production for 12 different applications before finding out.

> The orchestration systems to achieve that currently do not exist. (That’s where Nomad/Mesos/Kubernetes will eventually come in, there are not good enough in their present state).

Plenty of companies are using these today in Production. Plus there's also AWS's offering EC2 Container Service (ECS). While not as good as the others listed, it works and integrates nicely with AutoScaling Groups and Elastic Load Balancers.

## Summary

There are a number of valid problems with Docker raised by the author.

- Docker API changes with every release.
- Support for releases is short, requiring users to upgrade to fix bugs.
- Docker maintained services are not stable.
- The state of orchestration for Docker is still relatively immature.
- Transitioning to any new technology is hard. Especially when that technology requires a shift in how you architect and build systems.

But I'm optimistic that it's possible to overcome these all in the short and long term.

## Edit 6/11/16

Lots of feedback on reddit about my post, which I love. There were a couple of points I probably didn't make clear enough so here goes.

I was not intending in anyway to be condescending to the author of the original post, Thehftguy. I apologise if this has caused any offence.

My intent was to show how I consider Docker to be production ready for our organisation (because we are using it), but to concede there are a number of problems with it. Production ready means different things to different organisations, it's important that each makes their own decision. I'm not trying to advocate that Docker is production ready for everybody, it's not. I am trying to give people all the information they need to make the decision themselves by correcting some of the factual innacuracies and misleading statements in Thehftguy's post.

## Edit 2 6/11/16

An earlier version of this blog post implied that the Docker API was not backwards compatible. Docker were at pains to point out to me this is not true, so the post has been changed to clarify that position.

[1]: https://github.com/docker/docker/issues/20424
[2]: https://lwn.net/Articles/327738/
[3]: https://docs.docker.com/engine/userguide/storagedriver/selectadriver/
