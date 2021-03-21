---
title: "Rest API General Guidelines"
date: 2021-03-21T00:00:00
lastmod: 2021-03-21T00:00:00
slug: rest-api-guidelines
draft: false
tags: ["rest-api", "guidelines"]
categories: ["posts", "rest-api"]
summary: "Rest Api General Guidelines is for understanding Best Practices of Rest Api Architecture"
---

# Table of contents

* [Main idea](#main-idea)
* [Best Practices](#best-practices)
    * [Don't write as Restful, Think as Restful](#dont-write-as-restful-think-as-restful)
    * [Use nouns instead of verbs in endpoint paths](#use-nouns-instead-of-verbs-in-endpoint-paths)
    * [Name collections with plural nouns](#name-collections-with-plural-nouns)
    * [Nesting resources for hierarchical objects](#nesting-resources-for-hierarchical-objects)
    * [Handle errors gracefully and return standard error codes](#handle-errors-gracefully-and-return-standard-error-codes)
    * [Allow filtering, sorting, and pagination](#allow-filtering-sorting-and-pagination)
    * [Maintain Good Security Practices](#maintain-good-security-practices)

## Main idea

The main idea on Rest API is that, parties communicate by sharing their current state instead of calling some action

Legacy APIs

![](/posts/rest-api/assets/general-guidelines-1.png)

When you want to call backend or some application you call some action in server, so you cannot predict what the outcome
will be, you need to ask from server what is its new state, what happened after your action

One of the main problem here is that, you need to have lots of DTOs, mapping and lots of duplicates which can fail
easily (userUpdateRequest, userUpdateResponse, userGetRequest, userGetResponse, etc.)

New APIs

![](/posts/rest-api/assets/general-guidelines-2.png)

The main idea in Rest API is you transfer your current state, so instead of calling some action on backend, frontend is
sharing current state about user1 with backend, in other word, frontend is replacing user1 state in backend with its own
state

When we use Rest API we will operate all operations as CRUD operation, it will help us to cut down features to
well-designed CRUD operations, better to understand by API users, better abstraction

## Best Practices

### Don't write as Restful, Think as Restful

How to think as restful?

1. API's abstraction should be different than its implementation, if you want to write better API don't try to represent
   your backend logic as is, instead think how API may be better to use by its clients
2. Try to be proactive, don't try to translate legacy API's OR SOAP style to simply to REST API, try to be proactive,
   think that how this API should be easy to use, how can I make it like a CRUD
3. Everything is CRUD

### Use nouns instead of verbs in endpoint paths

Why Nouns not verbs

* Noun reminds us actions not resource e.g. ```GET /getUserById```  OR ```GET /users/1```  in first call we are
  executing action, in second call we ar just reading some user resource from server
* As we say everything is CRUD, you need to have something (and it is a noun) so you are doing CRUD operations on it

### Name collections with plural nouns

We should name our collections as plural, mainly for consistency

```
GET /users    
GET /users/1
GET /users/1/orders    
```

### Nesting resources for hierarchical objects

Nested/Sub resources are good handle relations

```
GET /users/1/orders     <- nested resource
GET /orders?userId=1    <- main resource by filtering
```

When to use nested resource?

1. If nested resource can be created/updated/queried only if you have information about parent, like you cannot do
   anything in orders without knowing which user is it related
2. If you want to do operation on resource without knowing or having its parent, in that case nested resources are not
   correct choice

### Handle errors gracefully and return standard error codes

### Allow filtering, sorting, and pagination

### Maintain Good Security Practices

### Cache data to improve performance
