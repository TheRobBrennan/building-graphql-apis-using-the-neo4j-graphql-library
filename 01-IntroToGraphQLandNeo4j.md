# Intro to GraphQL and Neo4j

## Lesson Overview

## What is Neo4j?

![https://neo4j.com/graphacademy/training-graphql-apis/_images/01graphplatform.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/01graphplatform.png)

Neo4j is a native graph database with many features and functionality making up the Neo4j Graph Platform. Specifically:

- The Neo4j DBMS
- The property graph data model
- The Cypher query language
- Language drivers using the Bolt protocol for building applications
- Graph analytics with Graph Data Science
- Data visualization with Neo4j Bloom
- Neo4j Aura database-as-a-service
- A GraphQL integration for building GraphQL APIs backed by Neo4j

## What is GraphQL?

![https://neo4j.com/graphacademy/training-graphql-apis/_images/01graphqloverview.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/01graphqloverview.png)

GraphQL is an API query language and runtime for fulfilling those queries. GraphQL uses a type system to define the data available in the API, including what entities and attributes (**types** and **fields** in GraphQL parlance) exist and how types are connected (the data graph). GraphQL operations (queries, mutations, or subscriptions) specify an entry-point and a traversal of the data graph (the **selection set**) which defines what fields to be returned by the operation.

### Important GraphQL Concepts

#### GraphQL Type Definitions

GraphQL type definitions define the data available in the API. These type definitions are typically defined using the GraphQL Schema Definition Language (SDL), a language-agnostic way of expressing the types. However, type definitions can be also be defined programmatically.

![https://neo4j.com/graphacademy/training-graphql-apis/_images/01typedefs.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/01typedefs.png)

#### GraphQL Operations

Each GraphQL operation is either a Query, Mutation, or Subscription. The fields of the Query, Mutation, and Subscription types define the entry points for an operation. Each operation starts at the field of one of these types.

![https://neo4j.com/graphacademy/training-graphql-apis/_images/01operation.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/01operation.png)

#### The Selection Set

The selection set specifies the fields to be returned by a GraphQL operation and can be thought of as a traversal through the data graph.

![https://neo4j.com/graphacademy/training-graphql-apis/_images/01selectionset.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/01selectionset.png)

The response to a GraphQL operation matches the shape of the selection set, returning on the data requested.

![https://neo4j.com/graphacademy/training-graphql-apis/_images/01result.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/01result.png)

#### Resolver Functions

GraphQL resolvers are the functions responsible for actually fulfilling the GraphQL operation. In the context of a query, this means fetching data from a data layer.

![https://neo4j.com/graphacademy/training-graphql-apis/_images/01resolvers.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/01resolvers.png)

#### Benefits of GraphQL

Some of the benefits of GraphQL include:

- **Overfetching** - sending less data over the wire
- **Underfetching** - everything the client needs in a single request
- The **GraphQL specification** defines exactly what GraphQL is
- **Simplify data fetching** with component-based data interactions
- **"Graphs all the way down"** - GraphQL can help unify disparate systems and focus API interactions on relationships instead of resources.
- **Developer productivity** - By reasoning about application data as a graph with a strict type system, developers can focus on building applications.

#### GraphQL Challenges

Of course GraphQL is not a silver bullet. It’s important to be aware of some of the challenges that come from introducing GraphQL in a system.

- Some well understood practices from REST don’t apply
- HTTP status codes
- Error handling
- Caching
- Exposing arbitrary complexity to the client and performance considerations
- The n+1 query problem - the nested nature of GraphQL operations can lead to multiple requests to the data layer(s) to resolve a request
- Query costing and rate limiting

Best practices and tooling have emerged to address all of the above, however it’s important to be aware of these challenges.

## What is the Neo4j GraphQL Library?

## EXERCISE: Exploring the Movies GraphQL API

## Check Your Understanding

## Summary
