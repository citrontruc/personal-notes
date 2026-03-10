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
    - [Replication in practice](#replication-in-practice)

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

### Replication in practice

We could use queries but this could lead to problems ==> Autoincrement or datetime.Now statements are not guaranteed to give the same result if run on another machine. These are only but two examples and there are more so we will try to rely on other methods for our replication.

A better method would be that every new data is written to the logs and we have systems to notify that the logs have been updated with new values. BUT Be very careful to check if your logs are compatible with newer versions / other versions of your system. You wouldn't want to have to update all of your databases at the same time.

**Replication Lag**: Since data is read asynchronously, you can see discrepancies if the data you just added has not been replicated yet to our followers. Eventual consistency. ==> Read after write consistency. If a user writes something, he should be able to read his result right after that. ==> Some things should be read from the leader (example: the user profile can be modified. You want to show the last version to the user ==> Read it from the leader). You could also track time since last edit and choose from where to read it accordingly... The bigger your system becomes, the more complicated it is to handle that you end up with the same leader in the same datacenter....

**Monotonic Reads**: If yu read the same result from two followers with one of them slower than the other, you might end up with a situation where you see a result and then the results disappear because they do not exist yet in the follower with the old data. ==> Each user must read data from the same follower.

**Coonsistent prefix reads**: You must show data in the correct order. ==> Keep causal dependencies of data.
