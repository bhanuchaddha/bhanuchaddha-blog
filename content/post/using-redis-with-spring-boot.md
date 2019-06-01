---
title: "Using Redis with Spring Boot"
date: 2019-06-01T01:11:42+02:00
author: "Bhanu Chaddha"
linktitle: Using Redis with Spring Boot
next:
prev:
weight: 1
authorAvatar: image/logo.svg
image: image/1/redisSpringBoot.png
tags : [
    "backend",
    "java",
    "redis",
    "springboot",
    "development",
]
categories : [
    "tutorial"
]
draft: false
---

> Redis is an open source, in-memory data structure store, used as a database, cache and message broker. Here we would use [Redis](https://redis.io/) as Message Broker to create messaging layer in our demo finance application. [This](https://github.com/bhanuchaddha/Finance-System) application consist of multiple micro services which interact with each other. We would create event driven construct, where services would connect to messaging layer and would publish and subscribe events to messaging layer.
Auto reloading feature does not work out of the box in Intellij. Below are the changes required to make it work correctly.

In [this](https://github.com/bhanuchaddha/Finance-System) application we have an Account Service which provide account information of the customers and a Payment Service which provide the payment information of the customer. You can check the complete application [here](https://github.com/bhanuchaddha/Finance-System) .We would set up messaging between these two services. Whenever a new payment would be created using Payment Service, An event would be published to the messaging layer. Our Account service would receive this event and update the balance in accounts.

We would implement this set up in 3 steps.

##   Step 1: Setting up Redis server

We can run redis locally by following steps on [Redis official website](https://redis.io/download). Or you can run embedded redis in your application. I have taken later approach in this case. Check out [Messaging Layer](https://github.com/bhanuchaddha/Bank-on-Cloud/tree/master/messaging-layer).

I have added embedded-redis as dependency in our maven project.

```
<dependency>
   <groupId>com.github.kstyrc</groupId>
   <artifactId>embedded-redis</artifactId>
   <version>0.6</version>
</dependency>
```

And hooked `start` and `stop` redis functions with the messaging-layer service’s life-cycle.

Below is the sample code.

```java
@SpringBootApplication
public class MessagingApplication {

   private RedisServer redisServer;
   public static void main(String[] args) {
      SpringApplication.run(MessagingApplication.class, args);
   }


   @PostConstruct
   public void startRedis() throws IOException {
      redisServer  = new RedisServer(6379);
      redisServer.start();
   }

   @PreDestroy
   public void stopRedis(){
      redisServer.stop();
   }
}
```

Now if you start messaging-layer spring boot application it will start your redis server at port ```6379``` by default or at specified port.


## Step 2: Publish payment event on Redis Server


Now when you redis server up and running, Lets set up ```payment-service``` to publish event on messaging layer.

### **Add Redis client dependency**
We need to add below dependency to out maven project. This is required to connect Redis Server.

```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
    <type>jar</type>
</dependency>

```

### **Add Messaging configuration**

We need `RedisTemplate` to communicate to redis, and to configure RedisTemplate we need other components like `JedisConnectionFactory` and `ChannelTopic`.

ChannelTopic is the topic/queue where our Payment Service would publish the events. We are configuring topic named “payments” here. This is the same topic on which Account Service would subscribe to.

```java
@Bean
JedisConnectionFactory jedisConnectionFactory() {
    return new JedisConnectionFactory();
}

@Bean
ChannelTopic topic() {
    return new ChannelTopic("payments");
}

@Bean
public RedisTemplate<String, Payment> redisTemplate() {
    final RedisTemplate<String, Payment> template = new RedisTemplate<String, Payment>();
    template.setConnectionFactory(jedisConnectionFactory());
    template.setValueSerializer(new Jackson2JsonRedisSerializer<Payment>(Payment.class));
    return template;
}
```

That’s all needed to setup in our application to connect and publish event to our Redis server.

I have configured Jackson2JsonRedisSerializer to serialize our [Payment](https://github.com/bhanuchaddha/Bank-on-Cloud/blob/master/payment/src/main/java/com/bhanuchaddha/bank/payment/Payment.java) object to string correctly on the topic. Here I am publishing whole payment object as an event but we could create a custom event as well to publish.

Now we can autowire our Redis Template to any service layer to send a event to our payment topic.

Here I have created a wrapper class [RedisMessagePublisher](https://github.com/bhanuchaddha/Finance-System/blob/master/payment/src/main/java/com/bhanuchaddha/bank/payment/RedisMessagePublisher.java) and used that in service layer PaymentEventHandler.

I am using [Spring Data Rest](https://spring.io/projects/spring-data-rest) and ```@RepositoryEventHandler(Payment.class)``` to hook [RedisMessagePublisher](https://github.com/bhanuchaddha/Finance-System/blob/master/payment/src/main/java/com/bhanuchaddha/bank/payment/RedisMessagePublisher.java). But Redis Template can be used directly from any class. Below is how I am using the Redis Template.

```java
@Service
public class RedisMessagePublisher implements MessagePublisher {

    @Autowired
    private RedisTemplate<String, Payment> redisTemplate;
    @Autowired
    private ChannelTopic topic;

    public RedisMessagePublisher() {
    }
    public void publish(Payment message) {
        redisTemplate.convertAndSend(topic.getTopic(), message);
    }
}
@Component 
@RepositoryEventHandler(Payment.class)
public class PaymentEventHandler {

    @Autowired
    private RedisMessagePublisher publisher;



    @HandleAfterCreate
    public void handlePaymentSave( Payment payment) {
        publisher.publish(payment);
    }
}
```

## Step 3: Subscribe event from Redis Server

We would configure [Account Service](https://github.com/bhanuchaddha/Finance-System/tree/master/account) to subscribe events from our “payments” topic at Redis Server.

### **Add Redis client dependency**
We need to add below dependency to out maven project.

```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
    <type>jar</type>
</dependency>
```
### **Add Message Listener**

Events from our “payments” topic would arrive at the [Message Listener](https://github.com/bhanuchaddha/Finance-System/blob/master/account/src/main/java/com/bhanuchaddha/bank/accounts/RedisMessageSubscriber.java) and from there we can use the event as we need. We need to implement onMessage method of MessageListener interface. In our case I am using this event to adjust the balance in Accounts.

Lets Implement the [Message Listener](https://github.com/bhanuchaddha/Finance-System/blob/master/account/src/main/java/com/bhanuchaddha/bank/accounts/RedisMessageSubscriber.java).

```java
@Service
public class RedisMessageSubscriber implements MessageListener {

    ObjectMapper objectMapper = new ObjectMapper();

    @Autowired
    private AccountRepository repository;


    public void onMessage(final Message message, final byte[] pattern) {
        try {
            Payment payment = objectMapper.readValue(message.toString(), Payment.class);
            Account fromAccount = repository.findById(payment.getFromAccount()).get();
            Account toAccount = repository.findById(payment.getToAccount()).get();
            fromAccount.setAmount(fromAccount.getAmount().subtract(payment.getAmount()));
            toAccount.setAmount(toAccount.getAmount().add(payment.getAmount()));
            repository.save(fromAccount);
            repository.save(toAccount);
        } catch (IOException e) {
            e.printStackTrace();
        }
        System.out.println("Message received: " + new String(message.getBody()));
    }
}
```

### **Add Messaging configuration**
To use our MessageListener we need to add this Message Listener to `RedisMessageListenerContainer` . Below is how I have have configured the relevant components to subscribe the event.

```java
@Configuration
public class MessagingConfiguration {

    @Bean
    JedisConnectionFactory jedisConnectionFactory() {
        return new JedisConnectionFactory();
    }

    @Bean
    ChannelTopic topic() {
        return new ChannelTopic("payments");
    }

    @Autowired
    RedisMessageSubscriber redisMessageSubscriber;

    @Bean
    MessageListenerAdapter messageListener( ) {
        return new MessageListenerAdapter(redisMessageSubscriber);
    }

    @Bean
    RedisMessageListenerContainer redisContainer() {
        RedisMessageListenerContainer container
                = new RedisMessageListenerContainer();
        container.setConnectionFactory(jedisConnectionFactory());
        container.addMessageListener(messageListener(), topic());
        return container;
    }
}
```
We need ChannelTopic, JedisConnectionFactory and we need to configure these in RedisMessageListenerContainer along with the Message Listener.

That’s it!!! This is all you need to use Redis Pub/Sub with Spring boot application.

Lets test our application now. Check out the source code from [here](https://github.com/bhanuchaddha/Finance-System). And run above mentioned Accounts and Payments services with Messaging layer service.

Now if you create 2 accounts, say 1 and 2. Run below command twice to do that. It will create account number 1 and account number 2 both with the balance of 200.

```bash
curl -X POST http://localhost:8081/accounts -H 'Content-Type: application/json'    
-d '{
 "amount":200
}'
```
Lets check the created accounts:

```bash
curl -X GET http://localhost:8081/accounts 
// this would result in
{
    "_embedded": {
        "accounts": [
            {
                "customerNumber": 0,
                "amount": 200,
                "_links": {
                    "self": {
                        "href": "http://localhost:8081/accounts/1"
                    },
                    "account": {
                        "href": "http://localhost:8081/accounts/1"
                    }
                }
            },
            {
                "customerNumber": 0,
                "amount": 200,
                "_links": {
                    "self": {
                        "href": "http://localhost:8081/accounts/2"
                    },
                    "account": {
                        "href": "http://localhost:8081/accounts/2"
                    }
                }
            }
        ]
    }
}
```

Now if we make payment from account 1 to account 2

```bash
curl -X POST \
  http://localhost:8082/payments \
  -H 'Content-Type: application/json' \
  -d '{
 "fromAccount":1,
 "toAccount":2,
 "amount":150
}'
```

This will create new payment in payment service and an event is published at our Redis payments topic. As Account Service subscribe to this topic , It will receive the event and process it. You will notice below log in Account Service

```
Message received: {"id":1,"fromAccount":1,"toAccount":2,"amount":150,"date":1544562950359}
```

And now if you fetch the accounts again. You will notice that balance in accounts have been updated

```
curl -X GET \
  http://localhost:8081/accounts \
// this would result in
{
    "_embedded": {
        "accounts": [
            {
                "customerNumber": 0,
                "amount": 50,
                "_links": {
                    "self": {
                        "href": "http://localhost:8081/accounts/1"
                    },
                    "account": {
                        "href": "http://localhost:8081/accounts/1"
                    }
                }
            },
            {
                "customerNumber": 0,
                "amount": 350,
                "_links": {
                    "self": {
                        "href": "http://localhost:8081/accounts/2"
                    },
                    "account": {
                        "href": "http://localhost:8081/accounts/2"
                    }
                }
            }
        ]
    }
}
```
The benefit of using the Messaging Layer is that there is no direct communication between services and this could result in better decoupling and scalability of the application.