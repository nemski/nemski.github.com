---
layout: post
title: "Reasonable Steps to Securing NPM"
date: 2018-11-27 00:00:00
---

Two months ago someone going by the name of [right9ctrl](https://github.com/right9ctrl) published a malicious version of the package [event-stream](https://www.npmjs.com/package/event-stream) to NPM, including what some have determined was malicious code designed to steal bitcoin.
The public history of this issue can be found on an [event-stream Github issue](https://github.com/dominictarr/event-stream/issues/116).

The code was obfuscated via minification, not present in the unminified version of the code, and used encryption to make it even more difficult to determine it's intent.

---

This is not the first time news of a malicous NPM packages has been made public, [in July a malicious version of eslint was published which siphoned npm tokens](https://github.com/eslint/eslint/issues/10600).
Both packages have 2 million or more downloads a week.

There's been much debate on how to improve the security of npm packages. Some of the issues stem from the lack of a robust standard library for JavaScript, resulting in a large number of small packages maintained by disprit developers who collectively feel the weight of managing a successful Open Source software project.
Many point fingers at these maintainers, or at Open Source Software in general. However we've been effectively and securely developing and releasing Open Source software for decades.
Here I will not point fingers. Instead I will suggest two ideas that I think can improve the security of NPM packages without impacting on it's agility.

Suggestion One: Reproducible Builds
---

For this specific incident the malicious code was only present in the minified package that shipped to users. The attacker had succesfully hidden it by including an irrelevant library, [flatmap-stream](https://github.com/hugeglass/flatmap-stream), and removing traces of this from the Github source code.
Transparency into what code a package actually contains is at the core of trusting packages that come from NPM.

The idea around reproducible builds has been [around for a long time](https://wiki.debian.org/ReproducibleBuilds/History#Tell_the_tale). If we think of the minified version of these packages as "binary", which in essence they are since no human can read it without reverse engineering efforts, than the same approach can be taken.
It would be a lot simpler to implement too. Since 2013 Debian has been working to make all of it's packages reproducible and so far they are [94% of the way there](https://lists.debian.org/debian-devel-announce/2017/07/msg00004.html). NPM would not have to contend with the many issues that variances in build environments produce for Debian packages.

Suggestion Two: Core Maintainers for a Standard Library
---

At the heart of these woes is a sprawling web of dependencies that even the smallest projects inherit. This fragmentation of efforts opens avenues for attackers to use in a supply chain attack. Just one maintainer has to be the weakest link and any downstream application can be compromised.
In this case the attacker simply emailed the maintainer offering to take over development and they were given NPM access. Such a simple vector is going to continue to be successful until the core utilities required to make JavaScript functional are brought under one or few domains to manage and maintain.
Pretty much every other successful Open Source project has a team of core maintainers, often funded by corporate donations. NPM is well placed to start such a venture, but it requires a seperate not-for-profit organisation to be setup and funded by large corporations such as Google and Microsoft.

Summary
---

While NPM and JavaScript have revolutionised the modern internet, they are still relatively immature in comparison to other Open Source projects such as Linux and Firefox.
In order to continue to be innovative NPM must learn from the many OSS projects that have come before them.
