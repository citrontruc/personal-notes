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
    - [Identify your domains](#identify-your-domains)
    - [Challenges](#challenges)
    - [Communications](#communications)
    - [Availability](#availability)
    - [Logs](#logs)
    - [Metrics to keep in mind](#metrics-to-keep-in-mind)
    - [Don't start with microservices](#dont-start-with-microservices)

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

Kubernetes, Docker, cloud run, container apps (built on top of kubernetes), cloud brokers... Each app needs to have its own technical environment and can evolve freely. However, there are standards, so make sure you have the right expertise with kubernetes.

You can build on top of your own virtual machine. You just need to handle everything.

### Conway's Law

Companies build softwares that tend to resemble their style of communication. Microservice systems tend to appear in companies where teams are independent and can scale easily. You have existing and resilient communication channels.

A possible explanation for this law is the ownership of information / data. Business Units will probably have different data of interest and if you want to separate you applications in services, doing so by data sources can be a good thing.

## Microservices need a solid technical understanding of systems

### Identify your domains

A quick example of how to model an app for microservices is this one: <https://learn.microsoft.com/en-us/azure/architecture/microservices/model/domain-analysis>.

You first need to map out what you want to achieve and where you can find the information / where you will need to store it. Having too many services will bog you down, so try to have domains that make sense from a business point of view. DDD works quite well with microservices.

An antipattern is the junk drawer anti-pattern: you end up piling all of the miscellaneous capabilities in the same spot.

If they are not well designed, your whole system might be as slow / weak as your weakest link.

### Challenges

If you don't have CICD / DevOps, you are doomed! You need strong tests too to check that your service is compatible with all the other services. You also need very good documentation practices to manage to keep track of a lot of services developed by multiple teams.

You need a good vision to put in place microservices. You need to identify behaviour for each of your services and map a solution.

Code replication. All of the services will need authentication, logging...

You don't just need to create multiple git repos. Doing microservices is about being loosely coupled. This means keeping simple and standardized interfaces for each of your apis.

You need strong reliability engineering & some cloud most likely.

### Communications

Having different services mean that we will have latency everytime a services calls another. Not necessarily much but more than if we had loaded the file in memory.

RPC communications are often used with microservices. Message driven with queues can also be a solution (can be good when you need to communicate to multiple services or you don't know who to communicate to). Dead letter queue & just long queues can make sure that at least no messages are lost during spikes or when a service goes down. However, more complicated to put in place & eventual consistency. If you want to have an order of edxecution, see Saga pattern.

You need constant service discovery in case there are multiple regions, a machine goes down or other...

### Availability

How do you scale dynamically and make sure that errors are detected and lead to a recovery? What is the % of availability? What is your retry strategy? How many and when do you launch a retry? => We need a standardized policy.

Having a health check endpoint is a good solution to have a way of verifying availability.

### Logs

OpenTelemetry is a language neutral framework that can help monitor everything in the same place.

### Metrics to keep in mind

- Deployment frequency: If you need time between each deployment, adding complexity won"t help.
- Lead time for changes: if there is always a lot of time between changes and deployment, don't do microservice. Microservices need people to be reactive. Avoid manual steps & slow testing.
- Change failure rate: if you can't make changes without breaking stuff, you will have problems at each deployement. Cascading failure.
- Mean time to recovery: When a problem is met, you nee to correct it fast.

If you don't have good KPIs in the needs above, you won't be able to handle well microservices.

### Don't start with microservices

If you have a big monolith, don't try to restart with microservices or to separate everything. First, respect all the solid principles and when you identify some features that are on the side that can be separated from the rest of the system, create a microservice for these features and gradually split your monolith. The trajectory is more often: monolith -> modeular monolith -> microservice. If you don't need to go all the way, you can stop at modular monolith.

For services to be truly independent, they need their own databases. You need good telemetry, good traces and take time for configuration to find what helps you have information ahead of time.
