# Protocol Buffers

## Table of content

- [Protocol Buffers](#protocol-buffers)
  - [Table of content](#table-of-content)
  - [What is it?](#what-is-it)
  - [Why use it?](#why-use-it)
  - [Advantages](#advantages)
  - [Careful](#careful)

## What is it?

Data serialization format & set of tools to exchange data.

Note: has some language specific components.

## Why use it?

Passing around JSON and serializing them can be very expensive in terms of delays and operations. Protobuf has better latency. You can access direct data with protobuf without specifying fields.

## Advantages

Automatic schema validation and good performances. JSON is cool but you have to create your own methods & architectures.

## Careful

Protobuf translates to binary so not human readable ==> more complicated to debug. More limited support than JSON.
