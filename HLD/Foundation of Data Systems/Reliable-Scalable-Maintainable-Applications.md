# Challenges today (CPU vs Data Management)
* Earlier CPU was of huge importance because beyond certain point it wasn't possible to scale due to limitation of vertical scaling.
  * But today with distributed system and horizontal scaling that problem is solved we are no more bounded to CPU
* Real problem is with data.
  *  **Data distribution**: where is the data, and how fast can you get it?
  *  **Network latency and throughput**: moving massive amounts of data takes time.
  *  **Storage bottlenecks**: reading/writing at scale is tricky — databases, disks, object stores have limits.
  *  **Data consistency**: keeping distributed copies in sync is hard (CAP theorem problems).
  *  **Complex data access patterns**: joins, aggregations, searches across large datasets.
### Example in layman terms
  * Imagine you run a restaurant:
    - Old days: one superfast chef making all the dishes
    - Now: Many decent chefs (distributed) working in parallel, but you need good kitchen organization (data systems) so ingredients (data) arrive on time, orders are clear, and dishes don’t pile up at the pass.
    - Even if your chefs are fast, if your supply chain and order management suck, you’re in trouble.

# Let's Study about data intensive application

## Data Storage Architecture
[![Data Storage Architecture](Images/Data%20Storage%20Architecture.png)](Images/Data%20Storage%20Architecture.png)

## Concerns of distributed System
[![Concerns of distributed system](Images/Concerns%20of%20distributed%20system.png)](Images/Concerns%20of%20distributed%20system.png)

### 1. Reliability
* Perform expected function. 
* Tolerate user mistake. 
* Performance is good as per use case. 
* Unauthorised access is denied.

#### Faults vs Failures
* Fault: Deviating from spec.
* Failure: Entire system stopped working
* We can try to prevent faults but can't avoid entirely.
* We should have mechanism of fault tolerance that it doesn't lead to failure

##### To test the system there are ways to induce faults deliberately to test system.
TODO: Read about [Netflix chaos monkey](../Appendix/Netflix-Chaos-Monkey.md)
* We generally prefer fault tolerance but at times cure doesn't exist. 
  * Eg if someone got unauthorised access to sensitive info.
  
##### Hardware faults

###### **Common failures and resolutions**

| Failure            | What happens                                      | Impact                                                                 | How to handle                                                                                                                                                                |
|:-------------------|:--------------------------------------------------|:-----------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Disk crash         | physical wear out <br/> sectors become unreadable | Loss of data <br/>                                                     | Replication<br/> Health check                                                                                                                                                |
| Faulty RAM         | Physical damage, cosmic rays flip random bits     | Data corrupt<br/> 2. Application crash                                 | ECC(error correcting code) can correct single bit<br/> Checksum or hashing by comparing data stored in memory and disk<br/>Container based application doesn't impact others |
| Power grid failure | Natural or physical disaster                      | Data loss<br/>Shutdown<br/>[Split brain](../Appendix/Split%20brain.md) | Battery<br/>Geo Redundancy<br/>Distributed Consensus algorithm                                                                                                               |

* **RAID(Redundant Array of Independent Disks)** is a way to combine multiple hard drives to either: 
  * Keep your data safe (by making copies)
  * Make things faster (by splitting the data across disk)

* Split Brain: When system not able to communicate due to partition below algorithm comes in play refer appendix for details of algorithm.

###### **Comparison Table**

| Category                | **Leader-Based (Raft)**                                                                      | **Quorum-Based (Zab, Paxos)**                                                               |
|:------------------------|:---------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------|
| **Leader Presence**     | ✅ Only **one leader** at a time who makes decisions                                          | ✅ Zab: has a **leader (coordinator)**<br>❌ Paxos: no fixed leader by design                 |
| **Partition Handling**  | ✅ New leader elected only with **majority (quorum)**<br>❌ No leader if no quorum             | ✅ Majority (quorum) still needed for decision making                                        |
| **Log Handling**        | ✅ **Leader maintains logs**, followers replicate them for consistency                        | ✅ Zab: **actions done in order, broadcast from leader**<br>❌ Paxos: no logs, just proposals |
| **Roles**               | 👑 **Leader (manager)**<br>📓 **Followers (replicate logs)**<br>🎲 **Candidate (electable)** | ✅ Zab: **Leader + Followers**<br>✅ Paxos: **Proposers + Acceptors + Learners**              |
| **Failure Detection**   | ✅ **Heartbeat and timeout mechanism** — followers elect new leader if missed                 | ✅ Zab: Similar leader health checks<br>✅ Paxos: Proposals retried if no consensus           |
| **Conflict Prevention** | ✅ No leader elected unless **majority reachable** — prevents split-brain                     | ✅ Same quorum principle prevents split-brain                                                |
| **Action Ordering**     | ✅ Logs ensure **ordered, replayable actions**                                                | ✅ Zab: Leader enforces order<br>❌ Paxos: Orderless proposals, agreed on later               |
| **Complexity**          | 🟢 **Easier to understand and implement (Raft)**                                             | 🟠 Zab: Purpose-built for Zookeeper<br>🔴 Paxos: Abstract, tricky to implement              |

