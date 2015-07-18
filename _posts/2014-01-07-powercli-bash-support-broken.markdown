---
layout: post
title: PowerCLI Invoke-VMScript bash support is severely broken
date: 2014-01-07 00:00:00
---

PowerCLI is the PowerShell API toolkit for vSphere. It's quite a simple and easy to use solution for day-to-day scripting of vSphere tasks like trying to find a list of Virtual Machines, Snapshots older than a some time frame, Datastores running out of space and much much more.It's a must have for administrators anything larger than a small vSphere environment.

Invoke-VMScript is a "cmdlet" that allows you to run commands on the remote host via PowerShell. This has been used extensively by us to automate the build of new Virtual Machines and migration of existing machines to a newer OS/platform. We chose to use it because the only other alternatives are Java (too complex, no internal knowledge) and Perl. SOAP APIs with Perl are tedious at best and this is no exception. Perl just doesn't handle the complex SOAP data-structures in a manageable way. What takes 25 lines of PowerCLI can take 200 lines of Perl.

I've known for a long time PowerCLI's Invoke-VMScript support for bash was buggy but today I've established it is FUBAR. Up until today I've discovered two common bash commands that behave unexpectedly when run from Invoke-VMScript.

{% highlight bash %}
( exit 1 ); echo $?
{% endhighlight %}

This will return 0 from Invoke-VMScript, when it should return the value given to exit in the sub-script (1 in this case). When I pasted this to VMWare community forum (the supported method for logging bugs since we don't have the appropriate contract to log a ticket against PowerCLI), and bugged their PowerCLI twitter account for a response, I was told to use the "ExitCode" Object returned by Invoke-VMScript and I went on my merry way.

{% highlight bash %}
[[ $(/bin/true) ]] || echo TRUE
{% endhighlight %}

This should return TRUE always, but when run from Invoke-VMScript we get:

{% highlight bash %}
bash: -c: line 0: syntax error near `]]'
bash: -c: line 0: `[[ ]] || echo TRUE'
{% endhighlight %}

Simply assigning a command output to a variable and echoing it also doesn't work.

{% highlight bash %}
A=$(grep alias ~/.bashrc |tail -1); echo $A
{% endhighlight %}

This returns nothing. Running 'ps -ef' while it's executing reveals what I believe to be the core of all these issues:

{% highlight bash %}
/bin/bash -c bash > /tmp/vmware-root/powerclivmware255 2>&1 \
-c "A=alias mv='mv -i'; echo ; sleep 30"
{% endhighlight %}

The way it's been implemented, it appears to be executing parts of the code one at a time, rather than the whole script in its entirety. Even when we use backticks (`) instead of $() we get the same results. This makes anything but the most basic bash commands useless.

Last example, $! doesn't work:

{% highlight bash %}
/bin/true && echo $!
{% endhighlight %}

This should return the PID of the command we spawned (/bin/true in this case) but again we get nada.

I can make my scripts into files, ship them to the VM and run them, but this takes more commands, requires dos2unix to be installed on the remote host and is frankly a hack.

I don't enjoy PowerShell, it behaves very differently to what I'm used to in a scripting language so I think once I've completed the current project I will look at&nbsp;rbvmomi&nbsp;to see if we can use Ruby instead.

UPDATE: VMware have advised this is a known issue but due to "legal" reasons they cannot advise when this will be fixed. Either something really weird like a breach of Open Source licensing is going on or they're lying!
