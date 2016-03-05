---
layout: post
title: The AWS model of computing
date: 2016-03-05 21:00:00
---

Before I start shitting on VMware, I'd like to say kudos to them. They really kick started the virtualisation revolution, which has made the world of computing better. Since 2003, when vCenter (or Virtual Center as it was then known) was first introduced until now, we've seen a change from where almost all workloads had dedicated servers, to now where I hear System Administrators say often "Everything we run is virtualised" and "there is no reason not to virtualise". In my short career as an application support engineer turned system administrator I've never commissioned a physical server for anything other than to run some virtualised platforms; ok I've only ever commissioned one physical server, but that alone proves my point.

But then, in 2006, Amazon launch Amazon Web Services (AWS) and there was at first a big change, but from the other end of the spectrum. Not as many workloads moved from virtualisation to Public Cloud infrastructure as those that moved from physical to virtual, but AWS has now become a dominating force in the world of computing.

There are two different models at play here, people call it Public vs. Private cloud, but that doesn't really capture the difference in approaches you take when designing systems in these two virtualisation platforms. Heavily customised, slowly moving infrastructure vs. highly standardised fast paced instances; AWS encourages a standised, ephemeral nature of computing. Occasionally I see people complain about how AWS murdered their instance and how can they run a business where there is no SLA around individual server uptime. The fact is AWS has built their architecture with a very clear view of how they think computing should be done, and it's changing the way we think about infrastructure.

You can build systems in a reliable, fault tolerant manner using VMware and AWS. You can also build them in a repeatable way in either system, allowing you to blow away and rebuild servers at will. You can also build them like pets, with massive uptimes that have long manual provisioning processes, but I think we all agree that's the a bad idea. Only one product actively encourages you to do it the right way.

# Fault Tolerance vs. High availability

AWS's Elastic Compute Cloud (EC2) has a SLA of 99.95% for multiple Availability Zones (AZs) in one region. That means if more than one Availability Zone (AZ) is unavailable for more than 22 minute a month then you can claim a service credit. You can have instances in only one availability zone, so your instances may be unavailable for long periods at any time and this is considered normal. Some people find this surprising. For fault tolerance you should ensure you have instances in multiple AZs and, if possible, multiple regions.

One of VMware's key features is vMotion, which allows you to move instances from one physical host to another without service interruption. I remember the first time I heard about this feature it blew my mind, it is an amazing feat of engineering. However it encourages people to ignore designing their system in a fault tolerant way and instead fall back on this high availability feature. Some applications don't adhere nicely to this architecture, they do not natively support using say a load balancer to distribute requests across nodes. Such applications should be avoided at all costs.

Note what I'm talking about when I say fault tolerance should not be confused with VMware's Fault Tolerance feature.

# API all the things

