---
layout: post
title: Driving automation for better and quicker software releases
date: 2014-12-13 00:00:00
---

As the year has come to a close I've been looking back on what I've achieved and where I was 12 months ago. Last year we were in the same place we had been for several years, in a constant patch and release cycle for code provided by our developers. We would continually have to fix software ourselves, sometimes bad by design sometimes just day to day bugs that we find. We also had to add new functionality ourselves as new requirements came through from the business. Often this doesn't get merged upstream.

![image](https://31.media.tumblr.com/541598ebb38d6fa779fcb00d16f9fe77/tumblr_inline_ngip6mZPvU1s54kjo.gif)

Puppet came along early in the year and this enabled us to centralise and standardise the configuration for much of our infrastructure, so we could more quickly and easily roll out changes. This was a huge improvement for us, but it didn't really decrease the number and severity of software and configuration bugs.

The fact was our Quality Assurance processes were duplicated across the dev team and our team. The Dev team would do their own manual set of testing and we would run our own. As the year went on we started to included much more rigorous testing, as we had to test for bugs we had previously picked up but that had not been fixed upstream as well as functionality we'd implemented. But this was all still manual, time consuming and error prone. So how did we streamline this approach and improve quality in the end product at the same time?

![image](https://31.media.tumblr.com/654f0ddd2e9c51304691e8e62b6af771/tumblr_inline_ngipi9PcIH1s54kjo.jpg)

Automation was key. We had automated, somewhat, the release mechanism, so I looked at going backwards and automating the deployment mechanism. We already had some degree of automation with deploying new Virtual Machines, but it was clunky and had far out grown the original scope. This was achieved by implementing vCenter Orchestrator, as I've discussed in my previously blogs.

So next on my list was automating tests. I came away from a Puppet Camp in Melbourne in November charged and ready to tackle this problem, as I'd heard two case studies where clients were using cucumber to automate testing. For testing puppet code the default testing framework is rspec, but cucumber seems to have gained quite a following in recent years for its simplistic language of test steps (implemented in a language called Gherkin that uses Given, When and Then keywords to develop test cases).

I had a lot of trouble initially wrapping my head around how I would complete all the testing, as we needed to do a mix of white-box and black-box testing. The puppet code we had needed to be white-box tested to ensure the internal logic was sound. But we also needed to test the proprietary products we use, for which we cannot inspect the internal state. This requires a different mechanism of testing called black-box testing, where we would need to simply throw information into a lab environment and inspect the result to ensure it matched our requirements.

Eventually, after a discussion with a friend who is a software developer who told me to just jump right in, I started to build my tests for puppet first.

So I implemented rspec testing for our puppet modules and after a few days of bashing my head against a brick wall I was able to confidently eliminate a small number of our manual tests. What frustrated me about rspec though was it's insistence on unit tests, testing just the functionality of a single module and nothing else. This seemed pointless to me as a System Administrator as I wasn't concerned with if a single module had an issue, but if the end to end system functioned.

I hacked away and implemented some integration testing that compiled a puppet catalog of almost an entire server and tested all the pieces existed, and all the variables and logic worked end to end.&nbsp;

At this point I decided to give cucumber a try, to see what all the hype was about, although I was initially hesitant to implement my "testing code" and "test steps" in two different files. Cucumber-puppet is sporadically maintained but still gives the same functionality as rspec. Another few days later and I started to fall in love with the Gherkin syntax that allowed me to define my test code once and write almost human readable tests that eventually consisted of 127 individual tests. Not only that it insisted on a more top down approach, testing all my modules together rather than individual functionality. But the end test steps still are not readable by management, they contain various phrases like "should have resource File[server.xml]" as well as "file should contain content matching &lt;host&gt;db01&lt;\/host&gt;" which really isn't meaningful.

Then I stumbled upon [beaker](https://github.com/puppetlabs/beaker) and the solution to how to do black-box testing became apparent. Beaker is a wrapper of various shell tools and hypervisor APIs that allows you to build/configure a VM, kick off a puppet run with code from your branch and run some rspec tests. It's exactly what I needed, end-to-end testing (called acceptance testing).

There's still some work to do, such as integrating with vCenter Orchestrator which I've already invested heavily in to leverage automated provisioning (checkout my branch [here](https://github.com/nemski/beaker/tree/vco_provisioning)), but I'm almost there. Also I need to fit it in with our existing release workflow, but our designated "testing engineer" should be happy. Once completed I believe I might well have reached&nbsp;_**Nirvāṇa**._

I know this isn't really a new concept within the Software Development community, but too many businesses still fail at the basics. Especially the many companies that spent many many years just "integrating" products that various companies provided without doing real software development. In the Software Defined World, every company is a software company.
