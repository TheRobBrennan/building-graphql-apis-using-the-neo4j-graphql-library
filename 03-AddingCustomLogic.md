# Adding Custom Logic

## Lesson Overview

In the previous lesson we explored building a GraphQL API using the Neo4j GraphQL Library that implemented basic create, read, update, and delete (CRUD) functionality. In this lesson we explore adding custom logic to our GraphQL API using the power of the Cypher query language.

## Codesandbox Setup

To begin we’ll clear out our Neo4j database, open a new Codesandbox, and run a GraphQL mutation to add some initial data.

First, open Neo4j Browser for your Neo4j Sandbox instance and run the following Cypher query to delete all the data in the database

> Make sure you’re running this command in the correct database as this will delete all data in the Neo4j database!

```cypher
MATCH (a) DETACH DELETE a
```

Next, open [this Codesandbox](https://codesandbox.io/s/github/johnymontana/training-v3/tree/master/modules/graphql-apis/supplemental/code/03-graphql-apis-custom-logic/begin?file=/.env) and edit the `.env` file, adding the connection credentials for your Neo4j Sandbox. Refer to the setup instructions in the previous lesson if necessary.

In GraphQL Playground running in your Codesandbox, paste and execute the following GraphQL mutation to load the initial data we’ll be working with:

```gql
mutation {
  createBooks(
    input: [
      {
        isbn: "1492047686"
        title: "Graph Algorithms"
        price: 37.48
        description: "Practical Examples in Apache Spark and Neo4j"
        subjects: { create: [{ name: "Graph theory" }, { name: "Neo4j" }] }
        authors: {
          create: [{ name: "Mark Needham" }, { name: "Amy E. Hodler" }]
        }
      }
      {
        isbn: "1119387507"
        title: "Inspired"
        price: 21.38
        description: "How to Create Tech Products Customers Love"
        subjects: {
          create: [{ name: "Product management" }, { name: "Design" }]
        }
        authors: { create: { name: "Marty Cagan" } }
      }
      {
        isbn: "190962151X"
        title: "Ross Poldark"
        price: 15.52
        description: "Ross Poldark is the first novel in Winston Graham's sweeping saga of Cornish life in the eighteenth century."
        subjects: {
          create: [{ name: "Historical fiction" }, { name: "Cornwall" }]
        }
        authors: { create: { name: "Winston Graham" } }
      }
    ]
  ) {
    books {
      title
    }
  }

  createCustomers(
    input: [
      {
        username: "EmilEifrem7474"
        reviews: {
          create: {
            rating: 5
            text: "Best overview of graph data science!"
            book: { connect: { where: { isbn: "1492047686" } } }
          }
        }
        orders: {
          create: {
            books: { connect: { where: { title: "Graph Algorithms" } } }
            shipTo: {
              create: {
                address: "111 E 5th Ave, San Mateo, CA 94401"
                location: {
                  latitude: 37.5635980790
                  longitude: -122.322243272725
                }
              }
            }
          }
        }
      }
      {
        username: "BookLover123"
        reviews: {
          create: [
            {
              rating: 4
              text: "Beautiful depiction of Cornwall."
              book: { connect: { where: { isbn: "190962151X" } } }
            }
          ]
        }
        orders: {
          create: {
            books: {
              connect: [
                { where: { title: "Ross Poldark" } }
                { where: { isbn: "1119387507" } }
                { where: { isbn: "1492047686" } }
              ]
            }
            shipTo: {
              create: {
                address: "Nordenskiöldsgatan 24, 211 19 Malmö, Sweden"
                location: { latitude: 55.6122270502, longitude: 12.99481772774 }
              }
            }
          }
        }
      }
    ]
  ) {
    customers {
      username
    }
  }
}
```

With our initial data loaded let’s dive into adding custom logic to our GraphQL API using Cypher!

## The @cypher GraphQL Schema Directive

Schema directives are GraphQL’s built-in extension mechanism and indicate that some custom logic will occur on the server. Schema directives are not exposed through GraphQL introspection and are therefore invisible to the client. The `@cypher` GraphQL schema directive allows for defining custom logic using Cypher in the GraphQL schema.

**Using the `@cypher` schema directive overrides field resolution and will execute the attached Cypher query to resolve the GraphQL field.**

Refer to the `@cypher` [directive documentation for more information](https://neo4j.com/docs/graphql-manual/current/type-definitions/cypher/).

### Computed Scalar Field

Let’s look at an example of using the `@cypher` directive to define a computed scalar field in our GraphQL schema. Since each order can contain multiple books we need to compute the order "subtotal" or the sum of the price of each book in the order. To calculate the subtotal for an order with orderID "123" in Cypher we would write a query like this:

```cypher
MATCH (o:Order {orderID: "123"})-[:CONTAINS]->(b:Book)
RETURN sum(b.price) AS subTotal
```

With the `@cypher` schema directive in the Neo4j GraphQL Library we can add a field `subTotal` to our `Order` type that includes the logic for traversing to the associated `Book` nodes and summing the price property value of each book. Here we use the `extend` type syntax of GraphQL SDL but we could also add this field directly to the `Order` type definition as well.

Add this extension to the schema.graphql file:

```gql
# schema.graphql

extend type Order {
  subTotal: Float
    @cypher(statement: "MATCH (this)-[:CONTAINS]->(b:Book) RETURN sum(b.price)")
}
```

The `@cypher` directive takes a single argument `statement` which is the Cypher statement to be executed to resolve the field. This Cypher statement can reference the `this` variable which is the currently resolved node, in this case the currently resolved `Order` node.

We can now include this `subTotal` field in our GraphQL queries:

```gql
{
  orders {
    books {
      title
      price
    }
    subTotal
  }
}
```

```json
{
  "data": {
    "orders": [
      {
        "books": [
          {
            "title": "Graph Algorithms",
            "price": 37.48
          }
        ],
        "subTotal": 37.48
      },
      {
        "books": [
          {
            "title": "Graph Algorithms",
            "price": 37.48
          },
          {
            "title": "Inspired",
            "price": 21.38
          },
          {
            "title": "Ross Poldark",
            "price": 15.52
          }
        ],
        "subTotal": 74.38
      }
    ]
  }
}
```

The `@cypher` directive gives us all the power of Cypher, with the ability to express complex traversals, pattern matching, even leveraging Cypher procedures like APOC. Let’s add a slightly more complex `@cypher` directive field to see what is possible. Let’s say that the policy for computing the shipping cost of orders is to charge $0.01 per km from our distribution warehouse. We can define this logic in Cypher, adding a `shippingCost` field to the Order type.

Add this extension to the schema.graphql file:

```gql
# schema.graphql

extend type Order {
  shippingCost: Float
    @cypher(
      statement: """
      MATCH (this)-[:SHIPS_TO]->(a:Address)
      RETURN round(0.01 * distance(a.location, Point({latitude: 40.7128, longitude: -74.0060})) / 1000, 2)
      """
    )
}
```

## Custom Resolvers

## EXERCISE: Exploring the @cypher Directive

## Check Your Understanding

## Summary
