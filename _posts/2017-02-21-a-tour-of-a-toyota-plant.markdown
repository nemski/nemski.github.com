---
layout: post
title: A Tour Of A Toyota Plant
date: 2017-02-21 00:00:00
---

Today I was priveledged to visit the Toyota plant in Altona, Victoria. The plant is due to be shutdown on October 3rd 2017, so we were extremely lucky to be able to visit one of the last vestigates of Automative Manufacturing in Australia. Australia was the first Toyota manufacturing plant outside of Japan, opening in 1963. At first it simply built the cars from kits, but over time developed into a large operation located on the current 75Ha site. We were equally lucky that this plant is one of the only contained on a single site; so we were able to walk from the building that forges and assembles the engines to the plant that presses the body panels right through to where vehicles rolled off the production line. The entire process takes 24 hours, for a car to go from a bunch of very basic parts to being ready to deliver. Each car is made to order, with the limited customisations each consumer can choose. It's assembly is tracked from the body and engine fabrication through to the paint and assembly phases. The plant only ever produces the parts it needs (Just-In-Time Production) and orders some parts on demand (i.e. the seats, produced in Laverton and delivered within 2 hours, as requred) and others ahead of time, such as parts that come from overseas.

Toyota Production System
----

The Toyota Production System (TPS) is used by many companies in many different industries to improve efficiency and speed up a companies production process, manufacturing or otherwise. But how does this relate to Information Systems? In previous places I have worked the teams produced software releases on a semi-regular basis (every few months) and that release would be provided to my team to deploy to production. This process required increadible amounts of manual testing and deployment by our team, which increased the lead time to delivery by many more weeks. At best a new feature might take eight weeks to be rolled out. But inevitably there would be problems with the new release we would send back to the development team to be fixed, which would require another release cycle or a hotfix. This re-work slowed the process down even further.

The most efficient processes focus on minimising rework and increasing flow of production. Flow is much more evident on the assembly floor, as vehicles inch down the production line (which snakes backwards and forwards across a massive building) as workers attach parts to the cars at each stage. Flow is less evident in modern computer systems, but can be seen as we complete one task and move on to the other.

The more rework that is required, that is the more we have to go back and do a task again or find out where it went wrong, the less efficient we are. 

Jidoka
----

Throughout the process "autonomation" or Jidoka is evident. Machines produce and quality check parts, but the human involvement is ever present. Humans need to be able to trust the machine to do it's work without intervention, but if there's a fault the machine should either be able to fix it itself or stop the process immediately.

Often we see automation in IT that requires continuous human interaction. This has been most evident to me when somebody says "Just run Puppet until it succeeds". This is "Muda", or as Google call it "Toil". Human involvement will always be necessary to some degree, but it should be eliminated as much as possible by implementing "Kaizen". When there is a failure, it should be immediately detected to avoid rework.

Takt Time
----

Throughout the plant tour we were reminded of the Takt Time, how long each step would take. Often when people consider their processes, they optimise for speed without considering the impact on the process as a whole and without understanding why going faster is better.
Throughout the process it was extremely evident how tunable the speed of the plant was. Takt time when we visited was 136 seconds, but could go as low as 65 seconds. Every process in the plant was adjustable to take account for this change.

Having greater control and insight into our Takt or Cycle Time allows us to provide estimates to stakeholders about how long a piece of work will take. The less accurate we are with our estimates the less efficient our entire development process is. If we finish a task late the piece of work is pushed out and the team working on it may have to reprioritise other work, pushing it out even further and increasing Work In Progress.
If we finish a task ahead of time the next chain in the line is not ready to receive it. Since now we pick up another task we are again increasing our Work In Progress (WIP). More WIP interrupts our flow by forcing us to change from one process to the next. In the factory this is seen when the dyes are changed in a press, in IT we see it as context switching where our brain takes time to get into the context of the work we're focussing on.

Kaizan
----

A process of Continuous Improvement is implemented at a grass roots level. Each team is encouraged to develop initiatives to improve quality and reduce rework. An example was pointed out to us in the body work shop, where bonnets had hinges attached and were stacked on a trolley to be moved to the next stage. During stacking it was common for bonnets to fall against each other, causing scratches and dints. Not only were they checking these parts at this stage, to avoid sending them to be painted only to have them returned; the workers of this plant prevented the initial damage by hanging a tennis ball on a string on top of each bonnet.

This, combined with Genchi Genbutsu or "go and see for yourself", ensures we find failures at the source and address them at the source. Building resiliency into an upstream system to deal with failures of a downstream process is generally desirable as it's innevitable this failure will occur, but we should always try to fix a problem at the source first. It also encourages us to learn from the people on the ground, those that are intimately involved in the process often have the best knowledge about it.

Pokayoke
----

Error proofing is at the core of the Toyota Production System. When we accept errors are inevitable we can get on with the process of discovering and mitigating them. This is implemented within the automated systems in place which immediately stop the production line when an error is detected. Automated tests are an excellent example of Pokayoke. They enable us to quickly and efficiently detect errors and fix them before they are sent to the next stage of the process (referred to as the "customer" in TPS).

If faults can be fixed within the Takt Time, then the production line continues. If they will take longer they are pulled from the line. We fix a problem if we can, so as not to interrupt our flow, or we move it to the side to be dealt with later or by another person.

Conclusion
----

There was a lot to take in, I was fascinated by the technical and personal processes and how humble the system was. People were at the core of everything.

It was sad that the plant wouldn't be around for much longer, but I'm honoured to have seen and experienced it for myself.
