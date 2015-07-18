---
layout: post
title: Continuous Integration of Puppet code
date: 2015-03-16 00:00:00
---

We have a project with almost 10,000 lines of puppet code, which until recently had 0 tests. I’ve been working for the past few months to change this by implementing unit tests using [rspec-puppet](http://rspec-puppet.com/) and acceptance tests using [beaker](https://github.com/puppetlabs/beaker) (which heavily leverages [server-spec](http://serverspec.org/) under the hood).

![](https://31.media.tumblr.com/bb4b01d9174e370e83f8b1bbe5c36d80/tumblr_inline_nlb1hjMNhO1s54kjo.jpg)&nbsp;&nbsp;![](https://31.media.tumblr.com/0b3c1589873746b91c90034e9af88b78/tumblr_inline_nlb1hooe7g1s54kjo.png)&nbsp;&nbsp;![](https://31.media.tumblr.com/df68418a4e0d112c31952121f0513ee8/tumblr_inline_nlb1htIcSR1s54kjo.png)

What kicked off my desire to start this was a Puppet Camp I attended in Melbourne last year where a couple of the speakers talked about using Bamboo and Cucumber to execute them. Being the curious admin I am, I decided to trial Atlassian Bamboo along side JetBrains TeamCity to see which I liked better (I eliminited Jenkins without really looking at it due to it's perceived lack of useability).

Bamboo, TeamCity and Jenkins are a range of tools used to test, compile/build code and deploy it into test/QA/prod. Really I only need it for testing, there is no compilation or build and there isn't enough effort involved to justify automating deployment of code yet.

## Bamboo vs. TeamCity

Both tools offer what I needed, integration with Rake to run our tests (Bamboo required a plugin to be installed) and the ability to view test results in a well formatted way (Bamboo required some massaging see my previous [post](http://nemski.tumblr.com/post/113416918044/multiple-format-outputs-for-rspec)).

But TeamCity provided a much easier interface to configure my tests and the ability to use build configuration templates with parameters set in individual build configs that modified the default behaviour. This was very useful, as we have two seperate VCS roots; a local repository of code and a vendor import. The location of the vendor import depends on which code stream is being used (branch vs. trunk).
Both are great tools though and I recommend trialing both if you're looking for a Continuous Integration (CI) tool.

![image](http://i.imgur.com/YcQI5EJ.jpg)

## 

![image](http://i.imgur.com/Pxbik7R.jpg)

## Enter RubyMine

I'd also been trialling RubyMine, another JetBrains product, as an IDE for Ruby development. I'd been working on a number of Gems and found it's project templates as well as integration with rubygems, rake and bundler an eye-opener into the value of IDEs (having been a text-editor coder up until now). So much so that I forked out AU$133 of my own cold hard cash for it.

What I discovered today was it's native support for Puppet and integration with TeamCity. I can now modify my Puppet code in RubyMine (which brings with it support for automatically renaming variables and auto indenting) and then kick off TeamCity builds before I check my code in!

This is a game changer, I will never develop puppet code the same way again.
