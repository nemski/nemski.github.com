---
layout: post
title: Multiple format outputs for Rspec
date: 2015-03-12 00:00:00
---

I’ve been playing with Atlassian Bamboo to automatically execute and report on our rspec tests. We have unit and acceptance tests, which are executed via rake tasks.

I found a fantastic gem [rspec_junit_formatter](https://github.com/sj26/rspec_junit_formatter) which allows you configure Bamboo (and other CI tools) to parse your rspec test results. My problem was I wanted to keep the original formatting (documentation) for when the command is run interactively, and I needed to separate the unit test results from the acceptance test results. The project readme suggests adding options to .rspec file to append command line options, but that means writing outputs from unit and acceptance tests to the same file, which would make it difficult for bamboo to pick both results up.

So here’s how you do it, from within your spec helper!
{% highlight ruby %}
RSpec.configure do |c|
  c.add_formatter "RspecJunitFormatter", "spec.xml"
  c.add_formatter :documentation
end
{% endhighlight %}
