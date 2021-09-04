# The Neo4j GraphQL OGM

## Lesson Overview

In this lesson we take a brief look at using the Neo4j GraphQL OGM for programmatically querying Neo4j. Then in the Exercise, you will use the the OGM.

## The Neo4j GraphQL OGM

The Neo4j GraphQL OGM is an object graph mapper that uses GraphQL type definitions to define models and exposes a programmatic way to query Neo4j without using Cypher.

### Using The Neo4j GraphQL OGM

Install the Neo4j GraphQL OGM package:

```sh
npm install @neo4j/graphql-ogm
```

Once installed, import the OGM into your project:

```js
const { OGM } = require("@neo4j/graphql-ogm")
```

The OGM is instantiated by passing the GraphQL type definitions and a Neo4j driver instance:

```js
const ogm = new OGM({ typeDefs, driver })
```

Model objects correspond to types declared in the GraphQL type definitions and expose `find`, `create`, `update`, and `delete` methods for working with the models.

Here we use the `Book.find` method to query for all books in the database and log to the console.

```js
const Book = ogm.model("Book")

const fetchBooks = async () => {
  const books = await Book.find()
  console.log(JSON.stringify(books, null, 2))
}

fetchBooks()
```

```sh
[
  {
    isbn: "1492047686",
    title: "Graph Algorithms",
    price: 37.48,
    description: "Practical Examples in Apache Spark and Neo4j",
  },
  {
    isbn: "1119387507",
    title: "Inspired",
    price: 21.38,
    description: "How to Create Tech Products Customers Love",
  },
  {
    isbn: "190962151X",
    title: "Ross Poldark",
    price: 15.52,
    description:
      "Ross Poldark is the first novel in Winston Graham's sweeping saga of Cornish life in the eighteenth century.",
  },
]
```

We can use the familiar filter arguments when querying using the Neo4j GraphQL OGM. To filter for a specific book by `isbn` value:

```js
const Book = ogm.model("Book")

const fetchBooks = async () => {
  const books = await Book.find({ where: { isbn: "1492047686" } })
  console.log(JSON.stringify(books, null, 2))
}

fetchBooks()
```

```sh
[
  {
    "isbn": "1492047686",
    "title": "Graph Algorithms",
    "price": 37.48,
    "description": "Practical Examples in Apache Spark and Neo4j"
  }
]
```

We can pass a selection set to traverse relationships defined in the GraphQL type definitions. Here we traverse to the book’s subjects using a selection set:

```js
const fetchBooks = async () => {
  const selectionSet = `
  {
    title
    description
    isbn
    price
    subjects {
      name
    }
  }
  `
  const books = await Book.find({ where: { isbn: "1492047686" }, selectionSet })
  console.log(JSON.stringify(books, null, 2))
}

fetchBooks()
```

```sh
[
  {
    "title": "Graph Algorithms",
    "description": "Practical Examples in Apache Spark and Neo4j",
    "isbn": "1492047686",
    "price": 37.48,
    "subjects": [
      {
        "name": "Non-fiction"
      },
      {
        "name": "Neo4j"
      },
      {
        "name": "Graph theory"
      }
    ]
  }
]
```

Create, update, and delete operations are also available using the Neo4j GraphQL OGM. Refer to the [Neo4j GraphQL OGM documentation](https://neo4j.com/docs/graphql-manual/current/ogm/) for more details.

## EXERCISE: Neo4j GraphQL OGM

Let’s consider a hands-on example of using the Neo4j GraphQL OGM outside of a GraphQL API. Launch [this Codesandbox](https://codesandbox.io/s/github/johnymontana/training-v3/tree/master/modules/graphql-apis/supplemental/code/05-graphql-apis-ogm/begin?file=/.env) which is a simple node script that runs a report to log all orders.

- Update the `.env` file with the connection credentials for your Neo4j Sandbox instance.
- Modify the script to include book titles and the price of each book in each order.

See [this Codesandbox](https://codesandbox.io/s/github/johnymontana/training-v3/tree/master/modules/graphql-apis/supplemental/code/05-graphql-apis-ogm/end?file=/.env) for the solution.

## Check Your Understanding

### Question 1

The Neo4j GraphQL OGM uses GraphQL type definitions to define models.

Is this statement true or false?

[X] True

[] False

### Question 2

When querying using the Neo4j GraphQL OGM a selection set can be provided to indicate specific fields to be returned.

Is this statement true or false?

[X] True

[] False

### Question 3

The Neo4j GraphQL OGM can be used outside of GraphQL APIs, such as in a REST API implementation.

Is this statement true or false?

[X] True

[] False

## Summary

In this lesson, we introduced the Neo4j GraphQL OGM for querying Neo4j programmatically.
