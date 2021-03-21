---
title: "Rest API General Guidelines"
date: 2021-03-21T03:36:48+04:00
draft: false
---

# Table of contents

* [Main idea](#main-idea)
* [Best Practices](#best-practices)
* [Example cases](#example-cases)

## Main idea

The main idea on Rest API is that, parties communicate by sharing their current state instead of calling some action

Legacy APIs

![](assets/general-guidelines-1.png)

When you want to call backend or some application you call some action in server, so you cannot predict what the outcome
will be, you need to ask from server what is its new state, what happened after your action

One of the main problem here is that, you need to have lots of DTOs, mapping and lots of duplicates which can fail
easily (userUpdateRequest, userUpdateResponse, userGetRequest, userGetResponse, etc.)

New APIs

![](assets/general-guidelines-2.png)

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

## Example Cases

### Case 1
**Problem:**

You have users, and users can follow each other, you want to create API for getting users, following/unfollowing users, and getting list of followers and following users (like in instagram)

**Solution:**
```
#User CRUD:
GET /users  <- get list of users

POST /users/15/followings    <- user 15 follows user 17 
{
    "user": {
        "id": 17
    }
}

GET /users/15/followings/17   <- check if use 15 follows 17

DELETE /users/15/followings/17   <- user 15 unfollows user 17
```

**Conclusion:**

Translate actions to resources, e.g. do follow = POST following, unfollow = DELETE following



### Case 2
**Problem:**

We need to 
