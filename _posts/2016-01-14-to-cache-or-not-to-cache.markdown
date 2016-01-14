---
layout: post
title: To Cache or not to Cache
---

This is my analysis of the Steam Christmas caching incident, from my experience as a System Administrator who has worked with Akamai and understands a bit about how this kind of problem can arise.

# What we know

For 1.5 hours on Christmas Day 2015 Steam's website started serving authentication pages from a public cache. Specifically this issue was rather problematic for the accounts page (https://store.steampowered.com/account) as there was personally identifiable information served to persons other than the account holder. While the personal information leaked was minor (only user's email, billing address and some of but not all of their phone number and credit card number), it's a major embarrassment for the company.

In Steam's [official statement](http://store.steampowered.com/news/19852/) they mention the issue was caused by a mis-configuration with their "web caching partner". A simple query of their DNS records shows this parter is Akamai:

{% highlight bash %}
18:57 $ dig +trace steampowered.com
...
steampowered.com. 172800  IN  NS  a9-67.akam.net.
steampowered.com. 172800  IN  NS  a11-67.akam.net.
steampowered.com. 172800  IN  NS  a24-64.akam.net.
steampowered.com. 172800  IN  NS  a1-164.akam.net.
steampowered.com. 172800  IN  NS  a26-65.akam.net.
steampowered.com. 172800  IN  NS  a8-66.akam.net.
;; Received 318 bytes from 192.33.14.30#53(192.33.14.30) in 1364 ms

steampowered.com. 20  IN  A 23.9.237.210
;; Received 50 bytes from 2.16.130.64#53(2.16.130.64) in 34 ms
{% endhighlight %}

Akamai, via it's Kona product, provides an always on Distributed Denial of Service (DDoS) protection and Content Delivery Network (CDN). The two go hand in hand. By serving static assets from Akamai's many "edge" nodes distributed around the globe, they are able to provide faster access to resources from a source closer to each customer while reducing load on the "origin" servers.

# What might have happened

From this knowledge I'm going to draw together various hypotheticals based on my experience with Akamai and deploying rule changes using their Managed Service.

## Hypothetical one

The configuration change was made by Akamai's engineering team, likely with approval of Steam's engineers.

I'm assuming this based on this sentence from their official statement "During the second wave of this attack, a second caching configuration was deployed that incorrectly cached web traffic for authenticated users." and the offering from Akamai of their 24x7 Security Operations Center (SOC), who will analyse on-going DDoS attacks and instrument rules to mitigate them.

## Hypothetical two

Akamai configured the CDN to ignore Cache-Control headers from Steam and cache pages regardless of the "no-cache" header flag.

This is a common setting in Akamai's Kona product, specifically for static content (including json and xml files). But it can be configured for Dynamic content as well, based on if a cookie is set or not. This is detailed in a [blog post by Akamai](https://blogs.akamai.com/2015/10/dynamic-page-caching-beyond-static-content.html).
Akamai's interface is rather confusing in this way. Under caching options, selecting "Cache" means "Ignore origin headers". You have to explicitly tell it to respect origin headers, which is rather unexpected.

## Hypothetical three

Steam did not ask Akamai to deploy the change to staging for testing before it was rolled out.

I sympathise with why this may not have been done. One, it's possible there was no testing process in place or the process was manual and lengthy. It was Christmas Day, it was a very busy period for Steam and they were under an intense DDoS attack. The pressure to get it done and get back to Christmas lunch would be considerable. I'm sure if you stuck me in the same position, without the knowledge I have now, I probably would've made the same mistake.

## Hypothetical four

There was either no support staff available on Christmas Day, or the support staff did not have the appropriate training to respond to the incident.

It took them 90 minutes to resolve the issue, that's a hell of a lot of time considering it takes only 10 minutes to roll back the change and it should take even less time to put up a maintenance page. It's easy for support staff in this scenario to not fathom the seriousness of such reports until many of them come in. But, with appropriate training and run books, they should realise after a single report that this needs to be escalated quickly and with the highest priority.


Unfortunately I was not at aware of this issue when it happened, so I couldn't run some debugging on Steam's website to validate some of my assumptions, so take them all with a grain of salt. However I feel there are important lessons for everybody in these hypotheticals.

# How it might have been avoided

## Lesson one

Extra care must be taken when co-ordinating changes with partners. Managed Service Providers (MSPs) might not fully understand your environment and you possibly don't understand theirs. This is a perfect storm of assumptions on both ends; the provider might assume that forceably caching certain pages won't cause any issue, the customer might not understanding that the setting will ignore Cache-Control headers. Build extra robustness around how and when these changes occur, never expect the MSP to understand your environment enough to not make a fatal mistake.

## Lesson two

Configuring a CDN to ignore Cache-Control headers is especially dangerous. Akamai's product language and documentation is not especially clear in this area. Whenever you on-board a new service like this be sure to do your homework and try and nut out potential pit falls like this before they happen.

## Lesson three

Deploy all changes to a testing environment and make tests automated. [akamai-rspec](https://github.com/realestate-com-au/akamai-rspec) is a great library for testing changes in Akamai's staging environment and includes matchers like `not_be_cached` so writing a test for Steam would look like this:

{% highlight ruby %}
describe 'does not cache authenticated pages' do
  it 'the account page' do
    expect("https://store.steampowered.com/account").to not_be_cached
  end
end
{% endhighlight %}

Which ensures that the page is not served from the Akamai cache and that Akamai's CDN does not consider the page to be cacheable.

## Lesson four

Security Incidents need appropriate escalation paths. Sometimes companies get lucky, the right support person picks up a ticket, someone who notices the issue has the right contacts to escalate the issue and bypass the support desk. Don't rely on luck to know when you have a security incident, train support staff on how to deal with them and empower them to escalate even on Christmas Day.
