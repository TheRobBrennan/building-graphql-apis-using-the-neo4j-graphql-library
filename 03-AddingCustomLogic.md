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

## Custom Resolvers

## EXERCISE: Exploring the @cypher Directive

## Check Your Understanding

## Summary
