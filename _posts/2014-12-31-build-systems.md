---
layout: post
title:  "Tales from Build Systems"
date:   2014-12-10 17:00:00
categories: servo visualstudio microsoft
---

Inspired by the post on the [Mozilla Build System](http://glandium.org/blog/?p=3318), especially the goals for its updates, I was motivated to pass along some stories and lessons from a similar conversion that I led within Developer Division. When I first joined the Visual Studio Environment team, it took *three days* to build Visual Studio - build Visual C++, drop some binaries in special places to build Visual Studio, drop some binaries in another special place, build the .NET Framework (nee COMPLUS), drop those binaries back in a third special place, and then rebuild Visual Studio a final time. And that's assuming that there were no breaking changes, you didn't want to self-host (use a just-built either C++ or C# compiler to rebuild the C++ or C# code), etc. It was insanity, and the developers who sorted out bustage (via the "vscomluv" e-mail alias) were truly people of great patience.

Sometime around 2003, we were finishing up Visual Studio Everett and I was lucky enough to have no bugs left, so I talked with my dev manager about letting me sort out the madness, after demonstrating initial feasibility. Long story short, I got to steal two incredible developers - Dan Spalding, owner of every version of LINK.EXE ever, and Rudi (CHEETO) Martin, a truly awesome developer from the COMPLUS team - to spend about a month with me sorting out the technical bits and making sure we didn't come up with something that didn't fit the needs of their teams (VC++ and COMPLUS had unique requirements from the teams that were more focused on the IDE).

In this brief post, I'll just mention a few lessons I learned along the way.

# Centralize customization

As part of moving to a shared build system, we wanted to have a shared set of compiler flags. While digging into these flags, I came across an undocumented flag that had slipped into a corner of the Visual Studio build at some point in the past. After knocking on lots of doors, I discovered that not only did we no longer need the flag - its purpose was to disable an optimization that was behaving badly with code generation sometime in the mid-90s - but it was also causing the release build to limit itself to generating 486 output! That's the only fix we ended up backporting into the Everett release, as actually generating code for our minimum target machines was a double-digit percentage perf win on startup.

There were many very surprised and upset performance and compiler developers when we tracked that down. We probably would have noticed this all much earlier if the compiler flags had been centralized someplace that a compiler dev could see them, rather than stashed in a project-specific shared flag directory buried several levels deep in the IDE-only trees.

# Sometimes, C has those things for a reason

When .NET was first designed, people were very happy to see header files going away, to be replaced by metadata within the output binary files themselves. However, we ran into several problems. First, we had circular dependencies that were really tough to resolve - System.dll and System.Configuration.dll depend on one another (in order to avoid loading System.Xml, which System.Configuration relied on, in some minimal startup scenarios). The workaround in Everett was using the C preprocessor and macros inside of C# code, combined with building the binaries repeatedly until you reach a fixpoint with all depenedencies in place. Second, in a large project, sometimes you don't want to build *everything* and you'd like to just have a small file that you can reference.

Luckily, we had Rudi working with us and we "owned" the source tree (recall: everybody else was still finishing off the Everett release), so he whipped up an implementation of metadata-only assemblies and fixed up all of the .NET tooling to be able to process (i.e., not crash) on them. We originally intended it to be internal-only, but I'm pretty sure that it has since shipped. In retrospect, it was pretty obvious other people would need similar functionality, and it's unfortunate that it got designed/implemented in the dark corners of a build lab during a mad rush rather than during the normal product cycle, with program management & QA involvement.

# Magical tools can be too magical

One thing we looked into was increasing parallelism, so we grabbed one of the zillion tools that will "track your build's dependencies" and let it watch a few builds of the whole product to determine which portions could be built in parallel. Unfortunately, the output was unusably complex - it would attempt to find maximum parallelism, even if it meant identifying dependencies far, far across the build system (including across projects that would never be built simultaneously on a non-build-lab machine).

At the end, we got a much more understandable build by simply asking developers to mark subprojects that could obviously be built in parallel, and since at the time we were basically bound by disk I/O to no more than four simultanous writing actions anyway (this is before SSDs, so the best we could do was RAID-0 paired platters), that resulted in a much more sensible system for developers and pretty much kept the machines fully used anyway.

# The long tail of use cases is really long

One day during this project I had a dev lead from a far-flung team drop by my office and ask if we would still support incremental builds when using the binaries built the night before as a baseline, with some headers in-place updated via a sed script, or would they have to introduce some more calls to touch to post-sed to ensure the timestampes were correct? And that was just another dev team - the localizers (we localized French, German, and Japanese on-site), QA teams, performance teams, and even some "metrics" groups all had some pretty far-flung requirements before we could flip the big switch. Luckily, this was mainly a matter of dropping by the appropriate managers-of-managers meeting for each of the disciplines to brief them on what was coming so they could ask their teams to panic and come to me with additional requirements, but if I hadn't communicated pretty widely some of the groups that hide off in corners would have been very upset when we made the changeover.

# Surprises!

When you're migrating 120 million lines of code, there will be things that break. Tools that turn out to be polynomial in the number of input files; tools that can't handle more than 2^31 lines of text; tools that simply mysteriously fail. Being inside of Developer Division, I was lucky when the tools were things that we or the Windows team owned. Unfortunately, we also used a bunch of hacked-up versions of SED.EXE, PERL.EXE, etc. that had come to us via mysterious mingw or other ports and where - at least at that time - I was prohibited from looking at or rebuilding them from sources. I'm not really sure what the lesson here is other than that I got pretty lucky in many ways! The whole migration could have been either completely hamstrung or required that I spend weeks rewriting all of our sed and perl scripts as javascript...
