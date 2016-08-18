---
layout: post
title: Lessons from the Australian Census
date: '2016-08-19T07:00:00.000+10:00'
author: Richard Banks
tags: 
modified_time: '2016-08-19T07:00:00.000+10:00'
---
If you live in Australia, I probably don't need to provide any background on the problems with the 2016 Australian Census system. Even if you live outside the country you might have heard about the issues we experienced; it certainly got enough coverage.
In case you haven't seen it, or have forgotten, the Australian Bureau of Statistics (ABS) encouraged Australians to respond to the 2016 Census using online means instead of using the traditional paper forms.

The ABS paid IBM just under $10 million to build the 2016 census site and gave ~$500K to RevolutionIT to load test it.
Unfortunately, on census night the system crashed and crashed hard, nominally due to a DDoS attack though reports since then indicate that this was not the case. The census site remained offline for two days and since coming back online has experienced ongoing problems. The political fall out and blame shifting has been impressive.

To be clear, I didn't work on the project and don't know anyone who did, so I don't know why things went as badly as they did. Press reports, however, have indicated there were a number of major problems with the project.

 * Engagement with a vendor (IBM) who has a reputation of not caring about value delivered, instead focusing on delivering only what was specifically asked for and no more, and extracting maximum revenue from their customer.
 * Completely unrealistic expectations of the expected user load. Expectations that everyone else could see were way too low.
 * Untested handling of failure scenarios.
 * A belief that a non-cloud infrastructure could scale to meet the spikes in load
 * Staff who could see the solution was going to fail, but either failed to call it out or were shut down by management.

For more, have a read through the various links at the end of the post for where this information has come from.

Of course, not long afterwards a few students put together a clone of the census site over a weekend for a few hundred dollars and handled a much higher load. They're quoted as saying: "I was very surprised they didn’t go with cloud infrastructure but that gets into the whole security thing."


## Architecting for Scalability

So the question for me is, what would I need to think about if I was designing the Census site? What considerations would I need to make?

At its core the Census is a simple data entry collection system.
Users log in to the system, using a pre-provided reference number (effectively, their correlation id) and then fill in their data.
The form itself has some dynamic aspects to it, but it's a low-medium complexity data entry form.
Census data is then saved in a database of some  kind, and once the census is closed, that data will be analysed by data scientists in a variety of ways.

So, for me, my big design considerations would be:

1. Scalability (duh!):

    Spikes in load are going to occur. You know that. I know that. Everyone knows that! Why the ABS/IBM thought a steady user load of 500,000 submissions an hour was realistic is beyond me, and beyond everyone I've talked to about it. I'd design for everyone trying to submit their submissions in a half hour period. Design for a very large spike, not an even flow of users.
    In any case, if I wanted to ensure availability during a spike I'd lean on autoscaling. I wouldn't self-host (the census site is hosted at IBMs Baulkham Hills data centre) as self hosting requires me to pre-purchase all the capacity I think I'd  need before hand. Using a cloud provider means I can add capacity on the fly, based on demand, and scale back down once any spikes are over, keeping my costs to a minimum.
    I would try and avoid building a solution based on VMs as well, as scaling VMs is not quite as easy as scaling a PaaS offering. So for me, I'd prefer PaaS offerings as the basis of my design to better support load at scale and a simple management experience.
    Also, unless I was deploying a monolith, I'd like the flexibility to scale different parts of the overall solution at different rates based on where the hot-spots are. 

1. DDoS and other security attacks:

    A national census is a high profile target. We know people will try to mess with it. Have DDoS protections in place, and test it. Remember that if a majority of Australians try to log in to the system on Census night it might look just like a DDoS attack so plan for it.
    Reports that IBM chose to ignore DDoS protections from their network vendor strike me as penny pinching madness.

