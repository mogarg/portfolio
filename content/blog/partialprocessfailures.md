---
title: "Understanding, Detecting and Localizing Partial Failures in Large System Software"
date: Sat, 23 May 2020 11:14:10 +0000
draft: true
featured: false
tags: [distributed systems, gray failures, cloud]
---

## Partial failures are common

- There is nothing in the logs that would show a problem, the system seems to be working fine as well.
- The process doesn't crash, but there are safety or liveness issues or slowdown in one or more functionalities of the system.

### Zookeeper case

- The `sequencer` has a lock on the `request processor` and is stuck on `writeString` operation because of a partial network failure.
- However, the `failure detector` is totally unaffected and is sending `heartbeat` messages to the followers.
- The main issues with testing for partial failures
  - The tests are too shallow e.g. sending `HTTP` requests regularly and observing response codes.
  - Not having the ability to localize failures.
  - Software partial failures are not well understood.
- Surveyed 100 partial failures of `Zookeeper`, `Cassandra`, `HDFS`, `Apache`, `Mesos`.
- Findings 1
  - Interesting that they found most of these failures in last 3 years of their 9 years in existence. This is because when software evolves it complicates the failure semantics.
- The focus is on process level failures.
- Findings 2
  - The reasons for failures is diverse
  - The top three causes are
    - uncaught exceptions
    - indefinite blocking
    - buggy error handling
- Findings 3
  - In 48% of the cases some part of the function was stuck. The process overall did not show any problems, however, certain parts of the process may be unresponsive.
  - In 17% of the cases some functionality was slow.
  - In 15% of the cases there were silent errors (or zombies). E.g. Uncleaned buffer from a certain client session was used for a different client.
    - Such failures are hard to detect without a detailed correctness specification.
- Findings 4
  - 71% of failures were due to specific conditions or input in production
    - Receiving a corrupted buffer.
    - While joining a quorum, a follower hung and another follower crashed.
  - About 68% of failures are sticky. The process would not recover from these failures by itself.
- The median diagnosis time is 6 days and 5 hours.
  - Mysterious bugs have the tendency to mislead.
  - So, developers have to turn on debug code, analyze heap or instrument code.
- It is tedious for developers to add timing and exception catching code because it brings a lot of maintenance burden.
- Checkers
  - The checkers should be tailored and customized for a specifij module. There exist Linux watchdog, httpd watchdog but they are too generic.
  - Secondly, the checkers should be stateful. They should access the latest program state. These are called checkers.
  - Third, the checkers should run concurrently with the process.
    - Decouple main execution state from the checker.
    - In this way we get isolation.
- `Mimic checking`
  - The `Mimick Checker` does the same operations as the main execution and hence can detect network being stuck.
  - The tool used was `OmegaGen` which systematically generates `mimic-type` watchdogs for systems softwares
  - The core technique used was `program reduction`.
    - But wait, we are trying to `mimic` the execution?
    - We can't put everything in the checker because then it would not be possible to localize problems.
- Program reduction
  - Operations such as converting to a string or sorting an array are well tested and it would be a waste of resources to test them again in production.
  - I/O operations, allocation, async wait, etc. on the other hand would depend on the production environment.
  - These operations are extracted based on heuristics and developers are allowed to tune these heuristics. E.g synchonized blocks in Java.
- Side affects
  - The operations could be transient or tolerable hence there is a synchronization of state.
  - A problem can be that the checker can modify the main programs state.
    - To solve this, The checker context replicates the checker state.
- Another problem can be that there is just too much copying which results in a performance overhead.
  - The checker avoids this by `immutability analysis` and `lazy copying`. It only copies the mutable state.
  - `lazy copying` can be further problematic because the state might change. Use `hash codes`.
- Use `I/O` redirection and `idempotent wrappers`.
- Results
