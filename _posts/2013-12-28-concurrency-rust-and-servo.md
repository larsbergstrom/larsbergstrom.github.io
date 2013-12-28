---
layout: post
title:  "Concurrency Models, Rust, and Servo"
date:   2013-12-21 11:42:40
categories: concurrency rust servo
---

This post explores the space of implementation choices in concurrency models for channel-based concurrent programming languages. It also discusses Rust's current choices in that space and their fit with the Servo parallel web browser. This post was prompted by the back-and-forth discussions on the [rust-dev mailing list](https://mail.mozilla.org/pipermail/rust-dev/ ).

## Background
A concurrent program's execution is the result of interleaving multiple sequential computations. Concurrency is distinct from parallelism, which is the simultaneous execution of parts of a single computation without changing the program's results (hopefully), though modern concurrent systems often exploit parallelism to support interleaving computations.

Broadly, there are two concurrency models used in message-passing programming languages: the <a href="http://en.wikipedia.org/wiki/Actor_model">actor model</a>, which we will not discuss here because it does not apply to Rust or Servo, and what I will for convenience here call the <a href="http://en.wikipedia.org/wiki/Communicating_Sequential_Processes">channel model</a>, though CSP or "Communicating Sequential Processes" model is more historically accurate.

### Channel model

In the channel model, the communication mechanism is the _channel_, which is typically a strongly-typed means of sharing information between concurrent computations. The strong types restrict the values that may be sent and received over the channel. Languages differ on whether they are uni-directional or bi-direction (that is, can a thread both send and receive from a channel or only perform one or the other operation).

The channel-based, or CSP, model was first implemented in occam. Variants of this model are exposed in Concurrent ML and available as a library in nearly every high-level programming language.

### Blocking modes

The biggest user-visible implementation difference between implementations of channels is whether or not the send method is asynchronous. In nearly all languages that have been implemented using explicit channels, the receive method blocks. There are a couple of exceptions where receive is defined to take a callback that will be invoked, but that is not a commonly used model.

### Asynchronous send

Asynchronous send methods are high performance for the common case, where there is sufficient buffer room to place the message. A correct implementation only needs to push the message onto the queue, which can be as simple as a fixed-sized ring buffer.

* Challenge 1: full buffer

In practice, most implementations add additional support for either infinite buffers or blocking on a full buffer or raising an exception on a full buffer. Each of these have significant implementation burden for correctness, but do not affect the good performance of the common case.

* Challenge 2: fairness (and chatty senders)

A common pattern in concurrent languages is to have "producer" and "consumer" computations. With an asynchronous send, rather than a direct send/receive pattern, either the program needs to have a pair of channels so that the consumer can send an acknowledgment message or the runtime has to perform ad-hoc scheduling tweaks to attempt to prevent the producer from sending an unbounded number of messages and running into the edge case in Challenge 1.

* Challenge 3: debugging

Incorrect protocol design in an asynchronous model results in memory exhaustion. If you are lucky, that exhaustion will show up in the queue where messages are being logged. Unfortunately, in practice such errors cause programs to crash inexplicably in "can't fail" allocations such as a small tuple on the heap and the reported bugs will be written off as unreproducible errors or transient faults.

### Synchronous send

The biggest advantage of synchronous send is that at the same time the receiver begins processing the message, the sender knows that the message has been delivered, not just enqueued. This knowledge is actually stronger than an acknowledgment because it provides two-way common knowledge, cutting short the infinite tower of "acknowledging the acknowledgments."

A secondary advantage is that debugging of broken protocols is straightforward - it results in a deadlock, and as long as your system has stack backtraces (many concurrent language implementations do not, by default!), those will point you at the threads that have hung.

* Challenge 1: performance

In a system with a synchronous send, if there was potential work that only the producer could have performed between the time when they initiated the call to send and when the message was received, that time is wasted unless the producer spawns an extra computation.

## Implementations

As of today, you can see Rust's design in `std::comm`, in the file [mod.rs](https://github.com/mozilla/rust/blob/master/src/libstd/comm/mod.rs#L580-L609) and the function `Chan::try`. Rust has an asynchronous `send` with infinite message queues and uses a gated backoff to force a yield to avoid flooding the channel. On an interesting note, by default Rust checks to see if there is a pending a receiver and, if so, performs a context switch to them.

### Servo usage breakdown

[Servo](http://github.com/mozilla/servo), the web layout engine that is the primary consumer of Rust, currently has around 125 non-test-code calls to Rust's asynchronous `send` function and one call to `send_deferred`, which is a version of the `send` function that avoids the context switch to the receiver. This call occurs within the profiler code at the point where profiling events are logged, as performing the context switch negatively impacted overall system performance by thrashing back and forth to flush the log events on every loop of a small operation.

Interestingly, several of those calls are acknowledgment calls. The Render task needs to let the pipeline know when it has finished shutting down and releasing all of the graphics resources it owned. The Layout task does a similar thing. There are more we will probably end up adding - Chrome uses an ACK protocol when their version of the compositing task has started processing a resize event to avoid having the process generating those events flood the system, which we currently suffer from. And the script task also shares memory with tasks such as layout that may need to handshake their shutdown protocol for safety.

I am not strongly arguing in favor of synchronous message sending. There are some patterns in servo that are much easier with asynchronous sending. Beyond the profiler example earlier, we would also like for our resources parsers to be able to deliver incremental results without blocking the process of parsing. But, since we are already performing a context switch in >99% of the `send` calls today, it's hard to believe that we're taking advantage of the #1 advantage we currently have - doing other work in the original computation while we wait for another computation to receive the message.
