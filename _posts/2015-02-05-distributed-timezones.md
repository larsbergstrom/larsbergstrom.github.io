---
layout: post
title:  "Distributed teams and time zones"
date:   2015-02-05 12:00:00
categories: servo distributed
---

I've only been at Mozilla for a little bit over a year now, but I can definitely talk about the hardest part for me of managing a team ([Servo](https://github.com/servo/servo)) that is primarily distributed but whose members work closely together on nearly everything: timezones.

Most of the other pieces on various aspects of distributed work or companies that are primarily distributed cover lots of important things. e.g., [culture](http://blog.fogcreek.com/maintaining-company-culture-in-a-distributed-world-part-1/), [wordpress](http://smile.amazon.com/Year-Without-Pants-WordPress-com-Future-ebook/dp/B00DVJXI4M), [vacation policy](http://www.paperplanes.de/2014/12/10/from-open-to-minimum-vacation-policy.html), etc. But, many of them are characterized by work that doesn't require close coordination by people in disparate time zones. This close collaboration requires a very different working model than I've seen at other companies, even distributed ones, and wanted to talk about what we do on Servo and how it's even slightly different from other groups at Mozilla.

## Latency: how timezones hurt

In a typical review process for a patch, the author first opens a new [PR](https://github.com/servo/servo/pulls) on GitHub. This action causes our review system, [Critic](https://critic.hoppipolla.co.uk/tutorial), to create a new entry to track the review process of all of the changes. Reviewers are semi-automatically notified, leave their comments, and additional commits are made by the original author until all feedback has been addressed, the patch is accepted, and then the automated systems will test and merge the fix.

In a single timezone, even with a distributed team this is a fast process - comments and changes land quickly, deeper questions can be answered over [IRC](http://logbot.glob.com.au/?c=mozilla%23servo) by the various participants, and the patch can be merged quickly.

Unfortunately, across multiple timezones things get much more challenging. Even assuming just three time zones - Brisbane, Australia; the US Bay Area; and London, we have some challenges. If someone in London needs a review from a late riser in the Bay Area, since London is 8 hours ahead, that means that a 10 AM, first thing in the morning review, has comments coming in at 6 PM, at the end of any reasonable workday. That brings one additional day of lag on any round of feedback. Even worse, Brisbane is 18 hours ahead! That difference means that a request for review from the US West Coast on Friday morning at 10AM will not be seen by anybody working reasonable hours in Australia until their first thing Monday morning, which is 3 PM Sunday on the US West Coast.

## Landing big changes

Of course, this latency does not affect all changes equally. Straightforward changes and fixes to [good first bugs](https://github.com/servo/servo/issues?q=is%3Aopen+is%3Aissue+label%3AE-easy) tend to land quickly. But, that heavily penalizes all big changes, since now in addition to the feedback loop, the big changes now have to also be rebased against the fast-moving master branch. In practice, for the biggest of our Servo changes (upgrades to new version of the Rust compiler), we actually lock the tree from all changes once we think the upgrade is within a week or so of landing.

## Dealing as an IC - lots of different work items

The first way that we deal with this problem for ICs is to ensure that people work in multiple areas on multiple items. This approach is very different from my previous life at Microsoft, where we would heavily encourage people to go very deep in a single area and focus on individual items to completion and landing them as quickly as possible in rapid succession. Except for the very easiest or most isolated feature areas, that's just not possible in Servo.

## Dealing as a manager - distributed reviewing

One thing that I do as a manager is also try to spread what Mozilla calls [Peers](https://www.mozilla.org/hacking/module-ownership.html) and we on Servo just call [reviewers](https://github.com/servo/servo/wiki/Governance) across time zones. That is, we try to ensure that there are people who can review changes in at least US and European timezones for all modules of Servo. Going forward, we've been completely overloading our Australian and Indian contributors, and I would like to continue to grow reviewers in all areas in those time zones. This approach does call for a bit of possibly-uncomfortable delegation of ownership - you can't have a single person who reviews all "big" changes to a system like [layout](https://github.com/servo/servo/tree/master/components/layout) or not only are their conference and vacation schedules a bottleneck for landing large changes, but so are the time zones that they work from! Anecdotally, I've heard from contributors in other open source projects who have completely given up trying to contribute due to living in the "wrong time zone" or being blocked from contributing during long working group meetings that take the owners offline.

## Team meetings

If you want to have some kind of weekly meeting, there's no getting around the fact that you have to either pick a time that doesn't work for everybody or you have to do some sort of alternation. On Servo, we alternate between an early meeting in the US West Coast time (which works for Europe) and one that is late in US West Coast time (which works for Asia). We also take extensive [notes](https://github.com/servo/servo/wiki/Meetings) - basically, a full transcription of the meeting - so that people who are not in the time zone can catch up on what was discussed. We also often have to either table discussions or have them over the [mailing list](https://lists.mozilla.org/listinfo/dev-servo) instead, if we cannot get everybody together in one timezone. Alternatively, for very large discussions that require realtime interaction, we will save them for the two or three times a year that we get the whole team together for a workweek in a single location. I often start preparing agenda items for these meetings months in advance, which is one of the unfortunate costs associated with massively distributed development (that is, you can't just get everybody in a conference room and hash out what the new graphics stack will look like - you have to wait until the next time all of the relevant players are co-located).

## The upside: volunteers and recruiting

Of course, if you're reaching the end of this article, you may wonder if there is any benefit to working in this manner, since none of these workarounds are a complete replacement for the ideal - a bunch of colocated people working together at the same time. The benefit, of course, is that we can recruit candidates who live globally, collaborate smoothly with companies who are not headquartered in the same time zone, and can support the massive worldwide volunteer community. There are a particularly large number of amazing volunteer contributors in India and China that open source projects completely miss out having work on non-trivial contributions if the project does not have global support.

And at Mozilla and especially in the tiny Research division, where we are very limited in our budget and rely heavily on collaborating with both corporate partners and volunteers, being great at working with them no matter where they are is a huge advantage and key to Servo's long-term success.
