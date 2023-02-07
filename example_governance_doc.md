# ACME Graph Integration Guidelines

## Table of Contents
- [ACME Graph Integration Guidelines](#acme-graph-integration-guidelines)
  - [Table of Contents](#table-of-contents)
  - [Summary](#summary)
  - [Getting Help](#getting-help)
  - [Background](#background)
  - [Getting Started](#getting-started)
  - [Guidelines](#guidelines)
    - [Schema Design Guidelines](#schema-design-guidelines)
      - [Naming Requirements](#naming-requirements)
      - [Query and Mutation Naming](#query-and-mutation-naming)
      - [Pagination](#pagination)
      - [Errors](#errors)
    - [Security](#security)
    - [For Client Developers/Consumers](#for-client-developersconsumers)
  - [Procedures](#procedures)
    - [Schema Reviews](#schema-reviews)
      - [When to Engage](#when-to-engage)
      - [How to Engage](#how-to-engage)
      - [Next Steps](#next-steps)
    - [Deprecations](#deprecations)


##  Summary

This document serves to assist both new and existing developers and consumers of the data graph to get started, either via additions or changes, and provide requirements and expectations for each. 

The graph is currently primarily maintained by the core graph team (#team-explosion in Slack). Beyond the core team, the following are the current stakeholders with the graph: 

* Client Teams:
  * Mobile
  * Web 
  * Embedded
* Subgraph teams

These stakeholders comprise the schema working group, or SWG. 

## Getting Help

If you're not familiar with GraphQL, please start by looking at [Apollo's Odyssey tutorials](https://apollographql.com/tutorials/) to get acquainted with the technology.

To request help with the ACME graph, please reach out in #graphql in Slack. 

## Background

As our company has grown over the past year, we realized that our existing REST API solution was failing to scale as we began to spin up new client applications and data requirements quick changed for each new addition. 

As a result, we landed on using GraphQL to power our new API in order to quickly add and change data as requirements changed. Additionally, GraphQL offered us an opportunity to enable clients to easily access the data they cared about. 

In order to accelerate development speed, we landed on using Apollo's Federation specification which enables portions of the overall graph to be known as "subgraphs." The overall graph is known as the supergraph. 

## Getting Started

If you are looking to expose your service as a subgraph, you will need to engage with the core graph team as early as possible over Slack in the #graphql channel and ideally providing the team with the following information: 
  * Context for why you're wanting to expose your service
    * For example, you are supporting a new user interface element in the Trap mobile applications that allows users to purchase booby-traps
  * The Okta team name to be added to Apollo Studio
    * If this does not exist, please reach out to IT to have this created with all team members

Once the team has this information, the core graph team can start assisting with the initial schema. This process can take some time, so if there is urgency in shipping the new schema, please inform the team during the initial contact. 

During this process, you will go through a [schema review](#schema-reviews) process, which is covered below with the assistance of the core graph team helping shape the initial schema.

## Guidelines

The core graph team has an established set of guidelines for contributing to the ACME graph, covered below. This is _not_ a list of schema design choices and information, but rather established standards the graph (schema or otherwise) is expected to follow. 

### Schema Design Guidelines

GraphQL's strengths lie in its ability to be descriptive and easy to understand. To that, we have a list of requirements to encourage a readable schema for consumers and developers alike. 

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

Beyond type and value names, we also enforce naming requirements on query and operations. For both: 

* We discourage the use of REST-style prefixes (e.g. GET, POST, PATCH)
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

Given that example, we could do the following: 

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

As with REST, ensuring a secure graph is paramount. To that, we recommend: 

* Authorizing users at your subgraph rather than ensuring this is done at the edge
* Sanitizing all input values, especially those using custom scalars


### For Client Developers/Consumers

As a consumer of the graph, we have a few guidelines: 

* Ensure the client identifies itself using a client name and version by passing an `ACME-Client-Version` and `ACME-Client-Name` on all requests
  * This assists the core team with notifications and helps subgraph owners understand usage by your application(s)
* Ensure the client uses APQs whenever possible; APQs offer performance benefits and helps clients


## Procedures

Lastly, the core graph team has a few procedures needed for the smooth operation of the graph.

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

With that, the SWG strongly recommends schema reviews whenever possibly by doing async reviews on a shared Google Doc. A template for such is [here](). This document will ask for necessary context, such as the currently proposed schema, alternative schemas, and any UI mocks if possible.

To engage with the SWG, please reach out to #graphql-swg on Slack along with the above doc. 

#### Next Steps

Once the SWG has received the document, they will begin to provide comments and suggestions within the doc. 

For new subgraphs, there will also be a separate meeting to discuss the proposal in-depth and provide further feedback. 

Once both the SWG and subgraph team(s) agree on the change, the proposal will be considered "approved" and will move to be implemented into the supergraph. 

### Deprecations

Once a field no longer is fit for purpose, either through product changes (e.g. removal of a feature) or required changes that are not backwards-compatible, it may become necessary to deprecate it. 

For subgraph teams needing to deprecate a field, the core graph team requires (barring special exemption): 

* Marking the field as `@deprecated` within your subgraph schema
* Using Apollo Studio, seeing current clients and providing them with information about the upcoming deprecation along with timetables
  * The core graph team strongly encourages at least 3 months of notice
* Once 3 months has passed, or until traffic has reached 0 traffic for the last 3 minor versions (whichever is less), the field can safely be removed
