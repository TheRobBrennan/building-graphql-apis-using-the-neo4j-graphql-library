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

### Node And Object Fields

In addition to scalar fields we can also use `@cypher` directive fields on object and object array fields with Cypher queries that return nodes or objects. Let’s add a `recommended` field to the `Customer` type, returning books the customer might be interested in purchasing based on their order history and the order history of other customers in the graph.

Add this extension to the schema.graphql file:

```gql
# schema.graphql

extend type Customer {
  recommended: [Book]
    @cypher(
      statement: """
      MATCH (this)-[:PLACED]->(:Order)-[:CONTAINS]->(:Book)<-[:CONTAINS]-(:Order)<-[:PLACED]-(c:Customer)
      MATCH (c)-[:PLACED]->(:Order)-[:CONTAINS]->(rec:Book)
      WHERE NOT EXISTS((this)-[:PLACED]->(:Order)-[:CONTAINS]->(rec))
      RETURN rec
      """
    )
}
```

Now we can use this `recommended` field on the `Customer` type. Since `recommended` is an array of `Book` objects we need to select the nested fields we want to be returned - in this case the `title` field.

```gql
{
  customers {
    username
    recommended {
      title
    }
  }
}
```

```json
{
  "data": {
    "customers": [
      {
        "username": "EmilEifrem7474",
        "recommended": [
          {
            "title": "Inspired"
          },
          {
            "title": "Ross Poldark"
          }
        ]
      },
      {
        "username": "BookLover123",
        "recommended": []
      }
    ]
  }
}
```

In this case we recommend two books to Emil that he hasn’t purchased, however since BookLover123 has already purchased every book in our inventory we don’t have any recommendations for them!

Any field arguments declared on a GraphQL field with a Cypher directive are passed through to the Cypher query as Cypher parameters. Let’s say we want the client to be able to specify the number of recommendations returned. We’ll add a field argument `limit` to the `recommended` field and reference that in our Cypher query as a Cypher parameter.

Modify this extension in the schema.graphql file:

```gql
# schema.graphql

extend type Customer {
  recommended(limit: Int = 3): [Book]
    @cypher(
      statement: """
      MATCH (this)-[:PLACED]->(:Order)-[:CONTAINS]->(:Book)<-[:CONTAINS]-(:Order)<-[:PLACED]-(c:Customer)
      MATCH (c)-[:PLACED]->(:Order)-[:CONTAINS]->(rec:Book)
      WHERE NOT EXISTS((this)-[:PLACED]->(:Order)-[:CONTAINS]->(rec))
      RETURN rec LIMIT $limit
      """
    )
}
```

We set a default value of 3 for this `limit` argument so that if the value isn’t specified the `limit` Cypher parameter will still be passed to the Cypher query with a value of 3. The client can now specify the number of recommended books to return:

```gql
{
  customers {
    username
    recommended(limit: 1) {
      title
    }
  }
}
```

