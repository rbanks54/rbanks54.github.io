---
layout: post
title: Lessons from the Australian Census
date: '2016-08-17T12:00:00.000+10:00'
author: Richard Banks
tags: 
modified_time: '2016-08-17T12:00:00.000+10:00'
---
If you live in Australia, I probably don't need to provide any background on the recent problems with the 2016 Australian Census, and those of you living outside the country might have heard about the issues.
In case you haven't, this year the Australian Beureau of Statistics (ABS) encouraged Australians to respond to the Census using an online system instead of paper forms.
This was intended to save the government around $100 million in costs. The damaged trust with the Australian public, the PR nightmare, the potential data quality problems in the results, and more are going to cost a lot more than that. 

In order to cope with the increased user load the ABS engaged IBM, who built the 2016 data collection system for just under $10 million using their SoftLayer psuedo-cloud; supposedly in such a way as to handle the load. The ABS also engaged RevolutionIT to load test the system.
Results from the load test and from IBM led the department into a false sense of confidence that the system was solid and stable  and would handle the expected load on the night.
Unfortunately, on Census night itself, at the time when most Australians were finishing up dinner and logging on, the system crashed and crashed hard. It remained offline for two days and in the ensuing weeks experienced ongoing problems. The political fall out is impressive.

To be clear, I didn't work on the project and don't know anyone who did, so I can't be sure what actually led to this farcical outcome, but press reports show there were a number of major problems

 * Engagement with a vendor (IBM) who has an reputation of caring less about value delivered than the revenue they can extract from a customer.
 * Completely unrealistic expecations of user behaviour. Behaviour that almost everyone else could see were wrong.
 * Untested handling of failure scenarios.
 * A belief that a non-cloud infrastructure could scale to meet spikes in load (IBM's SoftLayer)
 * Staff who could see the solution was going to fail, but either failed to call it out or were shut down by management.

[[Add links]]

Of course, not long afterwards a few students put together a site over a weekend using AWS that mimicked the census functionality and was able to build it for a few hundred dollars and handle a much higher load.
They're quoted as syaing: "I was very surprised they didn’t go with cloud infrastructure but that gets into the whole security thing."


## Architecting for Scalability

At its core the Census is a simple data entry collection system.
Users log in to the system, using a pre-provided reference number (effectively, a correlation id), create a password and then fill in their data.
The form itself has some dynamic aspects to it, but is generally pretty simple.
Data is recorded in a database, and once the census is closed, that data will be analaysed by data scientists in varying ways.

If you were building a system like this there's a few things to consider:
 1. Scalability:
    Spikes in load are going to occur. You know that. I know that. Everyone knows that! If you want to ensure availability during a load spike, don't self-host! (The census is hosted at IBMs Baulkham Hills data centre).
    Don't build a solution based on VMs either. Look to PaaS offerings to better support higher flexibility of scale, simpler management and support for features such as autoscaling.
    Expect to scale different parts of the overall solution at different times. 
 2. DDoS - you're going to run a high profile service? Hell, you're going to run a service? Expect people to mess with you. Have DDoS protections in place.
    Reports that IBM chose to ignore DDoS protections from their network vendor strike me as penny pinching madness.
 3. Data storage - a form submission application is going to be write heavy. Choose a data storage strategy that supports this.
    Being more specific, given people were given census ids before the night, a simple key/value storage mechanism would more than suffice. Azure table storage or AWS S3 would be fine. You could also use one of the many NoSQL key/value databases for this if you prefer.
    I don't know what storage mechanism was used for the Census, but the fact that the ABS purchased an extra 600 licenses for IBM Notes scares me. I'm hoping it was just for email, though anyone who inflicts Notes on people as an email client is unusually cruel.
 4. Redundancy and fail over - disaster recovery is important. The analysis by risky.biz on the failure points to a lack of planning and lack of practice in DR scenarios.
    Netflix practice failure in production all the time (using their ChaosMonkey utility) and if you were genuinely worried about it, you could do what many banks are looking at doing these days which is
    splitting solutions so that they are geo-redundant (run in multiple cloud data centres) and provider redundant (run in both AWS and Azure).


### References

IBM behaviours
http://www.theregister.co.uk/2013/08/06/ibm_committed_ethical_transgressions_to_win_botched_project/


News articles about the Census failure:
http://www.abc.net.au/news/2016-08-09/abs-website-inaccessible-on-census-night/7711652
http://www.abc.net.au/news/2016-08-14/abs-working-to-rebuild-trust-after-census-failure-chief-says/7733234
http://www.news.com.au/technology/online/ibm-servers-for-australian-census-2016-miscalculated-level-of-demand-in-a-short-space-of-time/news-story/63dee596f1b541446b83afe20d0584e3
http://www.news.com.au/technology/aussies-face-more-frustration-as-census-servers-continue-to-fail-despite-repeated-attempts/news-story/4e43f5e92dda5685778601b9b66fddf9
http://www.itnews.com.au/blogentry/the-real-security-threat-of-the-2016-census-433907
http://www.theregister.co.uk/2016/08/11/ibm_makes_meek_apology_for_oz_censusfail_offers_no_fail_detail/
http://www.arnnet.com.au/article/605089/ibm-reputation-an-all-time-aussie-low/ 
http://www.itnews.com.au/news/experts-cast-doubt-on-abs-census-dos-claims-433926
http://www.itnews.com.au/news/govt-could-chase-ibm-for-damages-over-census-failure-433519
http://www.lifehacker.com.au/2016/08/ibm-and-the-abs-census-let-the-blame-games-begin/

Twitter #censusFail stream - https://twitter.com/search?q=%23censusFail&src=typd 

Census failure analysis - http://risky.biz/censusfailupdate

Cloud based census for $500: http://www.news.com.au/technology/online/qut-students-design-a-500-cloudbased-census-server-four-times-better-than-ibms-9-million-system/news-story/0a4eeabf733cedfce0091ce6f062c60c

Purchase of 600 IBM Notes licenses https://www.tenders.gov.au/?event=public.cn.view&CNUUID=9F2F60F5-B6E4-5540-FA0002385D7EA7AF

