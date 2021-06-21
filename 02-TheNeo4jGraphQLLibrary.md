# The Neo4j GraphQL Library

## Lesson Overview

We will spend the rest of the course building a GraphQL API for an online bookstore using the Neo4j GraphQL Library. In this lesson, we set up our online development environment using Codesandbox and Neo4j Sandbox. We then see how to use GraphQL type definitions to define a property graph data model in Neo4j and create a GraphQL API using the Neo4j GraphQL Library. We’ll create data using GraphQL mutations and see how to query it using GraphQL.

## Setting Up Your Environment

Now it’s time to start using the Neo4j GraphQL Library to build GraphQL APIs! We’ll be using a browser-based service called Codesandbox to run Node.js JavaScript code, which in this case will be a GraphQL API application making use of the Neo4j GraphQL Library. Using Codesandbox means we won’t have to troubleshoot local development environment issues and makes it easier to share code. Each lesson will start with an initial skeleton application that we’ll modify as we explore new concepts. If you get stuck, each lesson also has the solution available as a Codesandbox.

We’ll also be using Neo4j Sandbox to spin up a hosted Neo4j instance in the cloud. We’ll connect to our Neo4j Sandbox instance from the GraphQL API application running in Codesandbox. To do this, we’ll use the connection credentials specific to our Neo4j Sandbox instance.

Follow these steps to get your development environment set up using Codesandbox and Neo4j Sandbox:

1. Create a blank Neo4j Sandbox instance using [this link](https://sandbox.neo4j.com/?usecase=blank-sandbox&_gl=1*1plit0m*_ga*MTc1NjI5MjQ3OS4xNjE5NDY3NjQ2*_ga_DL38Q8KGQC*MTYyNDI0NTM4MC43LjEuMTYyNDI0ODQyMi4w&_ga=2.245954188.947414018.1624236956-1756292479.1619467646). You’ll need to sign in to Neo4j Sandbox if you’re not already authenticated, then click the "Launch Project" button.

![https://neo4j.com/graphacademy/training-graphql-apis/_images/02blanksandbox.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/02blanksandbox.png)

2. Once your blank Neo4j Sandbox instance is ready, you can go through the guide or close it. Then navigate to the "Connection details" tab for the sandbox instance. Make note of the "**Bolt URL**", "**Username**", and "**Password**" values for your sandbox instance. You’ll need these values in the next step to connect to Neo4j from the GraphQL API application we’ll be building and running in Codesandbox.

![https://neo4j.com/graphacademy/training-graphql-apis/_images/02blanksandboxconnection.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/02blanksandboxconnection.png)

3. Open the Codesandbox for this lesson using [this link](https://codesandbox.io/s/github/johnymontana/training-v3/tree/master/modules/graphql-apis/supplemental/code/02-graphql-apis-overview-of-neo4j-graphql/begin?file=/.env). This Codesandbox contains the initial code for a GraphQL API application. However, the application is throwing an error because it cannot connect to a Neo4j instance. Fix this by adding your Neo4j Sandbox connection credentials.

![https://neo4j.com/graphacademy/training-graphql-apis/_images/02codesandboxerror.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/02codesandboxerror.png)

```sh
# Example Neo4j Sandbox configuration
NEO4J_URI=bolt://52.202.15.126:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=alibi-polarity-waxes
```

4. In order to save changes to the Codesandbox, you’ll be prompted to sign in to Codesandbox so that the changes are specific to your Codesandbox. Open the `.env` file, adding values for `NEO4J_URI`, `NEO4J_USER`, and `NEO4J_PASSWORD` specific to your Neo4j Sandbox instance. Save the file and wait for the application to reload. You will now see the GraphQL Playground application running in Codesandbox.

You can test that it’s working by running the following query in the GraphQL Playground window in your Codesandbox (you will get back an empty array without any error messages).

```gql
{
  books {
    title
  }
}
```

You will see a screen like this after updating the values in the `.env` file, with GraphQL Playground allowing you to execute GraphQL operations against your GraphQL API application connected to Neo4j Sandbox.

![https://neo4j.com/graphacademy/training-graphql-apis/_images/02codesandboxcredentials.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/02codesandboxcredentials.png)

## GraphQL Type Definitions and the Property Graph Model

Now that your development environment is set up, let’s take a look at what we’ll be building throughout this course. The goal of this course is to build a GraphQL API application for an online bookstore. We’ll need to handle customers searching for books, placing orders, as well as leaving reviews for books they’ve purchased.

We’ll start with the following property graph data model:

![https://neo4j.com/graphacademy/training-graphql-apis/_images/02book_graph.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/02book_graph.png)

Let’s see how we would define this property graph model using GraphQL type definitions. Each node label becomes a GraphQL type. Node properties are defined as GraphQL fields. Relationships are expressed as GraphQL objects or object array fields and include a special GraphQL schema directive @relationship that is used to capture the direction and relationship type.

Open the **schema.graphql** file, then make sure types as defined here are in Schema of your Codesandbox.

```gql
#schema.graphql

type Order {
  orderID: ID! @id
  placedAt: DateTime @timestamp
  shippingCost: Float
  shipTo: Address @relationship(type: "SHIPS_TO", direction: OUT)
  customer: Customer @relationship(type: "PLACED", direction: IN)
  books: [Book] @relationship(type: "CONTAINS", direction: OUT)
}

type Customer {
  username: String
  orders: [Order] @relationship(type: "PLACED", direction: OUT)
  reviews: [Review] @relationship(type: "WROTE", direction: OUT)
}

type Address {
  address: String
  location: Point
  order: Order @relationship(type: "SHIPS_TO", direction: IN)
}

type Book {
  isbn: ID!
  title: String
  price: Float
  description: String
  reviews: [Review] @relationship(type: "REVIEWS", direction: IN)
}

type Review {
  rating: Int
  text: String
  createdAt: DateTime @timestamp
  book: Book @relationship(type: "REVIEWS", direction: OUT)
  author: Customer @relationship(type: "WROTE", direction: IN)
}
```

A few important concepts to note:

- The `@relationship` directive is used to define relationships.
  `DateTime` and `Point` scalar types are available and map to the equivalent native Neo4j database types.
- The `@timestamp` directive is used to indicate the property will be **automatically updated when the node is created and updated**.
- The `@id` directive **marks a field as a unique identifier and enables auto-generation when the node is created**.

Read more about using GraphQL type definitions with the Neo4j GraphQL Library in the documentation [here](https://neo4j.com/docs/graphql-manual/current/type-definitions/). For more information on the GraphQL schema directives available with the Neo4j GraphQL Library refer to [this page](https://neo4j.com/docs/graphql-manual/current/directives/).

## Generated Mutations

## Querying Data with GraphQL

## EXERCISE: Updating the GraphQL Schema

## Check Your Understanding

## Summary