We can also return a map from our Cypher query when using the `@cypher` directive on an object or object array GraphQL field. This is useful when we have multiple computed values we want to return or for returning data from an external data layer. Let’s add weather data for the order addresses so our delivery drivers know what sort of conditions to expect. We’ll query an external API to fetch this data using the [apoc.load.json](https://neo4j.com/labs/apoc/4.2/import/load-json/) procedure.

First, we’ll add a type to the GraphQL type definitions to represent this object (`Weather`), then we’ll use the `apoc.load.json` procedure to fetch data from an external API and return the current conditions, returning a map from our Cypher query that matches the shape of the `Weather` type.

Add these types and extensions to the schema.graphql file:

```gql
# schema.graphql

type Weather {
  temperature: Int
  windSpeed: Int
  windDirection: Int
  precipitation: String
  summary: String
}

extend type Address {
  currentWeather: Weather
    @cypher(
      statement: """
      WITH 'https://www.7timer.info/bin/civil.php' AS baseURL, this
      CALL apoc.load.json(
          baseURL + '?lon=' + this.location.longitude + '&lat=' + this.location.latitude + '&ac=0&unit=metric&output=json')
          YIELD value WITH value.dataseries[0] as weather
          RETURN {
              temperature: weather.temp2m,
              windSpeed: weather.wind10m.speed,
              windDirection: weather.wind10m.direction,
              precipitation: weather.prec_type,
              summary: weather.weather} AS conditions
      """
    )
}
```

Now we can include the `currentWeather` field on the `Address` type in our GraphQL queries:

```gql
{
  orders {
    shipTo {
      address
      currentWeather {
        temperature
        precipitation
        windSpeed
        windDirection
        summary
      }
    }
  }
}
```

```json
{
  "data": {
    "orders": [
      {
        "shipTo": {
          "address": "111 E 5th Ave, San Mateo, CA 94401",
          "currentWeather": {
            "temperature": 12,
            "precipitation": "none",
            "windSpeed": 2,
            "windDirection": "SW",
            "summary": "clearday"
          }
        }
      },
      {
        "shipTo": {
          "address": "Nordenskiöldsgatan 24, 211 19 Malmö, Sweden",
          "currentWeather": {
            "temperature": 21,
            "precipitation": "none",
            "windSpeed": 3,
            "windDirection": "NW",
            "summary": "clearday"
          }
        }
      }
    ]
  }
}
```

### Custom Query Field

We can use the `@cypher` directive on Query fields to compliment the auto-generated Query fields provided by the Neo4j GraphQL Library. Perhaps we want to leverage a [full-text index](https://neo4j.com/docs/cypher-manual/current/administration/indexes-for-full-text-search/) for fuzzy matching for book searches?

First, in Neo4j Browser, create the full-text index:

```cypher
CALL db.index.fulltext.createNodeIndex("bookIndex", ["Book"],["title", "description"])
```

To search this full-text index we use the `db.index.fulltext.queryNodes` procedure:

```cypher
CALL db.index.fulltext.queryNodes("bookIndex", "garph~")
```

Neo4j full-text indexes use Apache Lucene query syntax - the `~` indicates we want to use "fuzzy matching" taking into account slight misspellings.

Next, add a `bookSearch` field to the Query type in our GraphQL type definitions which requires a `searchString` argument that becomes the full-text search term:

```gql
# schema.graphql

type Query {
  bookSearch(searchString: String!): [Book]
    @cypher(
      statement: """
      CALL db.index.fulltext.queryNodes('bookIndex', $searchString+'~')
      YIELD node RETURN node
      """
    )
}
```

And we now have a new entry-point to our GraphQL API allowing for full-text search of book titles and descriptions:

```gql
{
  bookSearch(searchString: "garph") {
    title
    description
  }
}
```

```json
{
  "data": {
    "bookSearch": [
      {
        "title": "Graph Algorithms",
        "description": "Practical Examples in Apache Spark and Neo4j"
      }
    ]
  }
}
```

### Custom Mutation Field

Similar to adding Query fields, we can use `@cypher` schema directives to add new Mutation fields.

**This is useful in cases where we have specific logic we’d like to take into account when creating or updating data.**

Here we make use of the **MERGE** [Cypher clause](https://neo4j.com/docs/cypher-manual/current/clauses/merge/) to avoid creating duplicate `Subject` nodes and connecting them to books.

```gql
# schema.graphql

type Mutation {
  mergeBookSubjects(subject: String!, bookTitles: [String!]!): Subject
    @cypher(
      statement: """
      MERGE (s:Subject {name: $subject})
      WITH s
      UNWIND $bookTitles AS bookTitle
      MATCH (t:Book {title: bookTitle})
      MERGE (t)-[:ABOUT]->(s)
      RETURN s
      """
    )
}
```

Now perform the update to the graph:

```gql
mutation {
  mergeBookSubjects(
    subject: "Non-fiction"
    bookTitles: ["Graph Algorithms", "Inspired"]
  ) {
    name
  }
}
```

## Custom Resolvers

## EXERCISE: Exploring the @cypher Directive

## Check Your Understanding

## Summary
