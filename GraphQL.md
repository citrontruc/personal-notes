# GraphQL

## Table of content

- [GraphQL](#graphql)
  - [Table of content](#table-of-content)
  - [Definition](#definition)
  - [What to do to use it](#what-to-do-to-use-it)
  - [When to use it](#when-to-use-it)
  - [When not to use it](#when-not-to-use-it)
  - [GraphQL vs REST](#graphql-vs-rest)
  - [Versioning](#versioning)
  - [Transform Data](#transform-data)
  - [Documentation](#documentation)

## Definition

GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data. Unlike REST, which uses multiple endpoints, GraphQL uses a single endpoint where the client defines the exact data structure needed.

We have a centralized endpoint. All the information on the operations is stored in the query.

## What to do to use it

You need to deine contracts and schemas beforehand. You also have no explicit versioning because there is only one endpoint.

## When to use it

Since query are defined at the server level, it is easier to use when you have multiple frontends. No need to wait for new endpoints to be created.

## When not to use it

When using public apis, when uploading files, when operations are simple.

## GraphQL vs REST

When we want to fetch data in rest, we often get a bit too many or too few data. We need to do multiple query when we need to retrieve multiple data. In GraphQL, you specify the data you want and you let GraphQL get that exact data.

## Versioning

You don't have exact versioning but you can evolve the schema in the Graph. We can specify that we have deprecated fields with the @deprecated flag.

## Transform Data

In GraphQL, there are ways to do Groupby or data transformation but they are suboptimal. The best use case for graphQL would be for data that is already transformed. Ideally on the data mart level.

## Documentation

On possède qu'un seul endpoint qui englobe toutes les query. Le schéma GraphQL est une documentation en soit.
