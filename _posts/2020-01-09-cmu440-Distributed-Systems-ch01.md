---
layout: post
title: cmu440-Distributed-Systems ch01 introduction
categories: Distributed Systems
description: Distributed Systems
keywords: Distributed
---



## definitions

```
A distributed system is a collection of autonomous computing elements that appears to its users as a single coherent system.
```

- a collection of computing elements each being able to behave independently of each other
- users believe they are dealing with a single system.


### overlay network
- Structured overlay
- Unstructured overlay
  - P2P networks

### Middleware and distributed systems
- Facilities for interapplication communication. 
- Security services.
- Accounting services.
- Masking of and recovery from failures.

#### middleware services
- Communication
  - Remote Procedure Call (RPC)
- Transactions
  - atomic transaction
- Service composition
- Reliability


## Design goals
### Supporting resource sharing
- google drive
- dropbox
- BitTorrent: center tracker, DHT
- IPFS: InterPlanetary File System

### Making distribution transparent

#### Transparency
- Access
- Location
  - URL
  - CDN
- Relocation ？
  - have been moved from one data center to another
- Migration ？
  - being
- Replication
- Concurrency
- Failure

#### Degree of distribution transparency
- newspaper
- LBS service
- trade-off CAP

### Being open
- interface
- IDL
- portablity
- extensible
  - raid: inextensible 
  - nfs: extensible

### Being scalable
- Scalability dimensions
  - Size scalability
  - Geographical scalability
  - Administrative scalability

### Analyzing service capacity

![ch01-note1.6](/images/posts/distributed-system/ch01-note1.6.png)

- the arrival rate of requests is λ requests per second
- the processing capacity of the service is μ requests per second
- the fraction of time pk that there are k requests in the system is equal to:

$$
p_k = (1 - \frac{λ}{μ})(\frac{λ}{μ})^k
$$

- $\frac{λ}{μ}$ 表示 1秒内 process 占用的时间，1s内进来 λ，每个请求 process 时间为 $\frac{1}{μ}$ ，所以 process 占用的时间为 $\frac{λ}{μ}$，也就是 
- 查看极限情况 $p_0$ 表示1秒中，system 是空闲的，所有任务都处理完了  
$p_0 = 1 - \frac{λ}{μ}$

- $\frac{λ}{μ}$ 表示 1秒内，

- 查看极限情况 $p_\infty$ 表示


- Hiding communication latencies
  - asynchronous communication
  - check form in web client using js
- Partitioning and distribution
  - DNS
  
### Pitfalls
• The network is reliable
• The network is secure
• The network is homogeneous • The topology does not change • Latency is zero
• Bandwidth is infinite
• Transport cost is zero
• There is one administrator