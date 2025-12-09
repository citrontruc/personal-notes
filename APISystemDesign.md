# API System design

## Table of content

- [API System design](#api-system-design)
  - [Table of content](#table-of-content)
  - [Heuristics to find solutions](#heuristics-to-find-solutions)
  - [Distributed Services](#distributed-services)
    - [API Gateway](#api-gateway)
    - [Detecting failure](#detecting-failure)
    - [Clock synchronization](#clock-synchronization)
    - [Resiliency](#resiliency)
  - [Async integration](#async-integration)
  - [PUB/SUB](#pubsub)
  - [Outbox pattern](#outbox-pattern)
  - [CQRS pattern](#cqrs-pattern)
  - [Service Registry \& Discovery](#service-registry--discovery)
  - [Saga pattern](#saga-pattern)
  - [Strangler fig pattern](#strangler-fig-pattern)
  - [Sidecar](#sidecar)
  - [Anti-corruption layer](#anti-corruption-layer)

## Heuristics to find solutions

- Write + Spike: Queue
- Latency + Global: CDN
- Load + Growth: Scale Out
- Read + Bottleneck: Cache
- Requests + Spike: Throttle
- Retry + Safety: Idempotent
- Core + Failure: Redundancy
- Dataset + Growth: Sharding
- Text + Search: Inverted Index
- Upload + Large-file: Chunking
- Durability + Failure: Replication
- Broadcast + Realtime: Pub/Sub
- Location + Search: Geohashing
- Write + Conflict: Optimistic Lock
- Untrusted + Execution: Container
- Realtime + Updates: WebSockets
- Traffic + Reliability: Load Balancer
- Distributed + Transaction: Saga pattern
- Concurrency + Consistency: Row locking

## Distributed Services

Main challenge of distributed services is having a global clock. No shared memory.

### API Gateway

All requests are sent to an api gateway which then dispatches the requests. Security checks / rate limiting and other services are performed at the api gateway level.

**Benefits**: Simple to put in place and helps you change the services easily and independently from one another. Hides internal servies from user.

**Drawbacks**: SPOF, can become bottleneck. Can slow down requests.

**Note**: can be used as a facade when you want to modernize old services. See stragler fig pattern.

### Detecting failure

In order to check that no service is down, we have the heartbeat & gossip protocol:

1) Heartbeat mechanism
    - Heartbeat messages are small signals that nodes send to each other to show they are still alive.
    - These messages serve as health checks, enabling each node to determine whether its peers are functioning properly or have failed.
2) Gossip protocol
    - Each node regularly sends a small “I am alive” message to others.
    - System considers the node dead if these messages stop for too long.
    - Like real gossip, nodes share what they know with others.

You have to balance when to send updates. Too many and you might have false positives, too few and you might react slow to failure.

### Clock synchronization

There are a few algorithms to try to keep a logical order of events:

**Lamport Clocks** use a simple counter on each machine:

- Each process starts with a counter at 0
- For every local event, increase the counter: LC = LC + 1
- When sending a message, attach the current counter value
- When receiving a message, update the counter to: LC = max(local LC, received LC) + 1

So now, if two processes interact, the one that did an action last will have a higher counter. problem is that we don't know which events are related to which other events and we can't handle concurrency.

**Vector Clocks** extend the idea on which Lamport’s clocks are based: “What if every machine knows every other machine’s order?”

If there are N machines, the vector keeps N entries, and each entry tracks that machine’s event count, M[i].

We need to have a single source of truth which happens thank to a leader election algorithm.

Managing data concurrently can get very complicated. In order to manage that, we need to have a good id that incorporates information on the timestamp of the data. See system design interview book.

### Resiliency

**Downstream Resiliency**. Downstream failures happen when your service depends on another service that is slow or not responding. To protect your system, you can use these patterns:

1) Timeouts
    - Don’t wait forever for a response.
    - Instead, set a maximum wait time to prevent requests or threads from getting blocked.
2) Retries with Exponential Backoff
    - If a request fails, try again — but wait longer after each try (1s, 2s, 4s…).
    - This stops you from overloading a service that is already struggling and prevents cascading failures.
3) Circuit Breakers
    - If a service keeps failing, stop sending it requests for a while - “trip the breaker”.
    - This prevents one broken service from causing failures across the system.
    - After a cooldown period, you can try sending requests again.

**Upstream Resiliency**. Upstream failures happen when your system receives more requests than it can handle. To avoid overload, you can use these techniques:

1) Rate Limiting / Throttling
    - Limit the number of requests a client can send.
    - This protects your system and prevents downstream services from being overwhelmed.
2) Health Checks with Load Balancers
    - Check the health of each server periodically.
    - If a server is slow or failing, the load balancer stops sending traffic to it.
    - This ensures that requests are sent only to servers that can handle them.

**Failure Causes**. To build a resilient system, you first need to understand why failures happen:

- Hardware failures: Disks can crash, memory can get corrupted, and network interfaces may fail.
- Software failures: Bugs, memory leaks, or misconfiguration can cause services to stop working.
- Cascading failures: One failing service can trigger failures in other services that depend on it.

Distributed systems fail often; it’s not a question of “if”, but when. Resiliency patterns are about absorbing, isolating, and recovering from failures rather than pretending they won’t happen.

## Async integration

Done through message queues. Useful to decouple services. Helps regulate workflow and avoid failure if one service is momentarily down.

Provides natural ways to balance loads, improve resilience and decouple.

**Careful**: Debugging is more complicated & we don't know when requests will be handled. Use when immediate responses are not required (for example in notification services).

**Dead Letter** queue for messages that have failed processing multiple times.

## PUB/SUB

Event bus to which services can subscribe. Much like the observer design pattern. Subscribers don't know about one another and subscriber does not know about publisher.

Careful: more complicated to debug. No idea when requests are treated exactly. Notably, are we sure we respect requests order?

**Useful for**: chat applications, saga pattern, database modification.

## Outbox pattern

Variant on previous patterns: instead of publishing to message queue, services are published to database (the outbox). A background process reads from the outbox and publishes events to the message queue. Processed events are marked as processed.

This has some advantages: logging & guarantee that events are published only if the whole operation is successful (after published to database).

**Careful**: how do you handle duplicates? Requires additional infrastructure.

## CQRS pattern

Pattern that separates write and read operations through two distinct workflows. Write operations are handled by a write database while read operations are handled by a read database. Both need to be synchronized every cycle.

Allows for independant optimizations of read and write operations. Enablesscaling independentaly write and read operations.

**Careful**: data synchronization. Two databases.

Use primarily when write to read ratio is high. Systems with high needs for query performance.

## Service Registry & Discovery

Centralized service which receives requests from every service. When it receives a request, it identifies the service and stores it in memory. For each service, you need IP, port and the timestamp of last registration note.

After every registration, when a service wants to know how to contact other services, it calls the server that sends the list of all available services.

Every so often, heartbeat to know which services are still available.

You can also do service discovery the other way around with a list of APIs in a DNS or something (simplest thing would be config file).

## Saga pattern

See notebook. We have a central service which coordinates microservices. The central service sends requests in queues and checks other queues to see where we are. Is a SPOF. We could have the micro services talk to one another directly but then they would be coupled. No need for lock, everything is handled by central coordinator.

**Careful**: need procedure to undo changes done by one service in case of failure. Monitoring becomes quite complicated. Sometimes, it is easier to have an event bus.

## Strangler fig pattern

We gradually replace a legacy system with a microservice architecture. New system gradually takes responsabilities until fully replaced. Works well with api gateway.

Reduces risk of migration. if you want test perdio, have legacy and new software compute solution and compare them. Legacy system sends solutions. No user disruption.

**Careful**: more complicated for users. Requires additional infrastructure. Testing is more complex.

Use when move to cloud or there are outdated services you want to modernize.

## Sidecar

You bundle a sidecar service in your containers. The sidecar handles logging, events and networking. Allows you to separate concerns and change side activities without modifying services. You can handle multiple sidecars if you want.

Gives reusable components of code, standardized implementation of traffic management, security & observability. Sidecar can be the one listrning to event bus while the service only handles code logic.

**Careful**: increase each service size. Orchestration of deployments when you change the sidecar. Latency?

## Anti-corruption layer

Have a layer betweeen your services and outside services to normalize data and check their values before transmitting them.

**CAREFUL**: bottleneck + you need to change it every time external services change.
