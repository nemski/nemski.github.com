---
layout: post
title: Integrating vCenter Orchestrator with Puppet
date: 2014-11-07 00:00:00
---

Today I'm excited to describe how I have integrated vCenter Orchestrator (vCO) and Puppet.

![image](https://31.media.tumblr.com/0fa655adabb8aa846650cf8fb0b6de45/tumblr_inline_nepcz8lSjG1s54kjo.jpg)

I've been working with vCO for several months now and there are several ways you can integrate it with Puppet to provision Virtual Machines. The first requirement is vCO itself, which is luckily free with vSphere Enterprise. Secondly you'll need a Puppet master (Open Source or Enterprise) using Hiera. Hiera is a fantastic tool to separate your business logic from your configuration data, more information [here](https://docs.puppetlabs.com/hiera/1/).

vCO allows you to provision a virtual machine using the vSphere API (using a VM template) using a "workflow" that can drive your commission (and decommission) tasks end to end. For instance our vCO workflow does the following:

*   Creates a host record in our DNS server (InfoBlox)
*   Clone's the VM to a Datastore Cluster
*   Creates a dvPortGroup in a specific VLAN
*   Connects a the VM to the appropriate dvPortGroup(s)
*   Adjusts the RAM and CPU as configured by the user
*   Provisions the host in our backup software (EMC NetWorker)
*   Boots the VM
*   Stores configuration data for the host in a Hiera backend
*   Kicks off a provisioning script which in turn runs puppet agent
*   Deploys the host to Opsview (Nagios) via a REST API

vCO is highly configurable and integrates with almost any SOAP and REST web service to allow for dynamic workflows that can be adapted your business requirements (depending on how willing you are to hack some JavaScript).

So how does the Puppet integration work? Well there are multiple options to do this. You can fork over a lot of money for VMware vCloud Automation Center (vCAC) and Puppet Enterprise and get out of the box integration, detailed [here](http://puppetlabs.com/solutions/cloud-automation/vmware/vcloud-automation-center). You can also use static JSON files as a Hiera source and write to them via vCO, but this is clunky and error prone (what happens if multiple workflows try to change the same file at once?).

Luckily I stumbled upon a hiera backend plugin called [hiera-http](https://github.com/crayfishx/hiera-http)&nbsp;(if you're going to use this be aware there's a bug with false values being returned as nil, which I've submitted a pull request for [here](https://github.com/crayfishx/hiera-http/pull/17). It's fairly simple to patch your local installation). The author gives an example on how to configure hiera to retrieve data from a CouchDB NoSQL database [here](http://www.craigdunn.org/2012/11/puppet-data-from-couchdb-using-hiera-http/).

CouchDB is perfect for our purposes because it natively supports REST (in fact it's the only API it supports) and JSON data!

Not only can you provision and decommission hosts, you can also instrument vCO workflows that modify the Hiera data for you, avoiding the need for IT staff to manually update flat files. It creates a single entry point for all change which is highly configurable, failure resistant and easy to use.
