---
layout: post
title: Configuration Management vs. Immutable Infrastructure
date: 2017-05-20 00:00:00
---

I've been working with Configuration Management for a number of years. I started off with Puppet, managing a few hundred servers. I [integrated it with vCenter Orchestrator](./2014-11-07-integrating-vcenter-orcehstrator-puppet.markdown) (now called vRealize Orchestrator), which Puppet conveniently now ship as a [plugin](https://docs.puppet.com/pe/latest/vro_release_notes.html#vro-puppet-plug-in-20) of their own. But having spent a lot of time working with Puppet and vSphere I was left feeling the solution was still too complex, especially if you wanted developers to self service, and too fragile.

Recently Anthony Shaw released a post about [Ansible, Salt and StackStorm](https://medium.com/@anthonypjshaw/ansible-v-s-salt-saltstack-v-s-stackstorm-3d8f57149368), compairing their offerings. I've played with Salt and Ansible a little, but never used them seriously. They still feel like a "leaky abstraction", there's too much complexity and domain knowledge required to use them.

I spoke at [DevOps Days Sydney](https://www.youtube.com/watch?v=I3DuUzGC-SU) about how mutability was a spectrum. There are lots of use cases for Configuration Management, a lot of applications, particularly "enterprise" apps, are not [12 Factor](https://12factor.net/). They distribute state across many machines, requiring machines to be backed up and maintained for many months or years. This requires us to continuously manipulate the state of these machines, we can automate it but instead of a herd of cattle we end up with a herd of Wagyu Beef. Sure we can shoot a cow and buy a new one, but it still takes a lot of massaging.

We can move some way away from this by making our most crucial changes, our application deployments, immutable. When we seperate out the application configuration and code from the underlying infrastructure that supports it we can deploy it with different tools like Docker, AWS CodeDeploy or Capistrano. If we still have mutable infrastructure though this still results in error prone infrastructure changes; we may have a variety of different states in play at one time and the unknown state brings risk. I call this the Frankenstein model, half mutable half immutable.

To achieve the ultimate in operational efficiency though we need to be fully immutable. Now when I say this most people probably imagine Docker on top of an Orchestration system like Kubernetes or Swarm. But these are complex beasts, requiring more domain knowledge than Configuration Management. You also have the problem of how to manage databases like MySQL or Postgres that are not designed to run on Containers, attaching volumes, managing secrets, running strong consistency key-value stores like etcd, the list goes on.

Somewhere though I think there is a balance. Sure tools like Kubernetes make sense at a certain scale, because the amount of engineering effort pays off. But at most organisations I've worked at they don't make sense.

So if you came here for answers, all I have is more problems. Because in the end it's always just a bunch of trade offs.

![](http://fox.mmgn.com/mmgn/news/normal/hacked-xbox-live-experiencing-network-issues-psn-attackers-suggest-it-s-them-1112628.jpg)
