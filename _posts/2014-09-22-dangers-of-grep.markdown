---
layout: post
title: The dangers of grep
date: 2014-09-22 00:00:00
---

Grep is a fantastic tool for everyday use in Unix systems and is one of the most highly utilised GNU tools today. However there's a common mistake I see when used in shell and Perl scripts.

![image](https://31.media.tumblr.com/880f24990d875abb8d5553a3652671a3/tumblr_inline_ncbvipStHt1s54kjo.jpg)

Consider the trivial example of the following file:

{% highlight bash %}
$ cat /tmp/grep.me
ab
cd
abc
{% endhighlight %}

Now lets do a normal grep for just the first line:

{% highlight bash %}
$ grep ab /tmp/grep.me
ab
abc
{% endhighlight %}

Now when coding shell scripts on the fly it's quite easy to see our mistake, a partial match on the third line that usually isn't what we wanted. But when prototyping scripts we may miss this, because abc might be added later.

This is very easily solved and anyone who's used grep/regex long enough already knows the answer. However it's common to see even experienced system administrators make this mistake.

{% highlight bash %}
$ grep '^ab$' /tmp/grep.me
ab
{% endhighlight %}
