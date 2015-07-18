---
layout: post
title: Introducing a Ruby Gem the ServiceNow API
date: 2015-02-04 00:00:00
---

[rs_service_now](https://github.com/nemski/rs_service_now)&nbsp;is a Ruby Gem that acts as an interface to ServiceNow. At the moment it only supports two operations, request and export (which do the same thing, but via different mechanisms), but I plan on expanding this to support more methods in the future. Export is an experimental approach at the moment, where as the request method is tried and true.

This was my first foray into writing a Gem, being motivated by a previous experience re-writing a backup script that was procedural with a more Object Oriented approach. I also used an IDE called RubyMine, which I recommend you check out if you do a lot of Ruby coding. It highlights code in real time to let you know of variables that are not used, not declared or code that is not syntactically correct and allows you to build and push Gems from within the IDE.

Anyway, rs_service_now will act as a wrapper for much of the work I do with custom code exporting data from ServiceNow, by allowing you to get the information with a single method call. The ServiceNow SOAP interface is infamous for restricting such exports to a limited number of rows and makes it difficult to get a large dump. The process to then get the export is difficult and the resulting data type differs if you are returned one vs. more than one record (hash vs. array of hashes).
