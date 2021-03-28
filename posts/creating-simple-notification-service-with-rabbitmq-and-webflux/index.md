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

*Application General Flow:*
![Example image](/posts/creating-simple-notification-service-with-rabbitmq-and-webflux/simple-notification-architecture.svg.png)

*RabbitMq Flow:*
![Example image](/posts/creating-simple-notification-service-with-rabbitmq-and-webflux/simple-notification-rabbitmq-architecture.svg.png)

All Messages send to exchange(left side).\
All User subscriptions(listening to new messages) are bound to separate queue.\
Queues are An exclusive queue, means that after user disconnect from subscription queue will be deleted automatically.

### Api for Sending Message


*Controller:* [link](https://github.com/tislib/blog-examples/blob/master/rabbitmq-webflux-simple-group-based-messaging/src/main/java/net/tislib/blog/examples/rabbitmqwebflux/controller/simple/UserSimpleMessageController.java)

```
@PostMapping
public Mono<Void> sendMessage(@PathVariable long userId,
                              @RequestBody String content) {
    return userSimpleMessageService.sendMessage(userId, content);
}
```

*Service:* [link](https://github.com/tislib/blog-examples/blob/master/rabbitmq-webflux-simple-group-based-messaging/src/main/java/net/tislib/blog/examples/rabbitmqwebflux/service/impl/UserSimpleMessageServiceImpl.java)

*topicName is the name of exchange:*
```
private final String topicName = "user-simple-message";
```

*We need to define routingKey to send a message to correct groups, each user has only one routingKey*
```
String routingKey = topicName + "-" + userId;
OutboundMessage message = new OutboundMessage(topicName, routingKey, content.getBytes());
```

*Declare Exchange:*
```
final Mono<AMQP.Exchange.DeclareOk> declareExchange = sender.declareExchange(
        ExchangeSpecification.exchange()
                .name(topicName)
                .durable(true)
                .type("topic")
);
```

*Send message:*

```
 return declareExchange.flatMap(item -> sender.send(Mono.fromSupplier(() -> message)));
```

*Final Code:*
```
@Override
public Mono<Void> sendMessage(long userId, String content) {
    String routingKey = topicName + "-" + userId;

    OutboundMessage message = new OutboundMessage(topicName, routingKey, content.getBytes());

    final Mono<AMQP.Exchange.DeclareOk> declareExchange = sender.declareExchange(
            ExchangeSpecification.exchange()
                    .name(topicName)
                    .durable(true)
                    .type("topic")
    );

    return declareExchange
            .flatMap(item -> sender.send(Mono.fromSupplier(() -> message)));
}
```

### Subscribe to user notifications

*Controller:* [link](https://github.com/tislib/blog-examples/blob/master/rabbitmq-webflux-simple-group-based-messaging/src/main/java/net/tislib/blog/examples/rabbitmqwebflux/controller/simple/UserSimpleMessageController.java)

```
@GetMapping(produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> receive(@PathVariable long userId,
                            @RequestParam(required = false) Integer timeout,
                            @RequestParam(required = false) Integer maxMessageCount) {
    return userSimpleMessageService.receive(userId, timeout == null ? null : Duration.ofMillis(timeout), maxMessageCount);
}
```

*Service:* [link](https://github.com/tislib/blog-examples/blob/master/rabbitmq-webflux-simple-group-based-messaging/src/main/java/net/tislib/blog/examples/rabbitmqwebflux/service/impl/UserSimpleMessageServiceImpl.java)

*topicName is the name of exchange:*
```
private final String topicName = "user-simple-message";
```

*RotingKey will be used to bind queue to exchange*
```
String routingKey = topicName + "-" + userId;
```

*Declare Queue:*
```
final Mono<AMQP.Queue.DeclareOk> declareQueue = sender
                .declareQueue(QueueSpecification.queue())
                .log("declare-queue", Level.FINER);
```

*Bind Queue: This code will create a queue and bind it to exchange via routingKey, queueName is returned*
```
final Mono<String> bindQueue = declareQueue
    .flatMap(declareOk ->
            sender.bindQueue(
                    BindingSpecification.binding()
                            .queue(declareOk.getQueue())
                            .exchange(topicName)
                            .routingKey(routingKey)
            ).map(bindOk -> declareOk.getQueue())) // this code is for returning queueName instead of bind result
    .log("bind-queue", Level.FINER);
```

*Prepare receiver flux:*
```
Flux<String> result = bindQueue
              .flatMapMany(receiver::consumeAutoAck)
              .map(item -> new String(item.getBody()));
```

*If timeout or maxMessage count is given, prepare them:*
```
if (timeout != null) {
    result = result.timeout(timeout);
}

if (maxMessageCount != null) {
    result = result.limitRequest(maxMessageCount);
}
```



*Final Code:*
```
@Override
public Flux<String> receive(long userId, Duration timeout, Integer maxMessageCount) {
    final String routingKey = topicName + "-" + userId;

    final Mono<AMQP.Queue.DeclareOk> declareQueue = sender
            .declareQueue(QueueSpecification.queue())
            .log("declare-queue", Level.FINER);

    final Mono<String> bindQueue = declareQueue
            .flatMap(declareOk ->
                    sender.bindQueue(
                            BindingSpecification.binding()
                                    .queue(declareOk.getQueue())
                                    .exchange(topicName)
                                    .routingKey(routingKey)
                    ).map(bindOk -> declareOk.getQueue())) // this code is for returning queueName instead of bind result
            .log("bind-queue", Level.FINER);

    Flux<String> result = bindQueue
            .flatMapMany(receiver::consumeAutoAck)
            .map(item -> new String(item.getBody()));

    if (timeout != null) {
        result = result.timeout(timeout);
    }

    if (maxMessageCount != null) {
        result = result.limitRequest(maxMessageCount);
    }

    return result;
}
```



## Advanced implementation (group to user notifications)

### Architecture

*Application General Flow:*
![General Flow](/posts/creating-simple-notification-service-with-rabbitmq-and-webflux/advanced-notification-architecture.svg.png)

*RabbitMq Flow:*
![RabbitMq Flow](/posts/creating-simple-notification-service-with-rabbitmq-and-webflux/advanced-notification-rabbitmq-architecture.svg.png)

*Subscription Flow:*
![Subscription Flow](/posts/creating-simple-notification-service-with-rabbitmq-and-webflux/advanced-subscription-flow.svg.png)

1. All Messages send to exchange per group(left side).
2. Each application is subscribed all groups (in that case user count will not affect queue count, only group count will affect it)
3. Users are subscribed to Flux
4. Group subscription is multicasted to user subscriptions, so when group notification is received by application group subscriber it will be sent to all user subscriptions
5. Queues are an exclusive queue, means that after user disconnect from subscription queue will be deleted automatically.

### Api for Sending Message (group)

*Controller:* [link](https://github.com/tislib/blog-examples/blob/master/rabbitmq-webflux-simple-group-based-messaging/src/main/java/net/tislib/blog/examples/rabbitmqwebflux/controller/advanced/GroupMessageController.java)

```
@PostMapping
public Mono<Void> sendMessage(@PathVariable long groupId, @RequestBody String content) {
    return userMessageService.sendMessage(groupId, content);
}
```

*Service:* [link](https://github.com/tislib/blog-examples/blob/master/rabbitmq-webflux-simple-group-based-messaging/src/main/java/net/tislib/blog/examples/rabbitmqwebflux/service/impl/UserMessageServiceImpl.java)

*topicName is the name of exchange:*
```
private final String topicName = "user-message";
```

*We need to define routingKey to send a message to correct groups, each user has only one routingKey*
```
String routingKey = topicName + "-" + groupId;
OutboundMessage message = new OutboundMessage(topicName, routingKey, content.getBytes());
```

*Declare Exchange:*
```
final Mono<AMQP.Exchange.DeclareOk> declareExchange = sender.declareExchange(
        ExchangeSpecification.exchange()
                .name(topicName)
                .durable(true)
                .type("topic")
);
```

*Send message:*

```
 return declareExchange.flatMap(item -> sender.send(Mono.fromSupplier(() -> message)));
```

*Final Code:*
```
@Override
public Mono<Void> sendMessage(long groupId, String content) {
    String routingKey = topicName + "-" + groupId;

    OutboundMessage message = new OutboundMessage(topicName, routingKey, content.getBytes());

    final Mono<AMQP.Exchange.DeclareOk> declareExchange = sender.declareExchange(
            ExchangeSpecification.exchange()
                    .name(topicName)
                    .durable(true)
                    .type("topic")
    );

    return declareExchange
            .flatMap(item -> sender.send(Mono.fromSupplier(() -> message)));
}
```

### Subscribe to user notifications

*Controller:* [link](https://github.com/tislib/blog-examples/blob/master/rabbitmq-webflux-simple-group-based-messaging/src/main/java/net/tislib/blog/examples/rabbitmqwebflux/controller/advanced/UserMessageController.java)

```
@GetMapping(produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> receive(@PathVariable long userId,
                            @RequestParam(required = false) Integer timeout,
                            @RequestParam(required = false) Integer maxMessageCount) {
    return userSimpleMessageService.receive(userId, timeout == null ? null : Duration.ofMillis(timeout), maxMessageCount);
}
```

*Service:* [link](https://github.com/tislib/blog-examples/blob/master/rabbitmq-webflux-simple-group-based-messaging/src/main/java/net/tislib/blog/examples/rabbitmqwebflux/service/impl/UserMessageServiceImpl.java)

*topicName is the name of exchange:*
```
private final String topicName = "user-message";
```

First We need to find which groups we need to subscribe in order subscribe to user notifications.\
We need to implement our own logic in locateGroups, it is currently dummy method, should be changed per application architecture
```
Flux<Long> groupIds = userService.locateGroups(userId);
```

*For each group, get group receiver(by connecting to group multicast) and combine them via flatMap*
```
Flux<String> result = groupIds.flatMap(item -> getGroupReceiver(item))
                .log("receiver-log", Level.FINER)
                .map(item -> new String(item.getBody()));
```

*If timeout or maxMessage count is given, prepare them:*
```
if (timeout != null) {
    result = result.timeout(timeout);
}

if (maxMessageCount != null) {
    result = result.limitRequest(maxMessageCount);
}
```

*Preparing Group Receiver:*\
We need to first check if we have group receiver(subscription) already defined, if no we need to prepare it.\
After that we need to create a Flux from our group receiver sink (this mechanism is for multicasting so each event will be sent to all Fluxes)

```
private Flux<Delivery> getGroupReceiver(Long groupId) {
      String routingKey = topicName + "-" + groupId;

      if (!groupConsumerMap.containsKey(routingKey)) {
          registerGroupReceiver(routingKey);
      }

      return groupConsumerMap.get(routingKey)
              .asFlux()
              .log("flux-log", Level.FINER);
}
```
asFlux() function creates a Flux from sink, it means that events coming to sink will be sent to new flux until it terminated.\

*Main part, registering group receiver and creating multicast sink:*\
Steps: 
1. Declare Group Queue (so each application replica will get copy of group message)
2. Create a sink which will be mulicaster
3. Receive messages from queue and send it to sink
4. Add sink to map (registering sink)
5. If sink terminated remove it from map

```
private synchronized void registerGroupReceiver(String routingKey) {
    Sinks.Many<Delivery> sink = Sinks.many().multicast().onBackpressureBuffer();

    Mono<String> declareQueue = sender
            .declareQueue(QueueSpecification.queue())
            .log("declare-queue", Level.FINER)
            .flatMap(declareOk ->
                    sender.bindQueue(BindingSpecification.binding()
                            .queue(declareOk.getQueue())
                            .exchange(topicName)
                            .routingKey(routingKey)).map(bindOk -> declareOk.getQueue()))
            .log("bind-queue", Level.FINER);

    declareQueue.flatMapMany(queueName -> receiver.consumeAutoAck(queueName, new ConsumeOptions()))
            .log("broker-log", Level.FINER)
            .doOnNext(sink::tryEmitNext)
            .doOnComplete(sink::tryEmitComplete)
            .doOnError(sink::tryEmitError)
            .subscribe();

    groupConsumerMap.put(routingKey, sink);

    sink.asFlux().doOnTerminate(() -> {
        groupConsumerMap.remove(routingKey, sink);
    }).subscribe();
}
```

*Final Code:*
```
@Override
@SuppressWarnings("ALL")
public Flux<String> receive(long userId, Duration timeout, Integer maxMessageCount) {
    Flux<Long> groupIds = userService.locateGroups(userId);

    Flux<String> result = groupIds.flatMap(item -> getGroupReceiver(item))
            .log("receiver-log", Level.FINER)
            .map(item -> new String(item.getBody()));

    if (timeout != null) {
        result = result.timeout(timeout);
    }

    if (maxMessageCount != null) {
        result = result.limitRequest(maxMessageCount);
    }

    return result;
}

private Flux<Delivery> getGroupReceiver(Long groupId) {
      String routingKey = topicName + "-" + groupId;

      if (!groupConsumerMap.containsKey(routingKey)) {
          registerGroupReceiver(routingKey);
      }

      return groupConsumerMap.get(routingKey)
              .asFlux()
              .log("flux-log", Level.FINER);
}

private synchronized void registerGroupReceiver(String routingKey) {
    Sinks.Many<Delivery> sink = Sinks.many().multicast().onBackpressureBuffer();

    Mono<String> declareQueue = sender
            .declareQueue(QueueSpecification.queue())
            .log("declare-queue", Level.FINER)
            .flatMap(declareOk ->
                    sender.bindQueue(BindingSpecification.binding()
                            .queue(declareOk.getQueue())
                            .exchange(topicName)
                            .routingKey(routingKey)).map(bindOk -> declareOk.getQueue()))
            .log("bind-queue", Level.FINER);

    declareQueue.flatMapMany(queueName -> receiver.consumeAutoAck(queueName, new ConsumeOptions()))
            .log("broker-log", Level.FINER)
            .doOnNext(sink::tryEmitNext)
            .doOnComplete(sink::tryEmitComplete)
            .doOnError(sink::tryEmitError)
            .subscribe();

    groupConsumerMap.put(routingKey, sink);

    sink.asFlux().doOnTerminate(() -> {
        groupConsumerMap.remove(routingKey, sink);
    }).subscribe();
}
```

