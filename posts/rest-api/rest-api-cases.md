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
**Problem:**

We need to 
