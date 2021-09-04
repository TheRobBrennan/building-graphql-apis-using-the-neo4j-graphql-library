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

## Adding Authorization Rules

## EXERCISE: Adding a Customer & Order

## Check Your Understanding

## Summary
