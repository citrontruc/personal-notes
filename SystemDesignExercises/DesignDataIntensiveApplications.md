# Design Data Intensive Applications

## Table of content

- [Design Data Intensive Applications](#design-data-intensive-applications)
  - [Table of content](#table-of-content)
  - [Context](#context)
  - [Performance is a feature](#performance-is-a-feature)
    - [Engineers are data engineers](#engineers-are-data-engineers)
    - [What people expect from applications](#what-people-expect-from-applications)
    - [Comorbidity](#comorbidity)
    - [Performances](#performances)
    - [Take off the developer hat](#take-off-the-developer-hat)
  - [What is latency ?](#what-is-latency-)
    - [Main components](#main-components)
    - [Diagnosis](#diagnosis)
    - [Possible roads to action ?](#possible-roads-to-action-)
  - [Digression on databases](#digression-on-databases)
    - [Relational or not relational?](#relational-or-not-relational)
    - [Questions to ask](#questions-to-ask)
    - [What about other databases?](#what-about-other-databases)
    - [How do we use databases to improve performance?](#how-do-we-use-databases-to-improve-performance)
    - [Digression: B-Trees](#digression-b-trees)

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

### Diagnosis

When you have a system that needs multiple parallel operations / calls. The slowest call slows down the whole transaction ==> optimize for slowest calls.

### Possible roads to action ?

We often talk about scaling up (vertical scaling) vs scaling out (hoorizontal scaling). Truth is you need a bit of both. Elastic scaling is good for spikes but mostly unpredictable.

This can be complicated to put in place since the task of deploying and developing is not always handled by the same teams.

## Digression on databases

### Relational or not relational?

SQL has been the most popular format for years. NoSQL has gained traction. Notably because you can keep all the data in a json in the same place. You can have your whole resume as a JSON and not split it in multiple tables. Trouble is: you do not enforce a schema in NoSQL. Errors are more complicated to detect and handle.

Furthermore, you need to implement standards at a moment and it can be easier to do it by column (example: city names).

NoSQL allows for easier individual updates (just add a field / change the field) but makes for more complicated mass updates. You need to support all old formats on read.

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

### Digression: B-Trees

B-trees are a data structure where each node contains an id value. Each child contains the child ids (example: parent has 200, children are 2X0. Child of 2YO are the 2YX value). Can also be used for search systems where you have to give most common searches starting by a substring.

In our case, each node have 10 children. We can split to 5 or 20 children. We want to be balanced.

Like a hotel.

==> Logging B-Trees. In order to avoid problems, you can store all the operations as logs so yo can rebuild your B-Tree. Each B-Tree node can be saved separately.
