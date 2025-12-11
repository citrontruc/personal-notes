# Design pattern

This note describes interesting design patterns. For a concrete implementation, see python or c-sharp folders.

## Table of content

- [Design pattern](#design-pattern)
  - [Table of content](#table-of-content)
  - [Disruptor](#disruptor)
    - [What problem do we solve?](#what-problem-do-we-solve)
    - [How does it work?](#how-does-it-work)

## Disruptor

### What problem do we solve?

Imagine a case where we have a multiple producers and multiple consumers. Every time we send a message, it must go to all the consumers. We could have a queue of messages for every producer but it means sending a lot of messages. Most queues are either constantly full or constantly empty.

We want to solve this by creating a circular buffer.

### How does it work?

We have our actors run around our circular buffer and put messages in one of the slots of our circular buffer. While running around the buffer, if you see a message, treat it.

Solves the problem of having a lot of buffers and avoids having to redistribute to every buffer; Everybody handles information at their own speed.

Risk is overwriting data in the circular buffer. ==> Ring must be sized for peak of communications.
