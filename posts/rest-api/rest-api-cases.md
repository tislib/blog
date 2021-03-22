---
title: "Learn Rest API by examples"
date: 2021-03-21T03:36:48+04:00
slug: learn-rest-api-by-examples
draft: false
tags: ["rest-api", "examples"]
categories: ["posts", "rest-api"]
summary: "Learn Rest Api by Real World Examples"
---

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
Let's say we have user resource, and we want to enable/disable, we want to give star to this user

Simple but not desired:
```
POST /users/15/add-star
POST /users/15/remove-star
POST /users/15/activate
POST /users/15/deactivate
```

What instead we can do?\
Solutions 1: *Converting actions to sub-resources*\
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

Solution 2: *Converting actions to fields *\
In this solution we think that activated and star are not sub resource, they are properties of our user
```
GET /users/15 RESPONSE: {...activated: true}
PATCH /users/15 REQUEST BODY: {activated: true}
```

**Conclusion:**
We can hack resources to support actions but with Rest architecture compatible
