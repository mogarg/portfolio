---
  title: Elpis 
  date: 2018-09-11T12:41:05-05:00
  description: A Multi-leader Cross Fault-Tolerance Algorithm 
  image: /img/elpis.png
  featured: true
  tags: [Byzantine Fault-Tolerance, Blockchains, DHT, Java]
---

## Thesis

My thesis can be found [here](http://hdl.handle.net/10919/85049).

## Motivation

While internet services are designed and deployed with an assumption of perimeter less security at the application layer, most production distributed systems today are designed assuming a classical Crash Fault Tolerant (CFT) model. In realistic deployment scenarios, keeping track of all bridged networks such as smartphones, network printers, etc. is almost impossible. Hence, designing distributed systems without considering the possibility of an active adversary makes the system vulnerable from its core. While there exists extensive research on Byzantine Fault Tolerant (BFT) systems, overheads associated with such solutions preclude widespread adoption.

## Problems

Cross Fault Tolerance (XFT) by Liu et al. addresses this problem by providing stronger consistency and availability guarantees than both the CFT and BFT under the assumption that a majority of replicas are correct and can communicate with each other synchronously. XPaxos designed by Liu et al. assuming the XFT model achieves similar throughput and latency as Paxos. However, it brings two challenges. One it fails to provide comparable performance as the number of faults are higher than one. Secondly, since it is reliant on a single leader for ordering commands, it suffers from similar bottlenecks as Multi-Paxos, the widely adopted and deployed version of Paxos. Designing a multi-leader consensus algorithm like M2Paxos can solve the second problem. However, M2Paxos does not guarantee liveness under contention, and the Byzantine nodes in the system can further exacerbate this problem by increasing the conflicting commands.

## Solution

To solve the problem of liveness in M2Paxos, I designed a leader-election algorithm which provides liveness while providing weak-consistency for object ownership. The weak consistency assumption implies that even though all nodes don't agree on a single leader for objects accessed by conflicting commands at all times, they can have these commands decided by forwarding their request to a node that they have elected as the leader. Ultimately, if there are no new conflicting commands proposed all nodes agree on a unique leader. The ownership acquisition for commands don't conflict can still execute in parallel. To make this Byzantine Fault-Tolerant I plan to use Verifiable Random Functions where nodes generate random tags which can be verified by other nodes. This algorithm can be leveraged to design a multi-leader Byzantine Fault Tolerant consensus algorithm which provides higher performance than XPaxos and which does not incur extra deployment costs.