Major resolutions are Redundancy, Replication, Monitoring, Failover(switching to back up) and Graceful degradation

* Apart from hardware redundancy we focus on multiple virtual machines. As a result rate of hardware and software failure both have increased with increase in data.
  * But now difference is if a machine stop working or a server crashes it won't impact much because other will take over.
  * In case we need to do maintenance we can do rolling upgrade i.e. upgrade(vulnerabilities fix any cluster upgrade) one server at a time. Thus, it will reduce downtime.


##### Software errors

* We generally consider hardware errors are independent and unrelated from one machine to another. 
  * There maybe correlation sometime like temperature in the server rack.

###### Few eg of software error
* Usually software or system errors are same across instances. Some faulty inputs that cause a bug across all the nodes.
  * [Leap second addition issue](../Appendix/Leap%20second%20addition.md)
* A runway process when we are using some shared resource a shared cluster (CPU, memory ). Thus, it will impact other services as well.
* Downstream become slow due to which our system(e.g. elastic search latency or high cpu usage issue in TIS).
* Cascading impact failure in one trigger other (because of high tps of one endpoint it starts impacting other endpoint leading to increase memory or cpu usage, or due to expensive query others are not being served in ES ).

This bugs usually occur when certain thing that was assumed won't happen happens . We can test in isolation, induce faults , monitor and alerting.

##### Human errors

Usually humans who build the system are cause that issue happens.

###### Few measures to prevent human error

* Design proper apis and UI so that it's easy to use and not much restrictive (like workday UI which is so bad). 
  * UI and APIs should prompt the user to do right thing and not the bad thing.
* Provide the developers with environment so that they can test, and it's not production.
* Unit test, Integration Tests and Automated testing.
* First roll back & roll out new changes.
* Monitoring metrics and alerts.

Reliability is really important it can cause losses and damage to reputation. Even in noncritical application like photo storge application.
Let's say if a parent store child picture in your photo system and if data is lost how would they feel. In case your revenue model is based on that then. 
At times, we sacrifice reliability in prototype and low budget app but, we should take conscious decision.

### 2. Scalability

* It's easy to scale from 0 -> 1000 but how to handle when user base increase from 1000 -> millions concurrent users.
  * How to cope with growth and handle performance?
  * How can we add computing resource to handle load.

##### Load

* Load can be defined as what is the bottleneck for the system when the scale come into picture.
  * Request per sec for web.
  * Read to write ratio for database.
  * Hit ratio for cache.
  * Number of concurrent active user in live chat.

At first, we take care of average load and then handle bottleneck.

* **Understand Load by Twitter Example:**

  * A user **post tweet** to follower avg RPS(4.6K/sec) and peak (12K/sec). 
  * **Twitter feed** : A user can view tweets posted by the one they are following (300K/sec)
  * Handling 12K/sec writes is no big deal. Scale here is while reading.
    * Issue is **fan-out**(no of request we need to make tweets available to all the user, and fetch tweets from all users) since each user follow many user and followed by many user.

**There are 2 ways:**

1. Post the tweets in global storage. Now when user wants to read timeline it performs query as per below [![diagram](Images/Twitter%20DB%20Schema.png)](Images/Twitter%20DB%20Schema.png).

2. We can maintain cache for each user and as soon as tweet come by any of their follower it's send to that cache. In this case it will be cheaper to create feed because it's precomputed . But writes are expensive since they are less we are okay.

* Now let's say each follower on an avg 75 followers and tweets are 4.6k/sec so 345K to cache for timeline.
  * But follower can vary as high as 30 million in case of celebrities in that case for a single tweet we will have to do 30 million writes.

To tackle above issue we have to follow hybrid approach in case of celebrities it will be on demand i.e. when user read feed rather than fanout.

##### Performance

###### **Two ways to look at it**
*  Keeping resources same what happens to performance when we increase load.
*  Also with increased load how much resource should be increased so that if performance should be same.

* For e.g. In batch processing system it's usually number of records processed or total time to complete job.
  * In online system it's response time (time b/w a client sending request and receiving response).

