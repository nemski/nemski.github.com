---
layout: post
title: vCO with JavaScript Objects
date: 2014-10-16 00:00:00
---

I sit here with a whisky and dry trying to drink away the day I wasted yet again on solving a semi-complex problem with vCenter Orchestrator.

My grand plan for vCO was to use it to integrate with Puppet by reading and writing JSON (to a file now and a key-value store later) to be used as a Hiera backend. Since the basis for vCO code is JavaScript I thought this would be an easy fit. How wrong I was.

As it turns out the vCO JavaScript implementation is wrapped around with Java. This means that when you pass variables between objects in vCO they have to be converted into Java objects and then back into JavaScript. vCO provides it's own datatypes to handles this, but they have limitations.

The equivalent to an Object in JavaScript is a Properties object, which provides put, get and keys methods. But it doesn't support nested objects unless you declare them as Properties objects as well. When you're reading or writing JSON, this isn't really possible as the serialize/deserialize (American spelling for ease of reference) methods don't convert them appropriately. Instead they get defined as "Java.lang.String" class. The same happens when passing between workflows, the Objects get cast as something else and the nested Properties are clobbered.

The workaround I've found is to not pass pure JavaScript Objects between workflows, only between workflows and actions or actions and actions. When passing the objects to/from an action they seem to stay intact.

This of course requires rewriting some simple workflows as actions, all in JavaScript.
