---
title: "Creating simple notification service by RabbitMq & Webflux"
date: 2021-03-21T00:00:00
lastmod: 2021-03-21T00:00:00
slug: creating-simple-notification-service-with-rabbitmq-and-webflux
description: "Simple Notification service where users can be notified by group"
resources:
- name: "featured-image"
  src: "featured-image.png"
page:
    theme: "wide"
tags: ["asynchronous-programming", "messaging", "rabbit-mq", "spring-webflux", "spring-boot"]
categories: ["posts", "asynchronous-programming"]
summary: "Simple Notification service where users can be notified by group"
toc:
  auto: false
---

## Concept

Simple Notification service where users can be notified by group

## Details

1. We have **users** , **groups** the mapping between user and grouping can be changed anytime
2. Notification can be created by calling rest api and sending message to a group. All group members will get the
   notification
3. Users can be added to and removed from group anytime, and it will cause immediately change on their subscriptions
4. Support for both **Server Sent Events** and **Long Polling** techniques
5. API for detecting which message is read and which message is delivered


