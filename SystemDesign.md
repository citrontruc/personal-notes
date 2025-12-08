# Heuristics to find solutions

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

## Service Registry

Centralized service which receives requests from every service. When it receives a request, it identifies the service and stores it in memory. For each service, you need IP, port and the timestamp of last registration note.

After every registration, when a service wants to know how to contact other services, it calls the server that sends the list of all available services.

Every so often, heartbeat to know which services are still available.

You can also do service discovery the other way around with a list of APIs in a DNS or something (simplest thing would be config file).

## Saga pattern

See notebook. We have a central service which coordinates microservices. The central service sends requests in queues and checks other queues to see where we are. Is a SPOF. We could have the micro services talk to one another directly but then they would be coupled.

## Strangler fig pattern

We gradually replace a legacy system with a microservice architecture. New system gradually takes responsabilities until fully replaced.
