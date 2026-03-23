# Domain Driven Design

## Table of Content

- [Domain Driven Design](#domain-driven-design)
  - [Table of Content](#table-of-content)
  - [Concepts](#concepts)

## Concepts

Entity, Value Objects and aggregates.

- Entities have a unique identity that describe a business element.
- Value objects don't have an identity and are updated by replacing ex: shipping address can be a value object, PaymentValue (with value and money).
- Aggregates are a set of related entities that are grouped with a single root entity ex: Order has order items and a shipping address.

Domain events indicate that comething meaningful has happened & allows other parts of the system to act on it. Example: an event if an order is created. The idea is that we can't do actions that lead us into an incorrect state.

First step is mapping your concepts to entities. Define a common language between tech and business so that everybody can communicate on the project.
