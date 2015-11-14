---
layout: post
title: The Monolithic Nature of Puppet
---

In this post I'm going to talk about what I see as a problem with the current design patterns for Puppet. These are design patterns that are promoted by Puppet Labs on their website and by their Professional Services engineers. If you haven't read Gary Larizza's [Puppet workflow blog series](http://garylarizza.com/blog/2014/02/17/puppet-workflow-part-1/) I highly recommend it. What I have to say heavily relates to the concepts laid out in that series, specifically [laying down a CI pipeline for your Puppet code](http://garylarizza.com/blog/2014/03/07/puppet-workflow-part-3b/). The TL;DR of that is to build a Puppet CI pipeline you need the following:

- Code tags/branches/revision numbers, whatever you use you need something to define what is production and what is dev.

- Automated Tests; you need some kind of testing around your recipes/manifests. Rspec-puppet and ServerSpec work quite well to do this. Rspec-puppet provides unit and integration tests for your manifests, while ServerSpec acts as an acceptance test.

- Continuous Integration; this runs your tests and gives a green or red build. I've used TeamCity and Buildkite to do this, but Bamboo and Jenkins can do it also. This requires that your automated tests orchestrate the provisioning of your infrastructure, which can prove difficult. In the past I've resorted to running it on already provisioned hosts (not ideal). [Beaker](https://github.com/puppetlabs/beaker) provides some ways of doing this also by restoring from a vsphere/ec2 snapshot or spinning up vagrant hosts, but it has a high barrier of entry.

- Continuous Deployment; r10k provides a great way to do continuous deployment through git branches. You will have your feature/bug fix branches that serve as individual environments in Puppet (called dynamic environments). Combined with webhooks it will automatically deploy commits to the appropriate branches. Commits to master will be deployed to the master environment (which might be your QA branch if needed), so you need a production branch. Merging master->production when there's a green build on master is a common git workflow and this can also be automated.

Automating as many steps as possible is the goal here. To me the best workflow possible is:

- I commit my changes to a branch
- I submit a PR for review
- Once my changes have been reviewed and I have a green build I merge

That's it, from there everything should be automatic. The Puppet recommended way is slightly different:

- Commit changes to an individual module in it's own repo, on a branch
- Commit changes to another branch in the "control" repo which reference the previous change in the Puppetfile
- Submit a PR on the individual module, but refer to tests run on the "control" repo for the build status
- Merge changes into the individual module

The extra step is necessary when you don't have a monolithic Puppet repository, instead each module is split into it's own repo and a control repo contains the Hiera data and Puppetfile. While the effort to split different modules into different repos is commendable it is not entirely polylithic. Each server has a role in the "roles" repository, and dependencies between roles and individual modules are not easily deduced. When a change is made to one module all roles must be tested.

It might be possible to design a CI system that determines which modules have changed by compiling a catalog for each role and comparing it to a catalog on the master branch, then only run acceptance tests for roles with catalogs that have changed.

The intricate nature of Puppet makes this a challenging problem and I'm still not sure of Puppet's relevance in an Immutable world. But we're not there yet, something for another blog post.