**NOTE:** _Latency and response time_ are different. **Response time** is what client sees besides the actual time to process request(service time). It includes network delays & queuing delays. **Latency** is duration request is latent(awaiting service).

* Even if we hit same request again and again response time will be different. It's not a single value but distribution of value.
* Most request are fast but there can be outliers. It may be huge data. But also when data and everything is constant there can be delays.
  * These change could be due to **context switching, loss of network packet(due to congestion, hardware & interference) and TCP retransmission, [garbage collection pause](../Appendix/Garbage%20collector%20pause.md), a page fault** forcing a read from disk, mechanical vibration in server rack etc.
    * **Page Faults (Disk Reads)**: A page fault occurs when a process requires memory not in RAM but in swap space (disk). Since disk access is much slower than RAM, this introduces unpredictable latency, especially under memory pressure or if the memory page hasn't been accessed recently.

* While measuring **response time** taking average is deceiving, and it doesn't tell how many users experienced delays.
  * That's why we use percentile.
  * Let's say 100 request are hit, and we take median sorted from highest to lowest i.e. p50 which is 100 ms that means half of the request response time is less than 100 ms.
  * Generally we take p99 let's say 1 sec to measure response time that means 99 request have a response time less than 1 sec
* We generally take p99.99th percentile (1 out of 10, 000) is slowest. It's hard to reduce the response time of slowest one because data is huge, and it ought to be expensive.
* If we take e.g. of ecommerce site and if expensive query take large time that is imp customer with many orders so they are valuable we should be improving.
* But generally 1 request can go bad due to external issues.


* **SLA** : Service level agreement with customers to deliver uptime as 99.9%.
* **SLO** : Service level objective to self to deliver uptime as 99.95%. If uptime is 99.92% no breach, but we should reflect.


* Generally it's queueing delay which is responsible. Because a server can process a small number of requests in parallel(limited by resources).
  * At times, it only takes few slow request - **head-line blocking.** Even if further request are fast client will wait for completing prev slow request and thus ^ in response time.
  * That's why response time should be checked at client side.
* When we generate artificial load we should not keep the queue size small. This will lead to skewed measurement.
* **Tail latency amplification** -> In case single end user request require multiple calls to other backend service although there will be very less slow request but end user request get delayed.


###### **Coping with load**

* There are 2 ways to it **vertical scaling** vs horizontal scaling. HS is  better VS is expensive.
* There are machine which need manual intervention for scaling vs which can automatically scale. First one are better.
* Distributing stateless service is easy across machine. But stateful are hard to handle. So we can keep database on single node unless scaling cost is very high on single machine.
* There is no such customise way to scale(magic scaling sauce).
  * Problem can be volume of read, volume of write, complexity of data, response time, access patterns.
  * For example, a system that handles 6*10^7 requests per min, each 1 kB in size,  vs system that handles 3 requests per minute, each 2 GB in size—even though the two systems have the same data through‐put 6 GB.


**NOTE** : Scaling depends upon which operation will be common which will be rare, the load parameters.


### 3. Maintainability

Make a system maintainable it helps in present and in the future using 3 principle.

1. Operability: Make it easy to fix , monitor using logging, dashboards, alerts, notification and other observability. Fast deployment and scaling.
   1. Responsibility
      1. Monitor health, quickly restoring. 
      2. Fixing vulnerability, security patches.
      3. Tab on how one system can impact other to prevent any future issue.
      4. Good practices for deployment (rolling deployment).
      5. Documentation
   2. Good to have
      1. Support for automating and integration with standard tools
      2. Avoiding any dependency on 1 machine so that downtime shouldn't impact
      3. Auto resolve, but also manual control.
      4. Minimize surprise.
2. Simplicity : remove complexity which are mentioned below to allow others to understand and to reduce chance of bug.
   1. Explosion of state space: when system have multiple component , variable also multiple possible states.
   2. Tight coupling, remove dependency
   3. Inconsistent naming and terminology. 
   4. Remove accidental complexity which is not due to functionality but due to implementation. Hide implementation details behind abstraction. Make a clean facade which can be reused. 
      1. E.g. creating a different connector/client class for calling downstream and then reusing it rather than dumping everything in one class. 
         1. High level programming language hide complexity of machine code registers , cpu
**NOTE:** We also have facade as design pattern.
3. Evolvability: Make it easy for future feature development.
   1. We can use TDD and proper refactoring to add new feature with ease.
   2. For eg under load we studied two approach of news feed how to refactor to 2 from 1.


In this we studied about NFR(non functional requirement). There are also functional requirements(what is to be done - data storage then accessing , searching and processing).