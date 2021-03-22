---
title: "Rest API Best Practices"
date: 2021-03-21T00:00:00 
lastmod: 2021-03-21T00:00:00 
slug: rest-api-best-practices 
draft: false
tags: ["rest-api", "best-practices"]
categories: ["posts", "rest-api"]
summary: "Rest Api General Best Practices, and their reasons why you should go with that way"
---
### Don't write as Restful, Think as Restful

* API's abstraction should be different from its implementation, if you want to write better API don't try to represent
   your backend logic as is, instead think how API may be better to use by its clients, always think as How API consumer will use it
* Try to be proactive, don't try to translate legacy API's OR SOAP style simply to REST API, try to be proactive,
   think that how this API should be easy to use, how can I make it like a CRUD
* Everything is CRUD, Any resource can be Created, Read, Updated, Deleted, Anything else can be represented as Resource

### Use nouns instead of verbs in endpoint paths

As we know REST is representational state transfer, we are transferring current state of resource from client to server OR wise versa.\
In Rest API endpoint is the resource identifier.

Incorrect variant:\
GET /getUserById/15

if we think as endpoint is resource identifier and method is crud operation on this resource we be confused:\
Get the resource which identifier is /getUserById/15

Correct variant\
GET /users/15

If now make sense, GET the resource which identifier is /users/15

### Name collections with plural nouns

We should name our collections as plural, mainly for consistency

/users      <- we can think this as we are getting list of users
/users/1    <- we can think this as we are getting user number one from list of users(previous collection resource)

If we use singular names, it will be confusing

/user  <- it may be misunderstood as we are getting single user
/user/1  <- it may be misunderstood as we are getting user's sub resource named 1

Examples:\
```
GET /users    
GET /users/1
GET /users/1/orders    
```

### Map Correct HTTP Methods to REST Operations

![](/posts/rest-api/assets/crud-operations.png)

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

### Convert Actions to *Resources* or *Fields*

Let's say we have user resource, and we want to enable/disable, we want to give star to this user

Simple but not desired:
```
POST /users/15/add-star
POST /users/15/remove-star
POST /users/15/activate
POST /users/15/deactivate
```

What instead we can do?\
*Solutions 1: Converting actions to sub-resources*\
We can think as our user resource has a sub resource named activated (adverb form of active) and if we create this sub resource we say that we activate user, if we remove this sub resource we say that we deactivate user
```
GET /users/15/activated    <- this will return 200 if user is activated, 404 if user not activated
POST /users/15/activated   <- this will activate user
DELETE /users/15/activated <- this will deactivate user
```
Say idea for starring
```
POST /users/15/star  <- this will star the user
DELETE /users/15/star  <- this will unstar the user
```
Basically what we're trying to do is to make our actions look like Rest API, so translate actions to resources

*Solution 2: Converting actions to fields*\
In this solution we think that activated and star are not sub resource, they are properties of our user
```
GET /users/15 RESPONSE: {...activated: true}
PATCH /users/15 REQUEST BODY: {activated: true}
```


[comment]: <> (### Handle errors gracefully and return standard error codes)

[comment]: <> (### Allow filtering, sorting, and pagination)

[comment]: <> (### Maintain Good Security Practices)

[comment]: <> (### Cache data to improve performance)