1. Data storage and sovereignty:

    We know a census form submission application is going to be write heavy so we should choose a data storage strategy that supports this.
    Being more specific, since people were given census ids before the night, wee could use those with a simple key/value storage mechanism to keep I/O to a minimum. Almost all data operations could be O(1), targeting single records via the census id. Azure Table Storage or AWS S3 would be fine for this purpose, and incredibly cheap to use.
    Once the census closes we can easily take this data and load it into any database structures we like, optimised for the analysis needs of the data scientists and do so without impacting the performance on the night.
    I don't know what storage mechanism was used for the Census itself, but the ABS purchasing an extra 600 licenses for IBM Notes scares me a little. I'm hoping it was just for email and not because the data was put into a Notes DB, though anyone who inflicts Notes on people as an email client is unusually cruel.
    In terms of data sovereignty both AWS and Azure have Australian data centres, so there's no issue there. From a security perspective, they're both heavily certified for data security practices, so it's likely that data would be safer there than in your own data centres. There really is no reason not to use them.

 1. Redundancy and failover:

    Disaster recovery and failover practices are critical for a high profile site such as the census. The analysis by [risky.biz](http://risky.biz) on the failure points to a lack of practice in verifying their plan-B scenarios.
    As a counter example, Netflix practices deliberately failing their systems in production all the time (using their ChaosMonkey utility) as a way of ensuring their approaches work. If I was building the census site, I'd be considering doing the same sort of thing, and I'd also look at geo-redundancy (multiple data centres) and having a version in both Azure and AWS so that I could fail over to the second provider in the event that my primary had a (very, very unlikely) catastrophic failure.


What would you do? Drop me a [comment on twitter](https://twitter.com/rbanks54), I'd love to know.

And if you want help with scalability and performance in the work you do, or just want a second opinion, get in touch. I'd be more than happy to have a talk to you about it.

### References

IBM:

 * [IBM's unethical behaviour](http://www.theregister.co.uk/2013/08/06/ibm_committed_ethical_transgressions_to_win_botched_project/)
 * [IBM makes meek apology for Census failure](http://www.theregister.co.uk/2016/08/11/ibm_makes_meek_apology_for_oz_censusfail_offers_no_fail_detail/)
 * [IBM's reputation at an all time low](http://www.arnnet.com.au/article/605089/ibm-reputation-an-all-time-aussie-low/) 

News articles about the Census failure:

 * [ABS site inaccessible on census night](http://www.abc.net.au/news/2016-08-09/abs-website-inaccessible-on-census-night/7711652)
 * [ABS working to rebuild trust](http://www.abc.net.au/news/2016-08-14/abs-working-to-rebuild-trust-after-census-failure-chief-says/7733234)
 * [Census miscalculated demand](http://www.news.com.au/technology/online/ibm-servers-for-australian-census-2016-miscalculated-level-of-demand-in-a-short-space-of-time/news-story/63dee596f1b541446b83afe20d0584e3)
 * [Census servers continue to fail](http://www.news.com.au/technology/aussies-face-more-frustration-as-census-servers-continue-to-fail-despite-repeated-attempts/news-story/4e43f5e92dda5685778601b9b66fddf9)
 * [The real security threat of the 2016 Census](http://www.itnews.com.au/blogentry/the-real-security-threat-of-the-2016-census-433907)
 * [Experts doubt ABS' DDoS Claims](http://www.itnews.com.au/news/experts-cast-doubt-on-abs-census-dos-claims-433926)
 * [Government could sue IBM](http://www.itnews.com.au/news/govt-could-chase-ibm-for-damages-over-census-failure-433519)
 * [Let the blame games begin](http://www.lifehacker.com.au/2016/08/ibm-and-the-abs-census-let-the-blame-games-begin/)

Other:

 * [Twitter #censusFail](https://twitter.com/search?q=%23censusFail&src=typd) 
 * [Census failure analysis](http://risky.biz/censusfailupdate)
 * [Cloud based census system for $500](http://www.news.com.au/technology/online/qut-students-design-a-500-cloudbased-census-server-four-times-better-than-ibms-9-million-system/news-story/0a4eeabf733cedfce0091ce6f062c60c)
 * [Purchase of 600 IBM Notes licenses](https://www.tenders.gov.au/?event=public.cn.view&CNUUID=9F2F60F5-B6E4-5540-FA0002385D7EA7AF)
