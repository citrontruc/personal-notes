# Design Data Intensive Applications

## Table of content

- [Design Data Intensive Applications](#design-data-intensive-applications)
  - [Table of content](#table-of-content)
  - [Sources](#sources)
  - [Context](#context)
  - [Performance is a feature](#performance-is-a-feature)
    - [Engineers are data engineers](#engineers-are-data-engineers)
    - [What people expect from applications](#what-people-expect-from-applications)
    - [Comorbidity](#comorbidity)
    - [Performances](#performances)
    - [Take off the developer hat](#take-off-the-developer-hat)
  - [What is latency ?](#what-is-latency-)
    - [Main components](#main-components)
    - [Diagnosis: why are systems considered slow?](#diagnosis-why-are-systems-considered-slow)
    - [Possible roads to action ?](#possible-roads-to-action-)
  - [Performant protocols](#performant-protocols)
    - [What goes inside the TCP protocol?](#what-goes-inside-the-tcp-protocol)
    - [Risks when two systems talk to each other with TCP](#risks-when-two-systems-talk-to-each-other-with-tcp)
    - [The UDP protocol](#the-udp-protocol)
    - [Why is it a generally bad idea to reinvent the wheel](#why-is-it-a-generally-bad-idea-to-reinvent-the-wheel)
    - [TLS / SSL](#tls--ssl)
    - [Physical limitations to communications](#physical-limitations-to-communications)
    - [Size of a web app](#size-of-a-web-app)
  - [You need performant databases](#you-need-performant-databases)
    - [Relational or not relational?](#relational-or-not-relational)
    - [Questions to ask](#questions-to-ask)
    - [What about other databases?](#what-about-other-databases)
    - [How do we use databases to improve performance?](#how-do-we-use-databases-to-improve-performance)
    - [Digression: B-Trees](#digression-b-trees)
    - [Column Storage](#column-storage)
    - [Materialized views](#materialized-views)
    - [Encoding](#encoding)
  - [Things to take care of](#things-to-take-care-of)
    - [Know what is going on](#know-what-is-going-on)
    - [Use the correct intermediary](#use-the-correct-intermediary)
  - [Distributed data](#distributed-data)
    - [Why?](#why)
    - [What?](#what)
    - [Handling special cases](#handling-special-cases)
    - [Simple leader replication](#simple-leader-replication)
    - [Multi leader Replication](#multi-leader-replication)
  - [Partition](#partition)
    - [Deparate data in smaller databases](#deparate-data-in-smaller-databases)
    - [How should data be separated?](#how-should-data-be-separated)
  - [Transactions \& ACID](#transactions--acid)
    - [Failure and retry](#failure-and-retry)
    - [Handling concurrency](#handling-concurrency)
  - [Distributed systems and Murphy's law](#distributed-systems-and-murphys-law)
    - [Murphy's law](#murphys-law)
    - [Clocks](#clocks)
    - [Quorum and consensus](#quorum-and-consensus)
    - [Linearizable systems](#linearizable-systems)
      - [Serializable but NOT Linearizable](#serializable-but-not-linearizable)
      - [Linearizable but NOT Serializable](#linearizable-but-not-serializable)
    - [Ordering requests](#ordering-requests)
    - [Consensus: making sure that all our nodes agree](#consensus-making-sure-that-all-our-nodes-agree)
      - [Two phase commit](#two-phase-commit)
      - [Other algorithm](#other-algorithm)
  - [Philosohpy to follow](#philosohpy-to-follow)
    - [Pipe logic: Connect simple parts](#pipe-logic-connect-simple-parts)
    - [Consumer Offset](#consumer-offset)
    - [User Actions as immutable](#user-actions-as-immutable)

## Sources

DesignDataIntensiveApplications
HighPerformanceBrowserNetworking

## Context

With the increase of the number of data source. The main challenge has been handling the influx of data. It more common to have small apis/apps that handle large data than the opposite. Multi core processors encourage this. We are in the more than Moor Paradigm where compute power does not grow exponentially right now.

Boom of AI & Big data did not improve the situation.

## Performance is a feature

### Engineers are data engineers

Most applications have databases, caches, search systems... Most developers don't realize the impact of having one of these systems be poorly optimized. In order to improve performances, engineers tend to add a new system (add cache if data queries are low) but they don't always think of optimizing the system in itself.

### What people expect from applications

When people talk about software, they mostly talk about three pillars:

- Reliability (fault tolerance, human error tolerant).
- Scalability (latency does not degrade too much. We can accept spikes in users).
- Maintainability (it is easy to correct bugs and add new functionalities). It also includes having a good visibility on wtf is going in inside your system. You must have visibility, KPIs, automatic updates, fallback... You must also plan for knowledge leavving your companies, good documentation, new hires...

### Comorbidity

Careful when talking about this: failure != fault. Fault is when something unexpected happens, failure is when a system fails. Faults can cause failure. Fault tolerance is better than fault avoidance (for the same reasons making something hard to decrypt is better than hiding it).

While it is easy to deal with each fault separately, it can be risky to deal with them all at once. Systems are not independant. When you have a lot of systems, expect failures everyday and make sure they don't propagate.

### Performances

If you want to have apps with good performance, you need to engineer them that way. Users quit applications that are too slow. While we can't count milliseconds, we perceive differences in delays.

Even the smallest faults can lead to downtime / slowdowns. We need to measure these error recovery time. If you don't measure it, you don't control it.

Know who your audience is. For example: if you are twitter. You have a lot of reads and not many posts. You could use simple databases and do joins in order to finds tweets. This approach is slow so it is better to have a separate queue or table for each user. This means that we have a lot of work for each post but it works. We can also try a hybrid approach where depending on notoriety of users, we handle tweets differently.

### Take off the developer hat

While most of the topics handled in the book are tech topics, a lot of them are larger than that and deal with psychology, website experience, communication, synchronizing teams... Take off the developer team once in a while.

## What is latency ?

### Main components

Reponse time is what the user sees. There is the time to process requests but also network delays and other. Latency is time it waits to be handled. Latency has four main components:

- Propagation delay (time to travel)
- Transmission delay (time to push data). Depends heavily on bandwidth. Bandwidth is rarely a problem.
- Processing delay (time to process headers)
- Queueing delay (time in queue)

Most requests are fast, some are slow. Use percentiles to measure speed.

SLOs and SLAs describe agreements for users.

### Diagnosis: why are systems considered slow?

The smallest bandwidth conditions the whole system (imagine multiple pipes with different sizes). Smallest sizes conditions the other pipes. Bandwidth is rarely the culprit. We have good optic fiber & compact data.

When you have a system that needs multiple parallel operations / calls. The slowest call slows down the whole transaction ==> optimize for slowest calls.

BufferBloat: We get a lot of requests that end up in the buffer because we don't have the mean to treat them all.

**The last mile problem**: During most of the travel, requests have the spped of light. Problems appear in last mile when we have to redirect requests in areas that are densely populated. Lots of machines. Use traceroute to check that.

### Possible roads to action ?

We could increase treatment speed of items in queue: We often talk about scaling up (vertical scaling) vs scaling out (hoorizontal scaling). Truth is you need a bit of both. Elastic scaling is good for spikes but mostly unpredictable.

Reducing the distance between machines could be a solution but it isn't always possible and last mile is the problem. In order reduce to the number of calls and the distance of calls, we can use a mix of CDN and cache. ==> Most obvious way to increase the efficiency of calls and reduce latency is by making more efficient protocols. Or to **send less data**.

All measures can be complicated to put in place since the task of deploying / handling architectures and developing are not always handled by the same teams.

## Performant protocols

A bit of vocabulary:

**IP Protocol**: IP handles addressing and wrapping calls. Delivers based on IP addresses. It makes sure the packages get to an address (IPV4 or IPV6). It is a best effort protocol. NextHops and addresses. Source and desitnation Address.

**UDP Protocol**: UDP does not need a connection, no handshake protocols and no ack. You don't detect when packages are lost. Used for streaming.

**TCP Protocol**: Protocole de communication pour pouvoir discuter de façon sécurisé et fiable sur un réseau mouvementé. Orienté connexion. TCP has handshake to create a connection. Packages have ack. Slower but more secure. Used for web, email...

**NAT**: network address translation. NAT serves as in intermediary for networks. Networks have reserved ranges, careful.

**DNS**: phonebook of the internet (translates addresses to IP address).

Know your hierarchy: Http defines the content of the page. It uses TCP to ensure reliability, breaks in segment, checks for error and sens them in order. I handles routing to the target. Moves accross routers.

We will not focus too much on the modality of exchanging between machines (sockets vs ServerSentUpdates).

### What goes inside the TCP protocol?

TCP has a three way handshake (Syn, SynAck and Ack). You need three round trips before transporting any meaningful message. Creating connections is complicated ==> Reuse connections.

There have been improvements on TCP but we still need that round trip.

If you don't need every frame or don't need correction, you can use UDP (ex: streaming).

### Risks when two systems talk to each other with TCP

**Congestion collapse**: if one of the systems has a higher bandwitdh / rate than the other, he will send a lot of copies of the same message and not wait for response. We fill the other system's buffer ==> collapse.

==> Flow control. Flow control is a mechanism that manages the data transmission rate between two nodes to ensure a fast sender does not overwhelm a slow receiver. It prevents data loss by coordinating the flow before the receiver's buffer overflows. How do we handle that?

- Stop-and-Wait: The sender transmits one packet and waits for an acknowledgment (ACK) before sending the next.
- Sliding Window: The sender can transmit multiple packets (defined by a "window size") before requiring an ACK. This is used in TCP.
- Signal size of buffer in every message.

==> Congestion control / Congestion avoidance. We want to avoid overwhelming the network. Start with small window and increase gradually. Exponential growth and multiplicative decrease (decrease size by 2). Be careful to not start too slow. If you only double size, don't start at 2.

We make the assumption that packets will be lost and try to program our way around this. We use packet loss as an indicator.

**Warning**: receiver does not look inside the packets until it has received every one of them.

### The UDP protocol

In UDP datagrams, information for IP packets are optional (example: source port and checksum can be dropped). No guarantee on message delivery & no guarantee on delivery order. No congestion control.

NATS and UDP: Since UDP is not connection oriented and UDP does not create connections. NATs are used to create a connection and close it when it is finished, in UDP, we don't know when to start and when to end. When we want to use UDP behind a NAT, we tend to use a different protocol to bypass the NAT (STUN or TURN).

Careful, you will also have to reimplement congestion avoidance, retransmission on loss and control rate of transmission.

### Why is it a generally bad idea to reinvent the wheel

On internet, we tend to have much more intermediaries than we think (proxy, nat, dns...) Each of them can change packets and can flag them as dangerous if they are not recognized. Since most of the intermediary won't look inside, they won't be able to handle your custom solution. Do not deviate from standard solution too much if you don't want to be flagged.

Websockets and smart stuff tend to wrap their solutions in TCP or HTTPS tunnels in order to avoid problems.

### TLS / SSL

(note: TLS is an upgraded version of SSL). We use both kinda interchangeably.

TLS is one below UDP and TCP. First step is TCP connection then TLS handshake, then HTTP.

Encryption protocol. Added on top of TCP. HTTPS = HTTP Secured.

TLS offers encryption, authentication and integrity (detect message tampering). TLS uses **Public key cryptography**. We use public key cryptography to exchange a private key and then use symmetric key cryptography which is much quicker.

PORT 80 reserved for http and 443 for https.

TLS creates a new session. Try to reuse session to avoid another roundtrip for the TLS handshake. Session ID and Session ticket to indicate the data to use (session state stored in a ticket).

Certificate verification to verify trusted authorities. We use this to avoid having to verify each website keys. We have certificate authorities which verify certificate keys. In case a certificate is breached, we revocate the certificate.

Don't include unecessary certificates in your browser, it takes time to check certificates.

### Physical limitations to communications

In order to send a message, you need a frequency range on which to send your message. All frequencies don't have the same performance. You have codified spectrum allocation so you can't do much about that but it is a limiation.

You also need to avoid sending noise for noise take space on your signal. The more you have noise on your channel, the stronger your signal will need to be.

### Size of a web app

Everytime you load a web application, you load more than 1Mb of code and data coming from multiple sources. Data is sometimes streamed from multiple sources. Since page size is growing fast, you need a fast network to send all of this data.

**You need to let your user interact with the page before it is fully loaded**.

When we open a webpage or a web application, we fetch hundreds of small resources ==> The limiting factor is roundtrip latency. We wouldn't get much from having a faster network. Except when streaming video or something.

Make sure you compress all of your data before sending them, avoid redirects if you can. They lead to DNS lookups, TCP connections...

KeepAlive and reuse sessions. On average, we have ~100 connections to server. If we had to create a new connection everytime, it means, ~ 100 round trip time to add to our total time.

## You need performant databases

A bit of vocabulary:

- OLTP (Online Transaction processing) Access pattern for transactions. Prioritize small fetch with random access.
- OLAP (Online analytics processing) Access pattern optimized for analytics. For very large data. Optimized for bulk import.
- Data Lake (raw data)
- Data Warehouse (Normalized data)
- Data Mart (Aggregated data)
- Star Schema: A fact table in the middle and dimension tables to the side.
- Snowflake if even more broken down with multiple fact tables.

### Relational or not relational?

SQL has been the most popular format for years. NoSQL has gained traction. Notably because you can keep all the data in a json in the same place. You can have your whole resume as a JSON and not split it in multiple tables. Trouble is: you do not enforce a schema in NoSQL. Errors are more complicated to detect and handle.

Furthermore, you need to implement standards at a moment and it can be easier to do it by column (example: city names).

NoSQL allows for easier individual updates (just add a field / change the field) but makes for more complicated mass updates and migrations. You need to support all old formats on read.

In all cases, you will need to have a certain amount of backward compatibility, because users won't update directly when the code format change and you will change your own format to add new information when specs change.

**IMPORTANT**: one of the hidden merits of having a schema is that it is easier to document and version. Since everybody has the same schema, we can archive it and communicate when it changes.

### Questions to ask

- Do I have to retrieve all of the data at once or do I need to retrieve just a part everytime?
- Would my schema change often?
- Do I have many to one relationships (relational is better for that).
- What volume of data do I have?
- How should I optimize my code?

SQL has imperative language, which is hard to parallelize. NoSQl is often declarative (you declare which result you need), which is easier to parallelize. CSS is declarative. MapReduce is in the middle.

### What about other databases?

Graphs: Very practical when relationships between entities are varied or numerous. Easy to add relationships, can be complicated to delete them or delete nodes. Not widespread because can cost a lot.

Triple Store: (subject, predicate, value). (Alice, BirthYear, 1964).

A lot of other things too. We will not study all of them.

### How do we use databases to improve performance?

Data bases are basicallly machines to store data. You need an index to access a specific data and avoid having to reread every sequential information.

Choosing an index is crucial. Indexes speed up read but slow down write (adds overhead).

==> How do we search indexes? Exact matching but fuzzy matching is also possible (we define distance measure and accepted distance). Example: Search index, cosine and 0.1. Can be really tricky.

==> Know what you are optimizing for (read / write / updates...)

### Digression: B-Trees

B-trees are a data structure where each node contains an id value. Each child contains the child ids (example: parent has 200, children are 2X0. Child of 2YO are the 2YX value). Can also be used for search systems where you have to give most common searches starting by a substring.

In our case, each node have 10 children. We can split to 5 or 20 children. We want to be balanced.

Like a hotel.

==> Logging B-Trees. In order to avoid problems, you can store all the operations as logs so yo can rebuild your B-Tree. Each B-Tree node can be saved separately.

### Column Storage

Store your data columns by columns. Practical if at a given moment, you only need to access one column at a time. Example: parquet format. Very good for data compression because you don't need to store every value since value repetition is very common in most columns.

Just be careful when writing new data, you might need to decompress & recompress (more optimal writes be done with LSM trees).

### Materialized views

The result of a function on data. Used to sum, get min, max count... Result is cached if no change happens on data.

### Encoding

Regardless of the data type, you will have to pass your info around. Therefore, you need encoding. Careful with language encoding library, they may not be compatible with other languages. Keep to standardized structures and even they have problems ==> csv does not differenciate string containing ints adn strings. ==> Always check how your data is encoded.

Note: we can have optional and required fields. More often than not, it means we check for the presence of a value but our encoding will be the same.

**Reader Schema vs Writer schema**: you encode data in a schema and read it into another. You don't need the schemas to be the same, just to be compatible. You can have some values required on write and uninteresting at read (example: version number).

## Things to take care of

### Know what is going on

**Question**: what happens if an older version of the code reads a new data and have to write on it? Does it switch back the code to the old version? Answer should be no. Make sure it is when implementing your code. Make sure you also don't delete a field buy accident. Make sure you write back all the fields in the type they were before.

**Question2**: When you have APIs which interact, make sure you always get an answer if a system fails and the logs. If an API / A thread silently crashes, you are doomed.

Make sure you know when you are calling a distant service and handle it accordingly.

### Use the correct intermediary

If you have a lot of requests, use a message broker. It might use some more time but:

- Retries when needed.
- Stores failure.
- If coded well, you can afford your message broker failing.
- Message broker can have it's own redirect table so if there is a change in the IPs, you can handle it at the broker level.

## Distributed data

### Why?

- Scalable? Maybe your data is too big and you need multiple machines to store it all.
- Fault tolerance? Maybe you need to have backup in case of trouble or you need high availability.
- Latency? You have users around the world and you need to get data as quickly as possible to users that are far away. Note: if you have problems with servers being too far away, you can't scale vertically your way out of this problem. You have to scale horizontally.

### What?

- Replication. More of the same data elsewhere.
- Partitioning.Split your whole data in smaller subsets so that different partitions can be assigned to different nodes. Sharding.

Both of the methods are not mutually exclusive. In fact, you will probably need both.

First step: Synchronous or asynchronous replication? How do we handle failed replicas? A common solution is the leader based replication. We have a leader that handles replication on other nodes. Writes only happen through the leader who communicates the changes to the followers. Reads can happen through any off the replicas.

Note: leader / follower can be synchronous or asynchronous. If synchronous: be careful. Whenever a node is down, you will endlessly retry to communicate change to it. Advantage is that at least you are sure that all the data are synchronized with no risk of falling out. Having everybody synchronous is too complicated so sometimes, we have one synchronous to serve as backup in case the leader dies and the rest is asynchronous.

Asynchronous is therefore most times prefered.

In order to add replication nodes, start by tacking a dump from the data and then request all changes from the leader that have taken place since last value.

### Handling special cases

Node outages (voluntary like security patch or unvoluntary). You need to have a catch up protocol. If you have a leader failure, it can be trickier. You need to promote one of your followers to leader. The new leaders communicate to all the followers that it is the new leader.

You can detect which node has failed with the heartbeat protocol.

**How do we handle changes from the old leader that have not been communicated yet?**
Also make sure that we don't have 2 leaders or that in our heartbeat protocol, our calls were not just delayed. Our old leader needs to be really dead in order to replace him.

### Simple leader replication

We could use queries but this could lead to problems ==> Autoincrement or datetime.Now statements are not guaranteed to give the same result if run on another machine. These are only but two examples and there are more so we will try to rely on other methods for our replication.

A better method would be that every new data is written to the logs and we have systems to notify that the logs have been updated with new values. BUT Be very careful to check if your logs are compatible with newer versions / other versions of your system. You wouldn't want to have to update all of your databases at the same time.

**Replication Lag**: Since data is read asynchronously, you can see discrepancies if the data you just added has not been replicated yet to our followers. Eventual consistency. ==> Read after write consistency. If a user writes something, he should be able to read his result right after that. ==> Some things should be read from the leader (example: the user profile can be modified. You want to show the last version to the user ==> Read it from the leader). You could also track time since last edit and choose from where to read it accordingly... The bigger your system becomes, the more complicated it is to handle that you end up with the same leader in the same datacenter....

**Monotonic Reads**: If yu read the same result from two followers with one of them slower than the other, you might end up with a situation where you see a result and then the results disappear because they do not exist yet in the follower with the old data. ==> Each user must read data from the same follower.

**Consistent prefix reads**: You must show data in the correct order. ==> Keep causal dependencies of data.

### Multi leader Replication

We have seen the challenges of having a simple leader replication. What should we do to avoid that? We can have multiple leaders. We use this architecture when we have multiple datacenters in order to avoid having writes that can happen only in one datacenter.

Multi leader architecture lets us tolerate a complete datacenter outage and gives us better performance. If we have local network issue, we are resilient. CAREFUL: we can't use autoincrement in a distributed environment. Use key described in System Design course.

Multiple editing in a document is a database replication problem. Take the changes of all the users, log them in a database and make sure the change is synchronized on all the databases. Double booking problem.

How to avoid conflicts in a multi leader situation. Conflict resolution can be done on read or on write.

- Each write has an ID, biggest ID wins ==> Careful, we will always have data loss.
- Each leader has an ID, biggest ID wins ==> Careful, we will always have data loss.
- Merge values somehow (ex: git).
- Record all the changes and merge at a later time.

**Topology**: Do all leaders send their changes to all the other leaders? Do we have a star shaped topology, do all leaders only send to their neighbor?... Each approach ahs their own challenges: if neighbors or star replication, what happens if a node fails? In everyone sends to everyone, we send a lot of messages. Problem of causality too.

**NOTE**: leaderless replication. You have no leader. It's like if everybody is the leader. In order to have the last information, you need to have n_read + n_write > n; With n_read the number of machines queried to read an information and n_write the nuber of machine the new data is written to. Choose n_write and n_read dependingg on what you want:  fast load or fast write or a middle of both. It is also a good practice to have mechanisms to "repair" data.

The trouble with quorum is if we have part of the network that is down / disonnected and we don't have as many answers as expected. There are some ways to still work (sloppy quorum where we consult less than n nodes overall). These ways of working are risky. Careful.

## Partition

### Deparate data in smaller databases

See Sharding and consistent hashing (System Design interview book is the best).

Other approaches to store data is by key range.

Make sure to have good IDs in order to have a way to retrieve data in a certain time frame.

### How should data be separated?

You should separate data on a value you search on. As much as possible, you should separate data to avoid skews. Question is: what happens if we have secondary keys? You will have to create your partition in all your shards and search on all your shards which would increase latency.

**Proposal**: have a global index which routes data based on all partitions. You will have to partition this global index but you can partition it differently than other nodes in order to avoid splitting partitions...

**REBALANCE**: you most liklely won't get it right from the get go. Make sure you have some rebalancing mechanisms. How do we even do that? Consistent hashing is still good. Make sure to have ways to evaluate your rebalancing and that it makes sens for your problem.

**What about routing**? Our data is separated in shards. How do we route the user to the correct shard? Easiest way is to have a routing intermediary. The user should not know about the way the data is organized and the nodes should not be aware that there are other nodes.

## Transactions & ACID

Reminder:

- Atomicity (works or fails completely)
- Consistency (valid state to valid state)
- Isolation (independant transactions)
- Durability (durable changes)

**IMPORTANT**: ACID is a theoretical situation. A lot of databases with ACID in fact adopt weaker versions of ACID and there can still be concurrency issues. In order to keep things simple: **KEEP transactions small**!

In order to have 1 and 3, you can have a commit system for transactions (SaveChanges) as well as compare and swap. Locks and this kind of thing. Careful with race condition. Very hard to test consistently.

Having transactions help us ground the whole discussion on handling data. There are ways to do without transactions.

### Failure and retry

**Failure and retry**: Retrying on failure is a good way to work with transactions. However, there can be trouble (example: if query costs too much memory, it will always fail). ORMs do not always have an automatic retry system.

Basic security principles for retries:

- Log results of transactions.
- Max number of retries with exponential backoff.

### Handling concurrency

- Dirty reads when on a read, you can view data that has been committed but not push. Have a way to remember old values during transactions so you can return them in a read.
- Dirty write when you can write on data committed but not pushed (if two writes happen at the same time, the second one must wait for the first to finish). We don't know in which order they will happen though. **Row level locks** are sometimes used to prevent dirty writes.

Having a read database can avoid problems (the read database is updated every so often). We don't want reads to block writes, we don't want writes to block reads and we don't want reads to see intermediary results currently being modified by writes.
We could end up with big problems if a dirty read happens in a backup of before automatic launches of requests.

In the case of counter increment. If both users want to increment counter by one, we will still have an increment by two. How do we do that?

**Consistent snapshots**: Each transaction must have an ID and know from which version it is making a transaction from. Writes made by aborted transactions are ignored. Writes by transactions with a larger ID are ignored.

**Write skews**: You have two request. They each ask for something and check a condition. Since they were launched at the same time, they both check the condition and launch the operation. Example: double spending. You see a value in the past and therefore do updates. We could create dedicated variables to identify and avoid race condition but this is not sustainable and could cause headaches.

**Serialization**: the fact that the result of a parallel execution is the same as if we had executed each query one after another.

- You could execute all transactions in order andd not in parallel but be careful ==> Long transactions would penalize other transactions.
- 2 phase locking (2PL) Readers could block writers. Readers don't block writers. Problem is that if transactions are long or have trouble, you can end up stuck. You can have deadlocking and so on...

## Distributed systems and Murphy's law

### Murphy's law

Always assume something is not working. It is very dangerous for a system to give bad results. We would rather they crashed completely - most times -. The bigger a system, the more likely it is that somethin is broken somewhere. Cloud services make it easier to just "rebuild" a new instance of a machine.

When you spot an error, a solution is just to fail and recover / restart. This can work with systems that are separated in microservices. This can be more difficult on other services. The important element is **detecting faults**. There are a lot of subtle problems that can show up (example: packet received but crashed before handling).

Timeouts: Short => There might be some false positives where actions are marked failed when they completed and we end up doing the work twice. Long => You have to wait a lot before detecting faults.

### Clocks

There are two different times that interest us: durations and precise times. Precise times mean launching tasks at a precise moment whereas durations ean waiting for some time. For precise time, you need clock synchronizations. For durations too : if you have a timeout of 5 minutes and a machine that runs 5 minutes late, all of its request will be interpreted as timed out.

There is also the question of timezones.

Leap seconds?

If you can it is safer to trigger elements in a series of events one after another rather than triggering them on time. Problem is if an intermediary step fails, you need a way to relaunch the whole thing.

### Quorum and consensus

The truth is what the majority thinks. If all the nodes think a node is dead, act as if it is.

In order to avoid an old leader to try do stuff, there is a token indicating the current "cycle". Everytime the leader is renewed, the cycle is incremented. An old leader will have the old token and its writes will be rejected. That is called fencing.

See Byzantine fault tolerance. This can be necessary in critical environments but not for most problems.

### Linearizable systems

A system is linearizable if it seems that steps are made in a specific order and that there are no concurrent operations. If a value updates for a user, it must update for all the other users too. Yoou need this to prevent double booking and duplicates. In order to have a linearizable system, you need to have a single leader, you can't have linearizability with multiple leaders. In order to have linearizability, you can ask for a read repair everytime the user asks for a read. The cost in performance can be very high and the gains are not great (if you don't know you are supposed to have another answer from the database, you don't notice).

**NOTE**: if your system is linearizable, it has causal consistency. We know what is supposed to have caused what. This is not guaranteed if we have parallel calls.

Serializability != Linearizability.

- Serializability is about multi-operation transactions. It ensures that if you run a bunch of transactions, the end result is the same as if you ran them one by one in some order. It doesn't care about real-time order.
- Linearizability is about single-operation consistency on individual objects. It ensures that once an operation completes, every subsequent operation (in real-time) sees that result. It is essentially "strong consistency" for a single point in time.

#### Serializable but NOT Linearizable

This happens when the system guarantees that transactions occurred in some valid serial order, but that order violates the actual timing of when the transactions happened.
The Scenario:

Imagine a distributed database with two nodes (Node A and Node B) using asynchronous replication.

- Transaction 1 (T1​): At 10:00 AM, Alice updates her profile picture on Node A. The system confirms the update.
- Transaction 2 (T2​): At 10:05 AM, Bob looks at Alice’s profile on Node B. Because of a network lag, Node B hasn't received the update yet, so Bob sees the old picture.

Why this fits:

- It IS Serializable: There is a valid serial order (T2​ then T1​). If Bob had looked before Alice updated, the result would be identical. The database history is a "legal" sequence.
- It is NOT Linearizable: In real-time, T1​ finished before T2​ started. A linearizable system requires that once T1​ is committed, any subsequent read (T2​) must reflect that change. Since Bob saw "stale" data after the write finished, linearizability is broken.

#### Linearizable but NOT Serializable

This occurs when every single read and write is instantaneous and reflects the absolute latest state, but the system allows "interleaving" that ruins the atomicity of a multi-step transaction.
The Scenario:

Imagine two people trying to increment a counter that starts at 0. Each "transaction" consists of two separate steps: Read the current value, then Write (Value + 1).

- User A reads the counter (0).
- User B reads the counter (0).
- User A writes 1.
- User B writes 1.

Why this fits:

- It IS Linearizable: Every individual Read and Write operation was processed perfectly. When User B read the counter, it was indeed 0. When User A wrote 1, the register updated instantly. There is no stale data or "time travel."
- It is NOT Serializable: If these were true serializable transactions, the result should be 2 (as if User A went, then User B went). Because the steps of the transactions interleaved, the result is 1, which is impossible in any serial execution (0→1→2).

### Ordering requests

If you want to generate sequences of numbers and increment values in a distributed system, you need to include the ID of the machine who did the operation and the counter value. This way, each iteration can now be uniquely identified by the id of the node that did the operation.

This also gives us a way to know who had which information when they incremented. They include the maximum they have currently seen on every request. When a node receives a request or response with a maximum counter value greater than its own counter value, it immediately increases its own counter to that maximum.

Problem is that most times, you need to be able to check if a value is the last value right now and not wait to compare with the others (example: double booking).

**TOTAL Order Broadcast**: we guarantee that the apps will see the messages, all of them and in the same order. We don't guarantee WHEN they will see them. It is like if you had a big log pile and have to check the logs every so often. When you want to do an operation, you check the logs. All logs are delivered in the same order.

Problem is always a question of: how do we have a big pile of logs and everybody agrees on their order?

### Consensus: making sure that all our nodes agree

#### Two phase commit

In order to check that all the nodes agree on which data to use, you can have a two phase commit: prepare everyone for commit and then commit. We can only go to the next phase if everybody is ready. We need a coordinator to coordinate all the nodes to prepare and commit.

If during preparation, someone says no, we abort the write. If during commit someone does not answer, we retry until everybody says yes.

If the coordinator dies, we have to wait for him to restart, pick up his transaction log and procede.

Problem is that the when the system breaks, it breaks hard. it is not particularly quick either. The coordinator must be at least replicated. The replicator logs to know where we are at is a crucial information. It must be kept somewhere.

#### Other algorithm

In order to have consensus without the risks of the coordinator dying, we can try to have an elected leader. The difficulty is to have the election done. We must also make sure that the election process does not take too long or we lose time and don't do work.

We also have got to decide haow to do stuff when the number of nodes change and scale. A membership service and service discovery system can help with that.

## Philosohpy to follow

### Pipe logic: Connect simple parts

Connect programs like pipe who do one action very well and return a clear result. For this kind of logic to work, you need good interfaces and a standardization of input and outputs.

With pipes, it is also pretty easy to identify what is going on in which order and to test each part individually.

MapReduce follows this logic.

### Consumer Offset

If you have a pipeline of messages and a user wants to see what the new messages are, look at his offset and send him only the messages with an offset that is superior to his offset.

Problem is if a consumer lags so far behind its offset points to a deleted message. If a consumer falls too far behind, raise an alert so that a human can try to fix this.

### User Actions as immutable

It is a better practice to write user actions as logs or immutable events rather than noting the effects to the database as mutable results. Commands are not user events. If the user tries to register a username, it is a command. It can fail because the username is already taken. We can create a user event if it succeeds.

If you want to make some analysis, it is useful to log all the events and not just the results. Indeed, knowing that a user put an item in the cart and put is back is important.