What Amazon succeeds at most is in the API. [More has been written on this](http://apievangelist.com/2012/01/12/the-secret-to-amazons-success-internal-apis/) which I won't regurgitate, but I encourage you to read it. Someone in the office said AWS v1 SDK sucked the other day and I couldn't help but laugh.

Last I looked the only supported SDKs for the vSphere API were Java, PowerShell and Perl. I've written it in Perl and it sucks, to the point where I switched to an unsupported SDK in Ruby because at least the language didn't suck too. I've written a lot of SOAP API code in Perl but VMware always has a special place for being the most complicated (but not the worst). While using vCenter Orchestrator I also wrote some code using JavaScript, that was a special kind of hell. I've also written it in PowerShell, that doesn't suck too much but it starts to look like C# after a while and I'm not a Windows developer. I don't mind using PowerShell for small scripts, but for large programs it required more knowledge of how to write .NET style code than I was prepared to learn.
Not only are VMware's SDKs in a limited number of languages compared to AWS (their website lists 11 supported SDKs), most of them don't hide the monstrocity beneath.

Here's some Ruby I wrote to add a disk to a Virtual Machine, using the [rbvmomi](https://github.com/vmware/rbvmomi) SDK which has been abandon.

[Ruby](https://github.com/nemski/rbvmomi/blob/master/lib/rbvmomi/vim/VirtualMachine.rb#L90-L130)
{% highlight ruby %}
 def add_disk(options={})
    fail "Please provide a datastore" unless options[:datastore].is_a? RbVmomi::VIM::Datastore

    ds = options[:datastore]
    file = options[:file]
    controller = options[:controller]
    controller ||= "SCSI controller 0"
    file_path = options[:file_path]

    if file_path.nil?
      file_path = ds.find_file_path(file)
    end

    # Construct the full path to the VMDK ([My Datastore] my_vm/my_disk.vmdk )
    full_file_path = "[#{ds.info.name}] #{file_path}#{file}"

    # Build the XML component for the raw disk
    disk_backing_info = RbVmomi::VIM::VirtualDiskFlatVer2BackingInfo.new(
                          :datastore => ds, 
                          :fileName => full_file_path, 
                          :diskMode => "persistent"
                        )

    # Find the virtual SCSI controllers
    vm_controllers = self.controllers

    vm_controller = nil
    vm_controllers.each { |c| vm_controller = c if c.deviceInfo.label == controller }
    fail "Could not find Virtual Controller #{controller}" if vm_controller.nil?

    # Because the unit number starts at 0, count will return the next value we can use
    unit_number = vm_controller.device.count

    # Get the size of the disk
    capacityKb  = ds.get_file_info(file).capacityKb

    # Build the XML component of the disk controller mount
    disk = RbVmomi::VIM::VirtualDisk.new(:controllerKey => vm_controller.key, 
                                          :unitNumber => unit_number,
                                          :key => -1,
                                          :backing => disk_backing_info,
                                          :capacityInKB => capacityKb)
    # Create an add disk operation
    dev_spec = RbVmomi::VIM::VirtualDeviceConfigSpec.new(
                :operation => RbVmomi::VIM::VirtualDeviceConfigSpecOperation.new('add'),
                :device => disk
              )
    # Create a device change event
    vm_spec = RbVmomi::VIM::VirtualMachineConfigSpec.new( :deviceChange => [*dev_spec] )

    puts "Reconfiguring #{self.name} to add VMDK: #{full_file_path}"
    self.ReconfigVM_Task( :spec => vm_spec ).wait_for_completion
  end
{% endhighlight %}

The Perl code is just as heavy and difficult to write.

Here's some pseudo code on how you do something similar in AWS, attaching an EBS volume to an EC2 instance.

{% highlight ruby %}
volume = client.describe_volumes({
  filters: [
    {
      name: 'tag-key',
      value: 'volume-name'
    },
    {
      name: 'tag-value',
      value: 'bacon'
    }
  ]
})

resp = client.attach_volume({
  volume_id: volume.first
  instance_id: ec2
  device: "/dev/sdb"
})
{% endhighlight %}

The v1 SDK requires a bit more code, you have to loop through a list of every volume looking for one that matches.

Ultimately I'm not sure if the problem is entirely with the API itself, I think the amount of effort Amazon has put into its SDKs also goes a long way.

# Platforms and Infrastructure

AWSs Platform as a Service (PaaS) offerings are another great differentiator to the VMware/Private Cloud offerings. I recently wrote some code to setup a staging database from a snapshot, it's about 96 lines of code. I didn't have to install or configure MySQL, I booted an RDS instance, seeded the database then deleted it and allowed it to create a final snapshot. The code (written in SparkleFormation which is a wrapper for CloudFormation) simply creates a new RDS instance from that snapshot.

Doing this with VMware would've taken me days, instead it took me hours.

# Wrap up

Moving to a Public Cloud offering like AWS takes a lot of fore thought, you need to plan and re-architect your environment to provide resiliancy and flexibility. But the ultimate result, if your architecture and application are mature enough, is an environment that supports much faster iterations and more resiliant, fault tolerant designs. Still AWS is not always the right choice; [Netflix recently spoke about their AWS migration project](http://arstechnica.com/information-technology/2016/02/netflix-finishes-its-massive-migration-to-the-amazon-cloud/) and noted they left their DVD business in their own datacenters, they  don't need the scalability AWS provides so the cost of migrating and running it in AWS doesn't make sense.

But the AWS model is something not to be ignored:

- Build in Fault Tolerance not just HA. Not just because it's more resilient but because it forces you to make cattle not pets.
- Have an API that's easy to use and abstracts the complexity from the developer.
- Use PaaS to abstract complexity of building boilerplate services.
