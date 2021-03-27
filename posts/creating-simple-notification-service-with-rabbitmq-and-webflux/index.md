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

**Repository:** [link](https://github.com/tislib/blog-examples/tree/master/rabbitmq-webflux-simple-group-based-messaging)

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

## Project Setup
Setup Spring Boot Project

### Dependencies

#### Maven
```
<dependencies>
    <dependency>
      <groupId>io.projectreactor.rabbitmq</groupId>
      <artifactId>reactor-rabbitmq</artifactId>
      <version>1.5.2</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-webflux</artifactId>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>io.projectreactor</groupId>
      <artifactId>reactor-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.amqp</groupId>
      <artifactId>spring-rabbit-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
```

#### Gradle
```
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-amqp'

    compile "io.projectreactor.rabbitmq:reactor-rabbitmq:1.5.2"

    implementation 'org.springframework.boot:spring-boot-starter-webflux'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'io.projectreactor:reactor-test'
    testImplementation 'org.springframework.amqp:spring-rabbit-test'
}
```

### RabbitMq Configuration
[RabbitMqConfiguration.java](https://github.com/tislib/blog-examples/blob/master/rabbitmq-webflux-simple-group-based-messaging/src/main/java/net/tislib/blog/examples/rabbitmqwebflux/RabbitMqConfiguration.java)

**We create following Beans for configuring RabbitMQ**

*Mono Connection:*
```
// the mono for connection, it is cached to re-use the connection across sender and receiver instances
// this should work properly in most cases
@Bean
Mono<Connection> rabbitMqConnection(RabbitProperties rabbitProperties) {
    ConnectionFactory connectionFactory = new ConnectionFactory();
    connectionFactory.setHost(rabbitProperties.getHost());
    connectionFactory.setPort(rabbitProperties.getPort());
    connectionFactory.useNio();  // <- with this flag our RabbitMq connection will be non-blocking
    connectionFactory.setUsername(rabbitProperties.getUsername());
    connectionFactory.setPassword(rabbitProperties.getPassword());
    return Mono.fromCallable(() -> connectionFactory.newConnection("reactor-rabbit")).cache();
}
```

*Sender bean for sending messages from application to message broker:*
```
@Bean
Sender sender(Mono<Connection> connectionMono) {
    return RabbitFlux.createSender(new SenderOptions()
            .connectionMono(connectionMono));
}
```

*Receiver bean for receiving messages from message broker any topic:*
```
@Bean
Receiver receiver(Mono<Connection> connectionMono) {
    return RabbitFlux.createReceiver(new ReceiverOptions()
            .connectionMono(connectionMono));
}
```

*Close broker connection before application is destroying:*
```
@PreDestroy
public void close() throws Exception {
    rabbitMqConnectionMono.block().close();
}
```

## Simple implementation (user to user notifications)

### Architecture

![Example image](/posts/creating-simple-notification-service-with-rabbitmq-and-webflux/simple-notification-architecture.svg)

### Api for Sending Message

