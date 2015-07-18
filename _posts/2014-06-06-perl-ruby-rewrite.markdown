---
layout: post
title: Perl to Ruby - Why I spent weeks re-writing my code
date: 2014-06-06 00:00:00
---

Last year I wrote a backup orchestration script in Perl that talks to various different applications to lock a database, snapshot the underlying block device and back it up. I won't go into the reason we chose this method to backup our database, as that would take a whole post in itself, but rather the reason I decided to re-implement my solution in Ruby.

![image](https://31.media.tumblr.com/7e7ff04a51437bb1a6c982c328c0bedb/tumblr_inline_n6qg2bEUCb1s54kjo.png)

First lets talk about the vSphere API. As with a lot of modern APIs it's based on a SOAP web services framework, which introduces some fairly complex datatypes. For instance [this snippet](https://gist.github.com/nemski/2a851db33c41380952da) in Perl finds all VMFS volumes that are currently not mounted to any ESXi host due to being "unresolved" and re-signatures the one that matches our $vmfsLabel so it can be mounted.

The first two lines involves using a vSphere library function get_views to retrieve the&nbsp;datastoreSystem object associated to the ESXi host specified by $host and using it to query for unresolved VMFS volumes.

> my&nbsp;$datastoreSystem&nbsp;=&nbsp;Vim::get_view(mo_ref&nbsp;=&gt;&nbsp;$host-&gt;configManager-&gt;datastoreSystem);
> my @unbound = $datastoreSystem-&gt;QueryUnresolvedVmfsVolumes();

Now I'm not a developer, so I won't try and analyse why this sucks other than to show you the equivalent Ruby code:

> @unbound_volumes = host.configManager.datastoreSystem.QueryUnresolvedVmfsVolumes

Not only is it one line as opposed to two, it's significantly cleaner. We're using accessor methods to get the datastoreSystem object and calling the query method directly on it.

Next we loop over our returned array and once we find the correct VMFS volume push the extent paths into another vSphere object

> my @extpaths;
> my $extents = $volume-&gt;{'extent'};
> foreach my $ex (@$extents) {
> push @extpaths, $ex-&gt;devicePath;
> }
> my $res = HostUnresolvedVmfsResignatureSpec-&gt;new(
> extentDevicePath =&gt; \@extpaths
> );

Needing to wrap the instance object we want to access in curly braces (the quotes are optional) is ugly, while in Ruby we can use a standard accessor method. I've seen worst examples where SOAP object returns either a hash or an array of hashes that must then be de-referenced. While whoever decided to implement this type of object in SOAP should be shot, it's certainly a lot easier to deal with in Ruby.

This probably comes down to the API library (both provided by VMware) and the ease of which it is to create accessor methods in Ruby as opposed to Perl.

Also Ruby allows you to use an "each" method for an array and supply a code block to it, so the equivalent is much shorter and nicer:

> @extpaths = []
> dstore.extent.each { |ex| @extpaths.push ex.devicePath }
> @res_spec = RbVmomi::VIM::HostUnresolvedVmfsResignatureSpec.new(:extentDevicePath =&gt; @extpaths)

You can find the full Ruby implementation [here](https://gist.github.com/nemski/4f57c61a497c117bd681).

Although my main reason for re-writing this script in Ruby was because the API library contained real accessor methods, which made the code much cleaner and easier to read the real benefits came from exception handling, especially the method ensure functionality. This combined with the inbuilt Ruby Timeout module provides a really easy way to execute time sensitive code that must never hang and cleanup after that code executes, whether it succeeds or fails. For example:

> Timeout.timeout(SECONDS) { lock_n_snap(opts) }
> 
> lock_n_snap opts
> &nbsp; # Lock database
> &nbsp; # Create snapshot
> ensure
> &nbsp; # Unlock database
> end

This code will throw an exception if the time out expires but still execute the code within the ensure block, so we never leave the database locked for more than the interval we specify, which is vital.

I continue to use Perl, it's proven to be faster and have less overhead in certain scenarios (see [benchmarks](http://benchmarksgame.alioth.debian.org/u64/which-programs-are-fastest.php)&nbsp;here). But I think the benefits of Ruby in the scenario above are obvious.
