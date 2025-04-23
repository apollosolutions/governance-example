# ACME Graph Integration Guidelines

## Table of Contents
- [ACME Graph Integration Guidelines](#acme-graph-integration-guidelines)
  - [Table of Contents](#table-of-contents)
  - [Summary](#summary)
  - [Getting Help](#getting-help)
  - [Background](#background)
  - [Procedures](#procedures)
    - [Getting Started](#getting-started)
    - [Schema Reviews](#schema-reviews)
      - [When to Engage](#when-to-engage)
      - [How to Engage](#how-to-engage)
      - [Next Steps](#next-steps)
    - [Deprecations](#deprecations)
  - [Subgraph Author Guidelines](#subgraph-author-guidelines)
    - [Schema Design Guidelines](#schema-design-guidelines)
      - [Naming Requirements](#naming-requirements)
      - [Query and Mutation Naming](#query-and-mutation-naming)
      - [Pagination](#pagination)
      - [Errors](#errors)
    - [Security](#security)
  - [Consumer Guidelines](#consumer-guidelines)
    - [Client Identification](#client-identification)
    - [Operation Naming](#operation-naming)
    - [Operation Variables](#operation-variables)
    - [Use of Automatic Persisted Queries (APQ)](#use-of-automatic-persisted-queries-apq)


## Summary

This document serves to assist both new and existing developers and consumers of the data graph to get started, either via additions or changes, and provide requirements and expectations for each. 

The graph is currently primarily maintained by the core graph team (#team-explosion in Slack). Beyond the core team, the following are the current stakeholders with the graph: 

* Client Teams:
  * Mobile
  * Web 
  * Embedded
* Subgraph teams

These stakeholders comprise the Schema Working Group, or SWG. 

## Getting Help

To request help with the ACME graph, please reach out in #graphql in Slack. 

The following is a proposed learning path for contributors not familiar with Apollo GraphQL.

### Learning Paths

Apollo offers two certification courses through their Odyssey platform: [Associate](https://www.apollographql.com/tutorials/certifications/apollo-graph-associate) and [Professional](https://www.apollographql.com/tutorials/certifications/apollo-graph-professional). Contributors are recommended to complete at least the Associate certification in order to understand the basics of GraphQL and Federation.

In addition to the certifications, Apollo also offers smaller modules for focusing on certain areas:

- [A bite-sized course on what federation is](https://www.apollographql.com/tutorials/introduction-to-federation)
- [Basics of GraphOS with a strong focus on Studio usage](https://www.apollographql.com/tutorials/getting-started-with-graphos)
- [Beta course for safe API delivery into GraphOS](https://www.apollographql.com/tutorials/graphos-shipping-supergraph)
- [Beta mini-course on growing your supergraph](https://www.apollographql.com/tutorials/graphos-growing-supergraph)
- [Beta course around leveraging Apollo Client on a React front-end](https://www.apollographql.com/tutorials/client-side-graphql-react)

### Enterprise Modules

Enterprise modules require ACME contributors to log into Odyssey using their SSO credentials
- [Effective graph stewardship](https://www.apollographql.com/tutorials/effective-stewardship)
- [Observability in GraphOS](https://www.apollographql.com/tutorials/supergraph-observability)
- [Testing for both clients and the servers](https://www.apollographql.com/tutorials/testing)

### Additional Resources

- Another great reference for GraphQL is the official [GraphQL website](https://graphql.org/learn/)
- Teams are also encouraged to run through [Supergraph Architecture Framework (SAF) assessment](https://saf.apollographql.com/) to better understand their GraphQL posture across several pillars like Reliability, Security, Operational Excellence, Performance and Developer Experience

## Background

As ACME has grown over the past year, we realized that our existing REST API solution was failing to scale as we began to spin up new client applications and data requirements quickly changed for each new addition. 

As a result, we decided on using GraphQL to power our new API in order to quickly add and change data as requirements changed. Additionally, GraphQL offered us an opportunity to enable clients to easily access the data they cared about. 

In order to accelerate development speed, we landed on using [Apollo's Federation](https://www.apollographql.com/docs/federation) specification which enables portions of the overall graph to be known as "subgraphs." The overall graph is known as the supergraph. These principles are outlined in our Mission and Vision statement below:

### Mission and Vision

ACME is looking to establish a supergraph that is the preferred development platform for all services.

The goal of the ACME supergraph is to be the aggregation layer on top of the a domain-based services architecture, defining and mapping relationships in the data across bounded contexts. Key to our success will be to make the supergraph the preferred mode of development over REST APIs as measured by the percentage of requests going to our GraphQL instance vs their REST counterparts. 

The surface of the graph will span each of our current domain services, beginning with the most used domains first.

The success outcomes for the mission established by ACME are as follows:

1. We will simplify the API landscape by making the GraphQL supergraph the preferred platform over continued development of RESTful APIs. 
2. Our focus will be on building reusable Domain APIs to focus on reusability, maintainability and scalability while minimizing the timelines required to update business logic.
3. We will also establish a "GraphQL Shepherds" guild to play a key role in training and up-leveling GraphQL expertise across the dev teams.
4. This, in turn will help to spread GraphQL expertise across the organization

## Procedures

The core graph team has a few procedures needed for the smooth operation of the graph.

### Getting Started

If you are looking to expose your service as a subgraph, you will need to engage with the core graph team as early as possible over Slack in the #graphql channel and ideally providing the team with the following information: 

  * Context for why you're wanting to expose your service to clients
    * For example, you are supporting a new user interface element in the Trap mobile applications that allows users to purchase booby-traps
  * The Okta team name to be added to Apollo Studio
    * If this does not exist, please reach out to IT to have this created with all team members

Once the team has this information, the core graph team can start assisting with the initial schema. This process can take some time, so if there is urgency in shipping the new schema, please inform the team during the initial contact. 

During this process, you will go through a [schema review](#schema-reviews) process, which is covered below with the assistance of the core graph team helping shape the initial schema.

### Schema Reviews

Schema additions and changes are always a time where it can be helpful to obtain feedback from all parts of the company, from clients to other subgraph teams. 

#### When to Engage

You should engage the SWG as early as possible if any of these match the schema changes or additions being proposed:

* New subgraph team onboarding
* Large or interesting schema additions
* Deprecations of fields or subgraphs ([see the below deprecations section](#deprecations))
* If a review would be helpful

#### How to Engage

While schema reviews are helpful, the core graph team also realizes that meeting after meeting isn't conducive to productivity. 

With that, the SWG strongly recommends schema reviews using Apollo's [schema proposals](https://www.apollographql.com/docs/graphos/platform/schema-management/proposals) feature. Managing schema changes using schema proposals enables teams to propose schema changes that are validated against our development variant, running composition, linting and schema checks on every change.

To engage with the SWG, please reach out to #graphql-swg on Slack along with a link to the appropriate schema proposal. 

#### Next Steps

Once the SWG has received the proposal, they will begin to provide comments and suggestions within the tool. 

For new subgraphs, there will also be a separate meeting to discuss the proposal in-depth and provide further feedback. 

Once both the SWG and subgraph team(s) agree on the change, the proposal will be considered "approved" and will move to be implemented into the supergraph.

Subgraph teams can pull down the approved proposal using `rover` and once published to the supergraph the proposal will be marked as "implemented"

### Deprecations

Once a field no longer is fit for purpose, either through product changes (e.g. removal of a feature) or required changes that are not backwards-compatible, it may become necessary to deprecate it. 

For subgraph teams needing to deprecate a field, the core graph team requires (barring special exemption): 

* Marking the field as `@deprecated` within your subgraph schema
* Using Apollo Studio, seeing current clients and providing them with information about the upcoming deprecation along with timetables
  * The core graph team strongly encourages at least 3 months of notice
* Once 3 months has passed, or until traffic has reached 0 traffic for the last 3 minor versions (whichever is less), the field can safely be removed

## Subgraph Author Guidelines

The core graph team has an established set of guidelines for contributing to the ACME graph, covered below. 

### Schema Design Guidelines

GraphQL's strengths lie in its ability to be descriptive and easy to understand. To that, we have a list of requirements to encourage a readable schema for consumers and developers alike. 

[Apollo also provides recommendations which we adhere to for the most part](https://www.apollographql.com/docs/technotes/tags/schema-design), but below are specific conventions we've landed on. We do recommend reading through Apollo's recommendations as they can be a useful tool to design better schemas. Furthermore, these conventions are validated using Apollo's built-in [schema linter](https://www.apollographql.com/docs/graphos/platform/schema-management/linting) which is run with every schema check.

A full list of schema linter rules can be found [here](https://www.apollographql.com/docs/graphos/platform/schema-management/linting/rules)

#### Naming Requirements

We require GraphQL the following naming conventions: 

* Object names to be Pascal-case and singular
* Fields to be camel-case
* Enum values to be screaming snake-case

```gql
# bad type name (snake cased)
type user_info {
    id: ID!
}

# bad type name (plural)
type Users {
    id: ID!
}

# good type name
type UserInfo {
    first_name: String! # bad field name
    firstName: String! # good field name
}

# bad enum name
enum userType {
    not_screaming # bad value name
    IS_SCREAMING # good value name
}
```

#### Query and Mutation Naming

Beyond type and value names, we also enforce naming requirements on operations. 

* We discourage the use of REST-style prefixes (e.g. `GET`, `POST`, `PATCH`)
* Use single-purpose queries and mutations whenever possible

Examples:

```gql
type Query {
    getProducts: [Product!]! # bad query name (use of GET prefix)
    products: [Product!]! # good query name
    user(id: ID, email: String): User # bad query (multipurpose query)
    userById(id: ID!): User # good query 
}
```

#### Pagination

Service owners should implement pagination in their fields whenever possible. We currently don't leverage any standardized model, however. Adding in pagination after implementation can be complex, so investing up-front will pay dividends down the road. 

For most use-cases, the following works:

```gql 
type Query {
    products(limit: Int = 100, cursor: String): [Product!]! # if using cursor-based pagination
    products(limit: Int = 100, offset: Int = 0): [Product!]! # if using offset-based pagination
}
```

#### Errors

When designing your schema, consider how clients will access the data and any expected error states that would need to be handled as UI and codify them in your schema. 

For example, if a customer attempted to purchase a product but ran into an error with the information they provided, we should ideally return back what they needed to change or fix within the UI. 

For example, we could do the following: 

```gql 
type Mutation {
    submitPayment(paymentInformation: PaymentInformation!): PaymentResult!
}

union PaymentResult = PaymentSuccess | PaymentError

type PaymentSuccess {
    id: ID!
    confirmationNumber: String!
}

type PaymentError {
    errorCode: String!
    message: String!
    invalidFields: [String!]!
}
```

This would result in a mutation that would either return a success state (happy path) or error state (unhappy path), and clients could properly handle either case vs. digging through the `errors` key in the response. 

### Security

As with REST, ensuring a secure API is paramount. To that, we recommend: 

* Authorizing users at your subgraph rather than ensuring this is done at the edge
* Sanitizing all input values, especially those using custom scalars


## Consumer Guidelines

As a consumer of the graph, we have a few requirements for utilizing the ACME data graph. 

### Client Identification

Clients must identify themselves by name and version, ideally matching release versions. Name and version can be set in the client initialization or via an HTTP interceptor, with documentation below. We currently identify name and version by `apollographql-client-name` and `apollographql-client-version` respectively. Requests without these headers will be rejected.

* [Web: https://www.apollographql.com/docs/react/networking/basic-http-networking#customizing-request-headers](https://www.apollographql.com/docs/react/networking/basic-http-networking#customizing-request-headers)
* [Android: https://www.apollographql.com/docs/kotlin/advanced/interceptors-http](https://www.apollographql.com/docs/kotlin/advanced/interceptors-http)
* [iOS: https://www.apollographql.com/docs/ios/networking/request-pipeline](https://www.apollographql.com/docs/ios/networking/request-pipeline)

### Operation Naming

Operations should have a self-descriptive name about the location and purpose of the request. For example:

```gql
# bad query (unnamed)
query($id: ID!) {
  user(id:$id) {
    id
  }
}

# bad query (poor name)
query User($id: ID!) {
  user(id:$id) {
    id
  }
}

# good query
query UserDetailsPage_UserQuery($id: ID!) {
  user(id:$id) {
    id
  }
}
```

### Operation Variables

Operations should use GraphQL variables instead of literals for input values whenever possible. For example: 

```gql
# bad query (literal input value)
query UserDetailsPage_UserQuery{
  user(id: "1") {
    id
  }
}

# good query
query UserDetailsPage_UserQuery($id: ID!) {
  user(id:$id) {
    id
  }
}
```

### Use of Automatic Persisted Queries (APQ)

[APQs offer a way to improve performance by not sending request bodies to the server, and instead a hash](https://www.apollographql.com/docs/apollo-server/performance/apq). As a result, we require that clients opt into using APQs as they can assist with both client and server performance. 

To set up APQs, see relevant documentation: 

* [React: https://www.apollographql.com/docs/apollo-server/performance/apq/#apollo-client-setup](https://www.apollographql.com/docs/apollo-server/performance/apq/#apollo-client-setup)
* [Android: https://www.apollographql.com/docs/kotlin/advanced/persisted-queries/#automatic-persisted-queries](https://www.apollographql.com/docs/kotlin/advanced/persisted-queries/#automatic-persisted-queries)
* [iOS: https://www.apollographql.com/docs/ios/fetching/apqs/](https://www.apollographql.com/docs/ios/fetching/apqs/)

