---
layout: post
title: Exploring vCenter Orchestrator
date: 2014-07-11 00:00:00
---

I recently deployed vCenter Orchestrator (vCO) as a PoC for replacing our current VMware automation solution, a custom PowerShell script. The current solution had become unworkable as it was cumbersome to add functionality (nobody was proficient in PowerShell in our team) and our requirements out-grew a single script.

![image](https://31.media.tumblr.com/8989f8a8eb7fd8f1478575d74108b841/tumblr_inline_n8jrgkYjZT1s54kjo.jpg)

Our options were:

* Rewrite the 900 lines of PowerShell in a more modular fashion

* Rewrite the code in another language more familiar to our team (Ruby or Python)

* Use Foreman (which tightly integrates with Puppet)

* Use an Orchestration platform like vCO or vCAC

The first two options would require weeks of development work and testing. Ruby and Python don't have as mature an API library for VMware as PowerShell so they would require additional development. Foreman looked promising at first, but its expectation of being the single source of truth didn't fit our requirements.

We have recently deployed Puppet in our environment (see my other blog posts for details) and are in the process of moving to Hiera to better manage our business data (currently stored in a 4,500 line site manifest as well as various files on each server). We have grand plans for how to unify the various processes that control provisioning and enabling functionality for our clients, but are yet to realise how that will fit together.

Currently provisioning a client requires touching various stacks, from the networking and virtualisation to the operating system and application. Adding or modifying behaviour of the services we deliver to a client is also dispirit. We're predominantly a Linux and VMWare environment, so the tools to solve these problems are plentiful. But choosing the right tools and tying them all together is the challenge.

With this in mind I started trying to replicate the existing functionality with vCO. Within a few hours I'd created a workflow that Cloned a VM, set the appropriate CPU and RAM resources and connected it to a network. This was all using out of the box workflows and actions and it worked seamlessly, but each component had it's own flaws.

The CloneVM task won't allow you to clone to a Datastore Cluster, which we utilise, since the API calls are different (PowerShell handles this transparently through the New-VM cmdlet), so I had to write a new action in JavaScript (the language of choice for vCO). The change CPU action only modified sockets, our Redhat licensing gives us unlimited cores but restricts us to two sockets (stupid but unavoidable), so I wrote a custom action to modify cores per socket as well. Finally the default mechanism for choosing a network required the user to select from an inventory list. This list is in no particular order (some places in vCenter client it's now alphabetised but not everywhere) and choosing from over 200 networks is cumbersome. So I wrote an action that finds all networks that match a name pattern given as input and created a decision workflow that prompts the user if there's more than one match.

![image](https://31.media.tumblr.com/8ca4b03ead4d43f010658353a4a17a19/tumblr_inline_n8jv3bmxyb1s54kjo.jpg)

It took a couple of days to get the hang of writing custom actions, but I've done a lot of development with the vSphere API so it's relatively easy to pick up.

My next steps are to automate the bootstrap process, to get us to a point where we can run our provisioning scripts. This will require integrating with Puppet (to sign SSL certificates via the REST API) and Hiera (probably just dump a YAML file in-place at this stage with host specific information).

That's the easy part, my eventual dream is to create a single web-portal that feeds data into the Hiera backend and Orchestrator (or possibly vCAC) and be a single pane of glass for our operations and provisioning teams to manage a client's services life-cycle. Because the service we deliver to our clients is constantly changing, as well as the way we deliver it, a single portal would allow us to scale so much better in terms of the resources needed to manage those services.

A special thanks to @rnelson0 on twitter who pointed out vCO as a solution. vCO is licensed automatically with vCenter Server so is free for most VMware shops.
