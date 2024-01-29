# RabbitMQ

> Message Broker Software
> AMQP(Advanced Message Queuing Protocol) 지원: 메시지 지향
> Queuing, Routing(Point to Point, Pub / Sub), 신뢰성, 보안

## Routing

> 메시지는 교환(Exchange)을 통해 큐로 전달됨
> 교환은 메시지를 어떻게 라우팅할지 결정하는 다양한 전략 존재

## Message Queuing

> 메시지를 임시 저장하고, 대기 중인 메시지를 순차적으로 소비자에게 전달하는 큐 제공

## 신뢰성, 내구성

> 메시지는 디스크에 저장될 수 있으므로, 서버 장애가 발생하더라도 메시지가 유실되지 않음

## 관리 인터페이스

웹 기반 관리 인터페이스를 제공하여 모니터링 및 관리 가능


## Producer / Consumer

> Producer (Publisher)
> 보내고 싶은 메시지를 대상 Queue를 지정하여 Key와 함께 Exchange로 전달

> Consumer (Subscriber)
> 받고 싶은 메시지가 들어있는 Queue를 구독(Key 필요)하다가 메시지가 들어오면 수신

## 사용방법

### Docker

> RabbitMQ Docker를 사용하여 바로 사용 가능

#### docker-compose.yaml
```
version: '3.7'
services:
  rabbitmq:
    image: rabbitmq:latest
    ports:
      - "5672:5672" #rabbit amqp port
      - "15672:15672" #manage port
    environment:
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=admin123!@#
```

### JAVA

> Producer
> 어디로 보낼 것인가? Exchange & Queue & Key
> 무엇을 보낼 것인가? Object
> 어떻게 보낼 것인가? RabbitTemplate

>Consumer
> 어디서 받을 것인가? @RabbitListener(queues="...")
> 무엇을 받을 것인가? Object
> 어떻게 받을 것인가? MessageCOnvertor(ObjectMapper)

### 코드 예제

#### Common
> application.yaml
```
spring:
  jpa:
  rabbitmq:
    host: localhost
    port: 5672
    username: admin
    password: admin123!@#
```
> build.gradle
```
dependencies {
	...
    // rabbitmq
    implementation 'org.springframework.boot:spring-boot-starter-amqp'
    ...
}
```

#### Producer
> Config
```java
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitMqConfig {

    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange("delivery.exchange");
    }

    @Bean
    public Queue queue() {
        return new Queue("delivery.queue");
    }

    @Bean
    public Binding binding(DirectExchange directExchange, Queue queue) {
        return BindingBuilder.bind(queue).to(directExchange).with("delivery.key");
    }

    @Bean
    public MessageConverter messageConverter(ObjectMapper objectMapper) {
        return new Jackson2JsonMessageConverter(objectMapper);
    }
    
    // end point 설정(import 패키지 주의!)
    // ConnectionFactory는 properties에서 매핑됨
    @Bean
    public RabbitTemplate rabbitTemplate(
	    ConnectionFactory connectionFactory, 
	    MessageConverter messageConverter) 
	{
        var rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setMessageConverter(messageConverter);
        return rabbitTemplate;
    }
}
```
> Producer.java
```java
import lombok.RequiredArgsConstructor;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.stereotype.Component;

@RequiredArgsConstructor
@Component
public class Producer {
    private final RabbitTemplate rabbitTemplate;

    public void producer(String exchange, String routeKey, Object object) {
        rabbitTemplate.convertAndSend(exchange,routeKey,object);
    }
}
```
> SendService.java
```java
import lombok.RequiredArgsConstructor;
import org.delivery.api.common.rabbitmq.Producer;
import org.delivery.common.message.model.UserOrderMessage;
import org.delivery.db.userorder.UserOrderEntity;
import org.springframework.stereotype.Service;

@RequiredArgsConstructor
@Service
public class SendService {

    private final Producer producer;
    private static final String EXCHANGE = "delivery.exchange";
    private static final String ROUTE_KEY = "delivery.key";

    public void sendOrder(UserOrderEntity userOrderEntity) {
        sendOrder(userOrderEntity.getId());
    }

    public void sendOrder(Long userOrderId) {
        var message = UserOrderMessage.builder()
                .userOrderId(userOrderId)
                .build();
        producer.producer(EXCHANGE, ROUTE_KEY, message);
    }
}
```

#### Consumer
>config
```java
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitMqConfig {

    @Bean
    public MessageConverter messageConverter(ObjectMapper objectMapper) {
        return new Jackson2JsonMessageConverter(objectMapper);
    }
}
```
> Consumer.java
```java
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.delivery.common.message.model.UserOrderMessage;
import org.delivery.storeadmin.domain.userorder.business.UserOrderBusiness;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Slf4j
@RequiredArgsConstructor
@Component
public class Consumer {

    private final UserOrderBusiness userOrderBusiness;

    @RabbitListener(queues = "delivery.queue")
    public void userOrderConsumer(UserOrderMessage userOrderMessage) {
        log.info("message queue >> {}", userOrderMessage);
        userOrderBusiness.pushUserOrder(userOrderMessage);
    }
}
```

> Producer는 단방향으로 MQ에 메시지를 적재
> RabbitTemplate 구현체를 이용
> RabbitTemplate 구현체는 ConnectionFactory와 MessageConverter 등의 객체를 참조
> ConnectionFactory
> 	MQ서버의 접속정보(properties) 
> 	binding할 Exchange / Queue / Key 정보 필요(Config에 설정)
> 메시지 발행 시, 어떤 Queue에 메시지를 발행할 것인 지 결정하고, 그 메시지에 적합한 Key 제공
> 메시지는 RabbitTemplate에서 설정한 Converter의 설정으로 변경되어 메시지 큐로 전송됨

> Consumer는 MQ에서 구독할 Queue를 설정
> 구독한 Queue에 메시지가 들어오면, 설정에서 지정한 Converter로 메시지를 변환하여 사용

