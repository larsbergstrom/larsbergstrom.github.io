---
layout: post
title:  "Transitioning Servo to Travis CI"
date:   2014-06-11 17:00:00
categories: servo travis
---

[Servo](http://github.com/mozilla/servo) is a new web browser engine being built at Mozilla Research, focused on memory safety and taking advantage of modern mobile and parallel hardware. We use the new systems programming language, [Rust](http://www.rust-lang.org/), to help us achieve these goals. But, using a self-hosting compiler under very active development brings its own issues. On my powerful laptop, a parallelized from-scratch Servo build takes around 44 minutes, 40 minutes of which are totally unrelated to Servo - building the [LLVM](http://llvm.org) compiler infrastructure, building Rust a first time and then building Rust a second time with the most recent version of Rust. For contributors without a $3500 MacBook Pro, it can take several times as long, and even with one, the machine is totally unresponsive while performing that build.

This post is about what we did to remove the part of our build that compiles the Rust compiler and our longer-term plans for building up our continuous build and release engineering infrastructure. 

*TLDR*: we’ve replaced building Rust locally in favor of binaries built by [Travis CI](http://www.travis-ci.org/) and shaved off 91% of our build time.


Previously, we have used Mozilla-internal infrastructure (maintainable by some members of paid staff, but usable by all contributors) that performs the following functions:

- Build Rust and Servo for both OSX and Linux (caching Rust compiler builds between runs)
- Runs unit tests on Linux and unit tests, basic DOM tests, and ref tests on OSX
- Integrates with GitHub and bors for automatic patch approval and merging

We’re moving all of this to [Travis CI](http://www.travis-ci.org/) both because we would like to support open infrastructure and it enables more contributors than just those on Mozilla staff. Additionally, Travis CI isn’t hidden behind a firewall without external access, sharing infrastructure with mission-critical projects. So, now we can push changes to Servo’s build process without worrying about breaking something in the server farm and jeopardizing FireFoxOS builds.

Today, we’re rolling out the first part - we’ve migrated building of the Rust compiler to Travis CI and are using their automatic caching on S3. Some key features we’re relying on are:

- Targeting [multiple OSes](http://docs.travis-ci.com/user/multi-os/).
- S3 [deployment](http://docs.travis-ci.com/user/deployment/s3/). Note that this allows any repository owner to “push” a new Rust compiler build to our S3 buckets, but the private S3 key remains secret only to me.
- The fantastic Travis CI support staff, who reply at all hours of the day - key because the Servo contributors working on this are in India and Chicago, IL, USA.

The new steps for updating a Rust compiler build used in Servo are [here](https://github.com/mozilla/servo/wiki/Updating-the-Rust-compiler-used-by-Servo).

What’s next?

- Extending the Rust prebuilt compilers to also create Android cross-targeting builds
- Getting Servo itself running and testing on Travis CI (this is [already working](https://travis-ci.org/mozilla/servo), but we just have to land it!)
- Change our automation, [bors](https://github.com/brson/bors) to fire off Travis CI verification build and test runs on a reviewed PR instead of internal buildbot builds

And what will we do later that we can’t do on today’s internal infrastructure?

- Creating and deploying prebuilt Servo binaries for multiple platforms that people can try out
- Android building and testing
- Basic DOM tests on Linux (related to font licensing issues… don’t ask)

Build and continuous integration systems are often not considered exciting, but they nip away at a shocking amount of developer time both here at Mozilla Research and previously when I worked at Microsoft. Leveraging awesome systems like Travis CI helps reduce that investment, since they’re handling all of the “hard bits."
