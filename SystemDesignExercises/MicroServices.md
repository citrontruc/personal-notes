# Microservice Architecture

## Table of content

- [Microservice Architecture](#microservice-architecture)
  - [Table of content](#table-of-content)
  - [Presentation](#presentation)
    - [Why would you want that?](#why-would-you-want-that)
      - [Main reasons](#main-reasons)
      - [Side benefits](#side-benefits)
    - [How to put it in place?](#how-to-put-it-in-place)
    - [Conway's Law](#conways-law)
  - [Microservices need a solid technical understanding of systems](#microservices-need-a-solid-technical-understanding-of-systems)
    - [Challenges](#challenges)
    - [Communications](#communications)
    - [Availability](#availability)

## Presentation

### Why would you want that?

#### Main reasons

- You have a system with a lot of small programs that are mostly independant.
- You want to take advantage of having multiple tech stacks (choose the best tool for each service).
- You want to scale some services independently of others.
- You want to experiment with new tools.
- You are working on an emerging topic that demands frequent changes.

#### Side benefits

Microservices also simplifies team work if you have multiple services that need work from different tech teams.

If a whole service fails or is laggy, the rest is not impacted and can dance around it.

If each service has its own value, you can quantify value in your app and always improve the service which has the most latent value. You can also identify services whose incremental gains are the smallest and know you should not focus too much on these ones.

### How to put it in place?

Kubernetes, Docker, cloud run, container apps, cloud brokers... Each app needs to have its own technical environment and can evolve freely.

### Conway's Law

Companies build softwares that tend to resemble their style of communication. Microservice systems tend to appear in companies where teams are independent and can scale easily. You have existing and resilient communication channels.

## Microservices need a solid technical understanding of systems

If they are not well designed, your whole system might be as slow / weak as your weakest link.

### Challenges

If you don't have CICD / DevOps, you are doomed! You need strong tests too to check that your service is compatible with all the other services. You also need very good documentation practices to manage to keep track of a lot of services developed by multiple teams.

You need a good vision to put in place microservices. You need to identify behaviour for each of your services and map a solution.

Code replication. All of the services will need authentication, logging...

You don't just need to create multiple git repos. Doing microservices is about being loosely coupled. This means keeping simple and standardized interfaces for each of your apis.

### Communications

Having different services mean that we will have latency everytime a services calls another. Not necessarily much but more than if we had loaded the file in memory.

RPC communications are often used with microservices. Message driven with queues can also be a solution (can be good when you need to communicate to multiple services or you don't know who to communicate to). Dead letter queue & just long queues can make sure that at least no messages are lost during spikes or when a service goes down. However, more complicated to put in place & eventual consistency. If you want to have an order of edxecution, see Saga pattern.

You need constant service discovery in case there are multiple regions, a machine goes down or other.

### Availability

How do you scale dynamically and make sure that errors are detected and lead to a recovery? What is the % of availability?
