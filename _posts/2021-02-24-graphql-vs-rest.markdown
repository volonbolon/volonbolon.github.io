---
layout: post
title: "GraphQL vs REST"
date: 2021-02-24 17:50:44 GMT
tags: API Architecture
description:
---
Ladies and gentlemen, we have a new contender in the ring. The well-established API champion has been challenged by GraphQL. 

Essentially, GraphQL is a query language for APIs. Instead of exposing different endpoints with different data, we let the client specify what's actually required. The client's query is used to build up a graph, where the objects are the nodes, and the relationship between them are the edges. 

Because the client is writing the query, we are not risking over fetching, potentially saving trips to the backend (how many times we need to perform more than one query to retrieve the information required for a single view?), and reducing bandwidth. Even further, with GraphQL we can ask for specific fields on each object. No need to retrieve the whole entity when we just want the name. Conversely, we are losing REST simplicity, where the endpoints are mapping different datasets. 

In GraphQL we just send a single query to the GraphQL server that includes the concrete data requirements. The server then responds with a JSON object where these requirements are fulfilled.

```
{
     post(id: 1) {
        title
        user {
            name
            email
            courses {
                title
            }
        }
     }
}
```

The REST mapping between datasets and endpoints is modeled, most of the time, on the data required for a given view. We all danced that waltz, trying to balance what we need for a screen with what we can include in a dataset. That's fine, but it also introduces some inertia in the development. We might consider if it is worth changing the view when we also need to version the API. 

With GraphQL, this problem is solved. Thanks to the flexible nature of GraphQL, changes on the client-side can be made without any extra work on the server. Since clients can specify their exact data requirements, no backend engineer needs to make adjustments when the design and data need on the frontend change.

### Schema and types

All the types that are exposed through an API are defined in a schema. The schema is the contract between API and clients, and defines what can be accessed, and how. 

We can, for instance, define a type `Person`. 

```
type Person {
	id: Int
  name: String!
  age: Int
}
```

A simple query of all the person names would be something like:

```
{
  allPersons {
    name
  }
}
```

But we can also parametrize the queries, for instance, we can simply slice the response with 

```
{
  allPersons(first:3) {
    name
  }
}
```

Or 

```
{
  allPersons(last: 3) {
    name
  }
}
```


For pagination, we can set an offset, obviously, but we can even include some sort of id:

```
allPersons(first:2 after:$personID)
```

### CUD 
We already have a glimpse of how to read, let's consider how to Create, Update, or Delete an entity. 

These operations follow the same syntactic structure as reads, but they are always preceded with the `mutation` keyword. 

```
mutation {
  createPerson(name: "David Bowie", age: 70) {
    name
    age
  }
}
```

Here we are creating a new person, and we are asking his name and age to be returned. 

## REST is not going anywhere
That's fine, GraphQL has plenty of advantages over our reigning champion, yet, REST is not going anywhere shortly. For one, it is still the lingua franca of Web services. Almost everything out there is built over REST. It is easy to use, and the GraphQL tooling is not yet on par. 