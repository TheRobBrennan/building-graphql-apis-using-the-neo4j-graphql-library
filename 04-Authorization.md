# Authorization

## Lesson Overview

In this lesson we introduce authorization, protecting objects in our API. We cover how to make data available only to authenticated users and how to restrict data access so that customers can only see or modify their own data.

## The @auth Directive

The Neo4j GraphQL Library provides an `@auth` GraphQL schema directive that enables us to attach authorization rules to our GraphQL type definitions. The `@auth` directive uses JSON Web Tokens (JWTs) for authentication. Authenticated requests to the GraphQL API will include an `authorization` header with a Bearer token attached. For example:

```http
POST / HTTP/1.1
authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJyb2xlcyI6WyJ1c2VyX2FkbWluIiwicG9zdF9hZG1pbiIsImdyb3VwX2FkbWluIl19.IY0LWqgHcjEtOsOw60mqKazhuRFKroSXFQkpCtWpgQI
content-type: application/json
```

Refer to the [Auth section of the Neo4j GraphQL Library documentation](https://neo4j.com/docs/graphql-manual/current/auth/) for full information.

## Setup

To enable authorization with the Neo4j GraphQL Library we need to specify the JWT signing secret used to decode and validate tokens. For this lesson we will use the following JWT secret to validate our tokens:

```sh
dFt8QaYykR6PauvxcyKVXKauxvQuWQTc
```

When instantiating `Neo4jGraphQL` pass a config object that includes the JWT secret. For example:

```js
const neoSchema = new Neo4jGraphQL({
  typeDefs,
  config: {
    jwt: {
      secret: "dFt8QaYykR6PauvxcyKVXKauxvQuWQTc",
    },
  },
})
```

This step has been completed in the initial Codesandbox we’ll use for this lesson. Open [the Codesandbox using this link](https://codesandbox.io/s/github/johnymontana/training-v3/tree/master/modules/graphql-apis/supplemental/code/04-graphql-apis-auth/begin?file=/.env), again forking and editing the `.env` file to add the connection credentials for your Neo4j Sandbox instance. You’ll notice we’ve included an additional environment variable to keep track of our JWT secret.

## JSON Web Token (JWT)

JWTs are a standard for representing and cryptographically verifying claims securely and are commonly used for authentication and authorization. Implementing a sign-in flow and generating JWTs is beyond the scope of this course so we will use a few static JWTs for testing our authorization rules. For an example of a sign-up/sign-in flow using GraphQL mutations and the Neo4j GraphQL Library see the `neo-push` [example application](https://github.com/neo4j/graphql/blob/master/examples/neo-push/server/src/gql/User.ts) in the `neo4j/graphql` repository.

### Example JWTs

We will use the following JWTs to test the authorization rules we’ll be adding to our GraphQL API. These tokens were generated using the JWT secret signing key above and can be validated using the same secret.

#### Token For Customer EmilEifrem7474

This token is the token we will use to make authenticated requests on behalf of the customer "EmilEifrem7474":

```sh
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJFbWlsRWlmcmVtNzQ3NCIsInJvbGVzIjpbImN1c3RvbWVyIl0sImlhdCI6MTUxNjIzOTAyMn0.YwftAMDTw6GqmYOFLGHC_f6UiUhfrJAGkZGfrGmiQ2U
```

The token’s payload includes the following claims:

```json
{
  "sub": "EmilEifrem7474",
  "roles": ["customer"],
  "iat": 1516239022
}
```

#### Admin user token

This token is used to make authenticated requests to the GraphQL API as an "admin" user:

```sh
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJCb2JMb2JsYXc3Njg3Iiwicm9sZXMiOlsiYWRtaW4iXSwiaWF0IjoxNTE2MjM5MDIyfQ.f2GKIu31gz39fMJwj5_byFCMDPDy3ncdWOIhhqcwBxk
```

It includes the following claims:

```json
{
  "sub": "BobLoblaw7687",
  "roles": ["admin"],
  "iat": 1516239022
}
```

We can use the online tool at [jwt.io](https://jwt.io/) to encode/decode and validate tokens. Try pasting one of the above tokens into this tool to view the token’s payload.

![https://neo4j.com/graphacademy/training-graphql-apis/_images/04jwtio.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/04jwtio.png)

## Adding Authorization Rules

Now that we have our sample tokens we’re ready to start adding authorization rules using the `@auth` schema directive in our GraphQL type definitions.

### isAuthenticated

The `isAuthenticated` rule is the simplest authorization rule we can add. It means that any GraphQL operation that accesses a field or object protected by the `isAuthenticated` rule must have a valid JWT in the request header.

Let’s make use of the `isAuthenticated` authorization rule in our bookstore GraphQL API to protect the `Subject` type. Let’s say we want to make returning a book’s subjects a "premium" feature to encourage users to sign-up for our application. To do this we’ll make the following addition to our GraphQL type definitions, extending the `Subject` type:

```gql
# schema.graphql

extend type Subject @auth(rules: [{ isAuthenticated: true }])
```

Now any request that accesses the `Subject` type must include a valid signed JWT or an error will be returned.

Here we query as usual without including a JWT in the request header:

![https://neo4j.com/graphacademy/training-graphql-apis/_images/04playgrounderror.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/04playgrounderror.png)

In GraphQL Playground to add a request header we add the following to the "HTTP Headers" box in the lower left window. Now that our request includes a valid token, the data is returned as expected:

![https://neo4j.com/graphacademy/training-graphql-apis/_images/04playgroundnoerror.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/04playgroundnoerror.png)

### Roles

[Roles](https://neo4j.com/docs/graphql-manual/current/auth/authorization/roles/) are the next type of authorization rule that we will explore. A JWT payload can include an array of "roles" that describe the permissions associated with the token. For example, our example token for the "admin" user includes the following payload:

```json
{
  "sub": "BobLoblaw7687",
  "roles": ["admin"],
  "iat": 1516239022
}
```

which means that this user has the "admin" role. Let’s add a rule to our GraphQL type definitions that in order to create, update, or delete books, the user must have the "admin" role.

```gql
# schema.graphql

extend type Book
  @auth(rules: [{ operations: [CREATE, UPDATE, DELETE], roles: ["admin"] }])
```

Note that we’ve included the `operations` array to specify this rule only applies to `CREATE`, `UPDATE`, and `DELETE` operations - all users will still be able to read book objects, but if any request tries to create or update a book the operation will fail unless a valid "admin" role is included in the token.

Try to execute the following mutation using our example tokens. What error do you get when not using the admin token? What is the result when using the admin token?

```gql
mutation {
  createBooks(
    input: {
      title: "Graph Databases"
      isbn: "1491930896"
      subjects: { connect: { where: { name: "Neo4j" } } }
    }
  ) {
    books {
      title
      subjects {
        name
      }
    }
  }
}
```

### Allow

A customer must not be able to view orders placed by other customers. Adding an [Allow](https://neo4j.com/docs/graphql-manual/current/auth/authorization/allow/) rule will allow us to protect orders from other nosy customers.

Here we add a rule to the `Order` type that a customer’s "sub" (the subject) claim in the JWT must match the username of the customer who placed the order.

```gql
# schema.graphql

extend type Order
  @auth(rules: [{ allow: { customer: { username: "$jwt.sub" } } }])
```

If we try to access orders that aren’t ours we’ll see a "Forbidden" error:

![https://neo4j.com/graphacademy/training-graphql-apis/_images/04forbidden.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/04forbidden.png)

but if we filter only for the orders placed by the customer we have access:

![https://neo4j.com/graphacademy/training-graphql-apis/_images/04notforbidden.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/04notforbidden.png)

Of course we will also allow admins to have access to orders, so let’s update the rule to also grant access to any requests with the "admin" role:

```gql
# schema.graphql

extend type Order
  @auth(
    rules: [
      { allow: { customer: { username: "$jwt.sub" } } }
      { roles: ["admin"] }
    ]
  )
```

### Where

In the previous example the client was required to filter for orders that the customer had placed. We don’t always want to expect the client to include this filtering logic in the GraphQL query. In some cases we simply want to return whatever data the currently authenticated user has access to. For these cases we can use a [Where](https://neo4j.com/docs/graphql-manual/current/auth/authorization/where/) authorization rule to apply a filter to the generated database queries - ensuring only the data the user has access to is returned.

We want a user to only be able to view their own customer information. Here we add a rule to the `Customer` type that will apply a filter any time the customer type is accessed that filters for the currently authenticated customer by adding a predicate that matches the `username` property to the sub claim in the JWT.

```gql
# schema.graphql

extend type Customer @auth(rules: [{ where: { username: "$jwt.sub" } }])
```

![https://neo4j.com/graphacademy/training-graphql-apis/_images/04customer.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/04customer.png)

**Note that our query doesn’t specify which customer to return - we’re requesting all customers - but we only get back the customer that we have access to.**

### Bind

The final type of authorization rule that we will explore is the [Bind](https://neo4j.com/docs/graphql-manual/current/auth/authorization/bind/) rule.

**Bind allows us to specify connections that must exist in the graph when creating or updating data based on claims in the JWT.**

We want to add a rule that when creating a review, the review node is connected to the currently authenticated customer - we don’t want customers to be writing reviews on behalf of other users!

**This rule means the username of the author of a review must match the sub claim in the JWT when creating or updating reviews:**

```gql
# schema.graphql

extend type Review
  @auth(
    rules: [
      {
        operations: [CREATE, UPDATE]
        bind: { author: { username: "$jwt.sub" } }
      }
    ]
  )
```

If a customer tries to create a review and connect it to a customer other than themselves the mutation will return an error. Try running this mutation using our example JWT. Does it work? Can you tell why?

```gql
mutation {
  createReviews(
    input: {
      rating: 1
      text: "Borrring"
      book: { connect: { where: { title: "Ross Poldark" } } }
      author: { connect: { where: { username: "BookLover123" } } }
    }
  ) {
    reviews {
      text
      rating
      book {
        title
      }
    }
  }
}
```

### Auth In Cypher Directive Fields

There are two ways to make use of authorization features when using the `@cypher` schema directive:

1. Apply the authorization rules `isAuthenticated` and `roles` using the `@auth` directive.
2. Reference the JWT payload values in the Cypher query attached to a `@cypher` schema directive.

Let’s make use of both of those aspects by adding a Query field that returns personalized recommendations for a customer. In our Cypher query we’ll have access to a `$auth.jwt` parameter that represents the payload of the JWT. We’ll use that value to look up the currently authenticated customer by username, then traverse the graph to find relevant recommendations based on their purchase history. We’ll also include the `isAuthenticated` rule since we only want authenticated customers to use this Query field.

```gql
# schema.graphql

extend type Query {
  booksForCurrentUser: [Book]
    @auth(rules: [{ isAuthenticated: true }])
    @cypher(
      statement: """
      MATCH (c:Customer {username: $auth.jwt.sub})-[:PLACED]->(:Order)-[:CONTAINS]->(b:Book)
      MATCH (b)-[:ABOUT]->(s:Subject)<-[:ABOUT]-(rec:Book)
      WITH rec, COUNT(*) AS score ORDER BY score DESC
      RETURN rec
      """
    )
}
```

Try running this GraphQL query. What results do you get?

```gql
{
  booksForCurrentUser {
    title
  }
}
```

![https://neo4j.com/graphacademy/training-graphql-apis/_images/04recommended.png](https://neo4j.com/graphacademy/training-graphql-apis/_images/04recommended.png)

## EXERCISE: Adding a Customer & Order

## Check Your Understanding

## Summary
