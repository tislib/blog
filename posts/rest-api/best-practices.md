---
title: "Rest API Best Practices"
date: 2021-03-21T00:00:00
lastmod: 2021-03-21T00:00:00 
slug: rest-api-best-practices 
draft: false
tags: ["rest-api", "best-practices"]
categories: ["posts", "rest-api"]
theme: "wide"
featuredImage: /posts/rest-api/assets/best-practices-cover.webp
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

| Method | Usage                 | Example
|--------|-----------------------|-------------------
| GET    | Get Resource          | ```GET /users``` ```GET /users/1```  ```GET /users/1/star```
| POST   | Create Resource       | ```POST /users``` ```POST /users/1/star```
| PUT    | REPLACE/Full Update   | ```PUT /users/15  {full body}``` 
| PATCH  | Partial Update        | ```PATCH /users/15 {partial body}``` 
| DELETE | Delete Resource       | ```PATCH /users/15``` 


### Nesting resources for hierarchical objects

Nested/Sub resources are good choice to handle relations

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
#### Solutions 1: Converting actions to sub-resources
We can think as our user resource has a sub resource named activated (adverb form of active) and if we create this sub resource we say that we activate user, if we remove this sub resource we say that we deactivate user
```
GET /users/15/activated    <- 200 -> activated, 404 -> dectivated
POST /users/15/activated   <- this will activate user
DELETE /users/15/activated <- this will deactivate user
```
Say idea for starring
```
POST /users/15/star  <- this will star the user
DELETE /users/15/star  <- this will unstar the user
```
Basically what we're trying to do is to make our actions look like Rest API, so translate actions to resources

#### Solution 2: Converting actions to fields
In this solution we think that activated and star are not sub resource, they are properties of our user
```
GET /users/15 RESPONSE: {...activated: true}
PATCH /users/15 REQUEST BODY: {activated: true}
```


### Handle errors gracefully and return standard error codes

#### Http status as Error code
*In rest API we should use HTTP status codes to describe what the problem is about\*
Main Idea for status codes:
* 2** (mainly 200 is used) status codes are for describing that operation is succeeded
* 3** redirection happened, the resource which you are trying to operate is moved to another resource identifier or moved to another server
* 4** client data are incorrect, client sent something wrong, need to fix it
* 5** server cannot process the problem, 5xx errors indicate that the problem cannot be fixed from client perspective and something corrupted server

In ideal world 5** error should not be happened, if we are returning 5** status codes, it means that we have some fault in application code or in our infrastructure

Let's go to detailed status codes (*most used error codes by REST APIs*)

| Status Code | Message | Rest Api Usage 
|-----|--------------------|-------------------
| 2** |                    |                   
| 200 | OK                 | Updating Resource with PUT OR PATCH method
| 201 | CREATED            | Creating Resource with POST Method
| 204 | NO_CONTENT         | Resource deleted, as we not returning any data in DELETE, it is better to indicate that there are no content |
| 4** |                    |
| 400 | BAD_REQUEST        | POST/PUT/PATCH, why creating/updating request, request input validation failed, client sent data which is wrong from servers perspective
| 401 | UNAUTHENTICATED    | User is not authenticated and this resource needs user identification (you need to login or send access token, etc.)
| 403 | FORBIDDEN          | User is identified but don't have access to this resource or cannot call this method in this access because of insufficient permission 
| 404 | NOT_FOUND          | Resource not found, we should return 404 if the resource we are operating on not found
| 405 | METHOD_NOT_ALLOWED | Resource not found, we should return 404 if the resource we are operating on not found
| 5** |                    |
| 500 | INTERNAL_ERROR     | We should use that if we have some found on application codes

Notes:
you should not use 404 if the resource which is you are accessing in backend logic not found, 404 is for when consumer try to do some operation on resource and that resource not found.\
Example:\
```
POST /restaurants\
{
   "user": {
      *"id": 13  <- we don't have a user with id=13*
   }
}
```

In this case we should not return 404 as the resource we are operating on is restaurants, what we could not find is different resource\
Instead we may use 400 Bad request with message user not found with id=13

#### Error message
Besides, status codes whe should generalize error response, we should return structured data for describing what went wrong\
for 400 errors it is better to show field errors alongside with field names like:
```
{
   "message" : "name, age fields rejected",
   "rejectedFields": [
      {
         "field": "name",
         "message": "must not be blank"
      },
      {
         "field": "age",
         "message": "must be between 0 and 150"
      }
   ]
}
```

### Allow filtering, sorting, and pagination
It is not important to handle filtering, sorting, pagination for all resources, this requirements should met consumer needs.\
But it is better to handle filtering, sorting and pagination for resources, and it is required to do it consistently

#### Filtering:

Filtering can be handled by two approach:
1. **Filter by query parameters: ```/users?age=13```** \
   Using filtering by query parameters is the best way, you may use filtering by query parameters if you can support *multipple and multichoice filtering* options like users can be searched by age, name, or both  
2. **Filter by path parameters: ```/users/by-age/13```** \
   This approach is good to choice if you are few and dedicated filtering options, like you can only filter by age or by name

#### Sorting:

```/users?sort=+age,-name```

#### Pagination:

```/users?page=1&pageSize=10```

### Bulk Operations in Rest API
Sometimes it is required to do bulk operation do increase performance

Let's check some bulk operations in Rest API world:

#### Remove sub-resources of resource OR Remove everything in collection
```DELETE /users``` <- this will cause to delete entire collection, to delete all its items

#### Bulk create resources and/or update
```PATCH /users ``` <- this will check all sent items one by one, and if the resource is exists it will update it, if not create it

#### Bulk replace resources
```PUT /users``` <- this will update all users to given state (given in request body) and remove everything else like replacing resource with given

### Api versioning
Api versioning and handling versions with caution is a must in Rest API world.\
Example for a version:
```
/api/1.0/users
```
if we go production with version 1 and we need to do some breaking changed to version 1 (which is affecting rest api design) we need to introduce new version
```
/api/1.1/users
```
### Use Hateoas

#### What is hateoas?
Hateoas stands for Hypermedia as the Engine of Application State

Example: 
```
GET /api/1.0/users/1

{
   "username": "taleh123",
   "firstName": "Taleh",
   "lastName": "Ibrahimli",
   "_links":{ <- hateoas links
        "self":{
            "href":"http://localhost:8080/api/1.0/users/1"  <- resource self link
        },
        "orders":{
            "href":"hhttp://localhost:8080/api/1.0/users/1/orders" <- link to orders
        }
    }
}
```

Hateoas is very useful to build self descriptive REST APIs. 
