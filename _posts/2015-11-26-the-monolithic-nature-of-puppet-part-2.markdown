---
layout: post
title: The Monolithic Nature of Puppet (part 2)
---

In [my previous blog post]({% post_url 2015-11-14-the-monolithic-nature-of-puppet %}) I discussed how the monolithic nature of the Puppet code base (even when using a control repo) makes it difficult to do Continuous Integration. I had some interesting discussions with people on Twitter about it and I think my implementation of testing may differ from some, but it was great to here everyone's thoughts. From what I understand of it (and it's easy to mis-interpret people's views when they're restricted to 140 characters), the alternative is to test just your individual modules not the implementation.

But in discussions with a colleague today I remembered Puppet is monolothic by nature in the catalog as well. The concept of classes in Puppet is a misnomer, each class or manifest is simply a namespace. When a catalog is compiled all resources are combined in a single artefact. This introduces some interesting problems, for instance sometimes when you haven't expressed your dependancies correctly (you are ensuring a file without requiring the package that creates the underlying directory for instance) it works anyway simply because the resource coincidentally gets executed first. Although the order of resource execution appears to be random (at least when not using the future parser) it is deterministic. But then you go and change something else in a completely different manifest and it starts failing.

Take for example the following pseudo-code:

{% highlight puppet %}
class Pig {
  file { 'Bacon':
  }
}

class Smoke {
  file { 'Hickory':
  }
}
{% endhighlight %}

Whether Bacon or Hickory gets created first is deterministic, but not predictable and certainly not obvious.

Expressing dependancies can become really hard also when we want to require an entire class be run first, like something that sets some firewall rules or creates some apt repositories. There are a few workaround to this, mainly the anchor pattern and in newer versions containment of resources. They help to group resources in a class together, but can easily lead to dependancy cycles and a fragile code base. Again this is because all resources are so inextricably intertwined, not just through explicit dependanices but implied dependancies (through automatic relationships). When these dependency cycles occur sometimes you have to refer to dot graphs that, if printed out, would stretch several meters wide.

You can also use stages to break up your Puppet run. Stages allow you to break up the monolith into composable parts, but you cannot have any resource dependancies between stages, only stages can be dependant on other stages. The documentation recommends you do not use stages except [in the simplest of classes](https://docs.puppetlabs.com/puppet/latest/reference/lang_run_stages.html#limitations-and-known-issues). The added complexity of stages makes them useless and appears to have been an attempt to fix an issue in a design decision in Puppet, classes are not really classes they are just namespaces.

If they really were classes then we could express dependancies between classes instead of resources. This would require some re-work of how existing modules are implemented. For instance I may have a module that sets up a custom apt repository, uses a third party module to configure the application, then does some custom configuration like replacing the init script with an in-house built one. The sudo code for this might be implemented like so:

{% highlight puppet %}
class Profile::Unicorn {
  apt::source { 'my.apt.repo':
    ...
  }

  class { 'unicorn':
    ...
    require => Apt::Source['my.apt.repo']
  }

  file { '/etc/init.d/unicorn':
    ...
    require => Class['unicorn']
  }
}
{% endhighlight %}

Now if dependancies were expressed between classes instead of resources, we would have to split this out into three classes; pre-unicorn, unicorn and post-unicorn. This wouldn't be as easy to write but it would solve a lot of problems. For instance the implementation I just gave won't work unless the unicorn class is contained. Also automatic relationships could become problematic and possibly unworkable.

This would have the added benefit of mapping which classes include other classes much easier. Compile a class, draw a relationship graph and get all classes within it, which would go some way to solving the problem I highlighted in [my other blog post]({% post_url 2015-11-14-the-monolithic-nature-of-puppet %}).
