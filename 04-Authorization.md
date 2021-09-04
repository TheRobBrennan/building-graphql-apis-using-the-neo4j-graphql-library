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

## JSON Web Token (JWT)

## Adding Authorization Rules

## EXERCISE: Adding a Customer & Order

## Check Your Understanding

## Summary
