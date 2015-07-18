---
layout: post
title: Clearing cache from command line behind a proxy
date: 2014-11-04 00:00:00
---

Quick trick, adapted from something my boss showed me, on how to clear a cache from behind a proxy.

If you're using a web browser you can just use CTRL+F5, which I think most people know already. Squid proxies will respect the "no-cache" header, flush the local copy and retrieve a fresh copy from the source.

At the command line this can be done as follows:

{% highlight bash %}
curl -H 'Cache-Control: no-cache' $URL
{% endhighlight %}

Which has the same effect. This is useful when you get a yum error such as:

{% highlight bash %}
http://yum.puppetlabs.com/el/6/products/x86_64/repodata/afad322c697e457d7ee608ccca3ef235a3efb34b-primary.sqlite.bz2: \
[Errno 14] PYCURL ERROR 22 - "The requested URL returned error: 404"
Trying other mirror.
{% endhighlight %}

Which is because the repomd.xml is out of date and refers to a repo db that no longer exists. Therefore you need to flush the proxy cache and yum cache as follows:

{% highlight bash %}
curl -H 'Cache-Control: no-cache' http://yum.puppetlabs.com/el/6/products/x86_64/repodata/repomd.xml
yum clean all
{% endhighlight %}
