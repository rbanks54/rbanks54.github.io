---
layout: post
title: Lessons from the Australian Census
date: '2016-08-17T12:00:00.000+10:00'
author: Richard Banks
tags: 
modified_time: '2016-08-17T12:00:00.000+10:00'
---
If you live in Australia, I probably don't need to provide any background on the problems with the 2016 Australian Census system. Even if you live outside the country you might have heard about the issues we experienced; it certainly got enough coverage.
In case you haven't seen it, or have forgotten, the Australian Beureau of Statistics (ABS) encouraged Australians to respond to the 2016 Census online instead of using the traditional paper forms.

In order to cope with the expected user load, the ABS paid IBM just under $10 million to build the 2016 data collection system and another ~$500K to RevolutionIT to load test the system.
Unfortunately, on the night as user load increased the system crashed and crashed hard. It then remained offline for two days and over the ensuing weeks experienced ongoing problems. The political fall out and blame shifting has been impressive.

To be clear, I didn't work on the project and don't know anyone who did, so I can't be sure what actually led to things going as badly as they did, but press reports indicate there were a number of major problems

 * Engagement with a vendor (IBM) who has a reputation of not caring about value delivered, instead focusing on delivering only what was specifically asked for and no more, and extracting maximum revenue from their customer.
 * Completely unrealistic expecations of user load. Expectations that everyone else could see were way too low.
 * Untested handling of failure scenarios.
 * A belief that a non-cloud infrastructure could scale to meet spikes in load
 * Staff who could see the solution was going to fail, but either failed to call it out or were shut down by management.

Have a read through the various links at the end of the post for where this information has come from.

Of course, not long afterwards a few students put together a clone of the census site over a weekend for a few hundred dollars and handled a much higher load. They're quoted as syaing: "I was very surprised they didn’t go with cloud infrastructure but that gets into the whole security thing."


## Architecting for Scalability

So the question for me is, what would I need to think about if I was designing the Census site. What considerations would I need to make?

At its core the Census is a simple data entry collection system.
Users log in to the system, using a pre-provided reference number (effectively, a correlation id), create a password in case they need to return to a partially completed form, and then fill in their data.
The form itself has some dynamic aspects to it, but overall it is at best a low-medium complexity data entry form.
Data is saved in a database somewhere, and once the census is closed, that data will be analaysed by data scientists in varying ways.

So, for me, the big design considerations are:
 1. Scalability:
    Spikes in load are going to occur. You know that. I know that. Everyone knows that! Why the ABS and IBM thought a user load of 500K submissions in an hour was realistic is beyond everyone I've talked to.
    In any case, if you want to ensure availability during a load spike you need to autoscale. This means don't self-host! (The census site is hosted at IBMs Baulkham Hills data centre). Self hosting requires purchasing all the capacity you think you need before hand. Using the cloud means you can add capacity based on demand and scale back down once the spike is over.
    Don't build a solution based on VMs either. Prefer PaaS offerings as they better support load at scale and are much simpler to manage.
    Also, unless you're deploying a monolith, expect to scale different parts of your overall solution at different rates. 
 2. DDoS and other security attacks:
    A national census is a high profile target. We know people will try to mess with it. Have DDoS protections in place, and test them. Remember that a majority of Australians trying to log in to the system on Census night might look like a DDoS attack, and plan for it.
    Reports that IBM chose to ignore DDoS protections from their network vendor strike me as penny pinching madness.
 3. Data storage and sovereignty:
    We know a census form submission application is going to be write heavy so we should choose a data storage strategy that supports this.
    Being more specific, since people were given census ids before the night, a simple key/value storage mechanism be great. Almost all operations could be direct O(1) updates of single records, addressed using the census id. Azure table storage or AWS S3 would be fine for this purpose.
    At a later date you could easily take this data and put it into a database structure suited for the analysis needs of the data scientists without impacting the performance on the nigh.
    I don't know what storage mechanism was used for the Census itself, but the fact that the ABS purchased an extra 600 licenses for IBM Notes scares me. I'm hoping it was just for email, though anyone who inflicts Notes on people as an email client is unusually cruel.
    Oh, in terms of the data sovereignty questions, both AWS and Azure have Australian data centres, so there's no issue there. They're both also heavily certified from a data security perspective, so it's likely that your data would be safer there than in your own data centres. There really is no reason not to use them.
 4. Redundancy and fail over:
    Disaster recovery is important. The analysis by [risky.biz](http://risky.biz) on the census failure points to a lack of planning and lack of verifying their DR scenarios.
    As a counter point, Netflix practice failing systems in production all the time (using their ChaosMonkey utility) and no one notices. If I was building the census site, I'd be genuinely worried about failuree scenarios so I'd be inclined to run the system in multiple data centres and not only run it in Azure (for example) but have a secondary deployment running in AWS that I could fail over to in the event that Azure had a very unlikely but catastrophic failure.


### References

IBM:
 * [IBM's unethical behaviour](http://www.theregister.co.uk/2013/08/06/ibm_committed_ethical_transgressions_to_win_botched_project/)
 * [IBM makes meek apology for Census failre](http://www.theregister.co.uk/2016/08/11/ibm_makes_meek_apology_for_oz_censusfail_offers_no_fail_detail/)
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

Cloud based census for $500: http://www.news.com.au/technology/online/qut-students-design-a-500-cloudbased-census-server-four-times-better-than-ibms-9-million-system/news-story/0a4eeabf733cedfce0091ce6f062c60c

Purchase of 600 IBM Notes licenses https://www.tenders.gov.au/?event=public.cn.view&CNUUID=9F2F60F5-B6E4-5540-FA0002385D7EA7AF

