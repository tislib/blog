---
title: "Creating simple notification service by RabbitMq & Webflux"
date: 2021-03-21T00:00:00 
lastmod: 2021-03-21T00:00:00 
slug: creating-simple-notification-service-with-rabbitmq-and-webflux 
description: "Simple Notification service where users can be notified by group"
resources:
- name: "featured-image"
  src: "featured-image.jpeg"
page:
  theme: "wide"
tags: ["asynchronous-programming", "messaging", "rabbit-mq", "spring-webflux", "spring-boot"]
categories: ["posts", "asynchronous-programming"]
summary: "Simple Notification service where users can be notified by group"
toc:
  auto: false
---

**
Repository:** [link](https://github.com/tislib/blog-examples/tree/master/rabbitmq-webflux-simple-group-based-messaging)

## Concept

Simple Notification service where users can be notified by group, this is useful for complex message routing.\
We are trying to build scalable system which can be easily replicated

## Use Cases

1. Group Chat, messages to groups should be sent to all users on that group
2. Event System, subscribe to some events and get notified

## Specification

### Simple Notifications: user to user

1. Send notification to user
2. Subscribe to specific user notifications as topic (each subscription will get a copy)

### Advanced Notifications: group to user

1. Create/Update/Delete users
2. Create/Update/Delete groups
3. Assign user to group(s)
4. Subscribe to User messages
5. Send a message to group, and it will be delivered to all member users by their subscription
6. When user - group assignment is changed, subscription should adopt itself automatically
7. In Replication mode, if user/group added/modified in one replica, another replicas should reconfigure subscriptions
   immediately
8. Support for both **Server Sent Events** and **Long Polling** techniques
9. API for getting all messages for particular user with delivered, read properties
