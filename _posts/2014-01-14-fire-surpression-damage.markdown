---
layout: post
title: How a fire suppression system caused more damage than it prevented
date: 2014-01-14 00:00:00
---

Our company uses a Managed Data Centre to host the bulk of our equipment. Due to the inherit complexity and overheads pertaining to building and maintaining a DC, this always made sense to me. Putting your servers in a back room has various risks associated to it such as when power to your building is out, your public facing systems are also offline.&nbsp;

My assumption here, based on experience, is our Hosted DC is much less likely to experience a power outage and has more redundancy than our offices do. We do not have a DR Data Centre due to the high cost and lack of management buy in.

However Hosted DCs are not perfect, the one we use doesn't provide redundant internet pipes instead we have to buy our own. This means, again due to cost, we don't have redundant internet.

Then on Sunday night at about 19:44 a cooling unit overheated due to a collapsed bearing, burnt out and caused a general fire alarm. The Fire Brigade attended and assessed the situation as safe, the units power was removed but all systems on the floor remained up and unaffected... or so they thought.

As part of the fire alarm, a fire suppression agent, Inergen gas,&nbsp;was released to contain the fire and reduce the impact to equipment in the DC. This is just one of the complexities of a modern Data Centre, traditional fire suppression systems such as Dry Powder are completely unsuitable for IT equipment as they are sucked into the chassis via the fans and coat all surfaces. This powder needs to be removed before you power the units back on or you will most certainly cause permanent damage. Power would therefore need to be cut when this was occurring.

However Inergen Gas systems are not perfect either. As we've become painfully aware of&nbsp;gas based fire suppression release can and does kill hard disks. There is both&nbsp;[anecdotal evidence](http://lists.ausnog.net/pipermail/ausnog/2014-January/022079.html)&nbsp;and&nbsp;[scientific evidence](http://www.datacenterjournal.com/it/inert-gas-data-center-fire-protection-and-hard-disk-drive-damage/)&nbsp;that loud noises, exceeding 110dB, can significant reduce performance and cause drives to fail. Releasing inert gases at high pressure produces such sound waves.

From our experience the failures can manifest as SCSI Bus resets which can bring a system to a grinding halt or force the disk into Read Only mode, SMART disk failures or data corruption. In two cases we had both firewalls in a cluster go down, but were able to power-cycle them to recover a single member and restore service, the other side of each cluster has to be rebuilt. Some disks show OK when we check them via Management Agent, but refuse to boot due to corrupted partition tables or file systems. There specifically seems to be something wrong with the Hard disks shipped by HP, Our EMC SAN threw some SCSI bus resets but no failures and our EMC DataDomain backup system had a single disk failure, while we lost 9 HP SAS drives, 7 of which are the same part number.

We'll be working with the DC and HP to understand how we can mitigate this issue going forwards.

Additional reading:

[http://www.hgi-fire.com/wp-content/uploads/2013/03/White-Paper-potential-problems-with-computer-hard-disks-V1.3.pdf](http://www.hgi-fire.com/wp-content/uploads/2013/03/White-Paper-potential-problems-with-computer-hard-disks-V1.3.pdf)

[http://www.datacenterjournal.com/it/inert-gas-data-center-fire-protection-and-hard-disk-drive-damage/](http://www.datacenterjournal.com/it/inert-gas-data-center-fire-protection-and-hard-disk-drive-damage/)

http://www.availabilitydigest.com/public_articles/0602/inergen_noise.pdf

[https://www.youtube.com/watch?v=uuEylKMvbjw](https://www.youtube.com/watch?v=uuEylKMvbjw)
