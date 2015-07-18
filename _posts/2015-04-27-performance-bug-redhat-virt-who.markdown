---
layout: post
title: Performance bug in RedHat’s virt-who
date: 2015-04-27 00:00:00
---

A few months ago I was troubleshooting RedHat’s Subscription Asset Manager (SAM). This is used as an internal licensing server and RPM repository, as opposed to the RedHat Network.

The issue I was chasing was when we deployed a new Virtual Machine, it was taking the SAM server up to 30 minutes to identify which ESXi host the VM was running on and hence correctly register it and allow it to download packages. Our deployment process for a new server was already 30+ minutes, so adding another 30 minutes to wait for SAM to catch up was a little over the top.

The process which discovers VMs is called virt-who, written in Python. It was scheduled to run every 15 minutes but was taking 15 minutes to complete. So having barely touched Python I dived in deep with plenty of time and print statements (i’m sure there’s a better library to do this for me). This eventually led me to a section of code, the core of the entire program.

> # Get list of host uuids, names and virtual machines

At first glance it appeared to be well optimised:

{% highlight python %}
object_contents = self.RetrieveProperties('HostSystem', ['vm', 'hardware'], hostObjs)
{% endhighlight %}

This line got a list of Hosts, it’s VM and Hardware properties and then proceeded to iterate through them.

{% highlight python %}
for host in object_contents:
  vmObjs = [] # List of objs for 'VirtualMachine' query
  if not hasattr(host, 'propSet'):
  for propSet in host.propSet:
  if propSet.name == "hardware":
    self.hosts[host.obj.value].uuid = propSet.val.systemInfo.uuid
  elif propSet.name == "vm":
{% endhighlight %}

Then it iterates through each property and if it’s a VM we obviously want to learn more about it. Now we get to the crust of the issue

{% highlight python %}
  for vm in propSet.val.ManagedObjectReference:
    vmObjs.append(vm)
    v = VM()
    self.vms[vm.value] = v
    self.hosts[host.obj.value].vms.append(v)
    # Get list of virtual machine uuids
    vm_object_contents = self.RetrieveProperties('VirtualMachine’, ['config’], vmObjs)
    for obj in vm_object_contents:
      # Get each VMs uuid and put into a dict
{% endhighlight %}

Got it? Ok so Python doesn’t start and close blocks in the way traditional languages do, usually with curly brackets ({}). Instead they start with a colon (:) and any code indented by some white space (i’m not sure on the amount, usually 4 or a tab) is within that block. There is no real end to the block, code after the block is just not indented.

So why then are we getting a list of properties for VMs in _vmObjs_ more than once? Why is that for loop embedded within the loop that gets a list of VMs?

The reason is, it’s not meant to be. Un-indenting that second for loop resulted, in my relatively mediocre vSphere environment of 326 Virtual Machines, in a 90% performance increase. 90 seconds as opposed to 15 minutes. Someone else I spoke with who had over 1000 VMs reported a 98% increase in performance.

I submitted a [pull request](https://github.com/virt-who/virt-who/pull/2/commits) to RedHat via Github, but I’m not sure if the fix has been released yet. I think it’s being tracked via [this](https://bugzilla.redhat.com/show_bug.cgi?id=1195585) Bugzilla request
