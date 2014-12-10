---
layout: post
title:  "Adventures in running an incubation project"
date:   2014-12-10 17:00:00
categories: servo nautilus microsoft
---

I'm currently managing [Servo](https://github.com/servo/servo), which is looking to make widespread changes to Firefox's engine, Gecko, which is around 8 million lines of code. That might seem like a lot, but I've been to this party before - in 2003 I co-founded and later managed the engineering organization of Nautilus, which was a team inside of Microsoft's Developer Division (DevDiv) looking to make changes similar in scope to Visual Studio's 120 million lines of code. Some parts of that effort ended relatively well - a new text editor, which also shipped in Expression Interactive Designer (nee Sparkle); extensibility code that went on to be the Microsoft Extensibility Framework; new debugger feature prototypes, most of which have now shipped; the process experiments we ran (especially related to tradeoffs in code coverage) had met widespread internal adoption by the time I left. Many others did not - the faster metadata reader/writer, written in managed code, never made it out of the Common Compiler Infrastructure codebase; our efforts to do better cross-language data modeling integration never bore fruit; most disappointingly, we had a full web-based IDE with syntax highlighting, build, etc. using AJAX and no plugins in 2003 - a year before GMail was released! - that never saw light of day.

In this brief post, I'm not going to go into various technical details or do too much name-dropping, though I did have the pleasure of working with an amazingly talented group of people that we assembled from not only architects in DevDiv, but also recruited from external companies. Instead, I'd like to talk about some of the lessons that I personally learned leading and managing this project for several years.

# Always be relevant

Your parent organization - who is paying your bills - is probably in the throes of some deep technical problems, trying to ship under ridiculous pressures, and feels like they could really use some of those Top Engineers you have working for you. So, you need to make sure that you have a bridge for them that isn't just "throw it all out, we've bring a couple million lines over, and you can rebuild your 120 million lines of code on top of that, k?"

We tried to be good in Nautilus about always having demos that were relevant to products of today and avoiding too much talk of meta-meta-models for programming languages. We were bad at talking about and planning the transition from incubation to production, which would have been quite easy and would have made our group substantially more compelling to the partners who reported to our VP, which is ultimately whose support we needed (and failed to get).

# Build peer manager relationships

It's less true at a place like Mozilla, where the steps for promotion aren't quite as direct as, "if your organization is of size N, you will reach pay grade X" as it was at Microsoft, but there's still a very real risk if it looks like you are on a team that's staffing up people who have a clear overlap with the area of ownership of a manager on another team. In addition to that manager's career advancement, there's also a deeper question: are you trying to take their job?

This question is mainly about the transition plan, and frankly does depend on what your plans are. In my current position, I don't have long-term management aspirations (I'm a researcher) and would prefer that people who own those groups continue to own them, and that any massive build-out that happens does so in their teams. If you do have management aspirations, I'd encourage you to promote the idea that there will be more opportunities for even higher advancement in the Brave New World, such as the position of manager of both their current team and the one that you're building.

# Try to mitigate power/control struggles early

During the course of Nautilus, we had two leaders fight for control among themselves and with upper management and both ultimately left Microsoft. The one who took over next was the appropriate compensation grade for the position, but I never got the sense they were very interested in it, as they largely had me run the group and also ultimately left.

The lesson for me is that it's easy for a project - particularly an incubation one - to have infinite possibilities and attracts people who each have their own vision of where it's going. Once you start executing with fixed people, timelines, and organizational constraints, though, reality sets in and the project they signed on for may not be the project they've ended up with. (Of course, another possibility is that I'm a terror for the poor people stuck managing me!)

# Understand that you'll be the target of executive ideas

When you're an incubation unit, people assume you have infinite time to try out their various ideas. While you can ignore most of them, it's hard to ignore executive-level drive-by leadership. And I'd argue that you shouldn't even try.

Many people on Nautilus got frustrated by these distractions, and often these experiments didn't pay off very well, but they did help ensure that we continued to exist until our next executive review and got the headcount we were requesting. And ultimately, that's a key aspect of leading an incubation effort - keeping it alive long enough to merge back and deliver the changes back into the parent organization. Which is a story for another day...
