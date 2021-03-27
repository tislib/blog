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

In this post we will create notification service. Which can be used to send notifications from server to client (e.g.
from backend to frontend)

## Use Cases

1. Group Chat, messages to groups should be sent to all users on that group
2. Event System, subscribe to some events and get notified
3. User RabbitMq Message Broker

## Specification

### Simple Notifications: user to user

1. Send notification to user
2. Subscribe to specific user notifications as topic (each subscription will get a copy)
3. User RabbitMq Message Broker and **queue per user** approach

### Advanced Notifications: group to user

1. Send notification to group (and all group members will be notified)
2. Subscribe to specific user notifications as topic (each subscription will get a copy)
3. User RabbitMq Message Broker and **queue per group** approach
