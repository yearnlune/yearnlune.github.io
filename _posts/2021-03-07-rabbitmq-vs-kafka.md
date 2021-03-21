---
tags: 
    - rabbitmq
    - kafka
title: Rabbitmq vs Kafka
date: 2021/03/07
author: 김동환
description:  Rabbitmq vs Kafka
disabled: false
categories:
  - general
---

Rabbitmq와 Kafka 둘 다 Producer와 Consumer간에 message(event)를 전달하는 시스템으로 안정적이고 확장 가능한 메시징 시스템이다. 성능적인 측면에서는 Rabbitmq에 비해 Kafka가 약 15배 정도 빠르게 처리한다고 한다. 그리하여 많은 양의 데이터를 수집, 저장, 처리해야 하는 경우 강력한 분산 시스템인 Kafka의 사용을 고려해야 한다. 성능 외적인 각각 특징을 통해서 장단점을 살펴보고, 어떤 시스템에 더 적합한지 알아보려고 한다.

　
# 메시지 리플레이

## Rabbitmq

consumer가 queue에 연결하여 메시지를 수신하거나, 수신후 Ack를 통해서 확인을 받은 후 해당 queue에서 메시지가 제거된다.

## Kafka

큐의 지정된 기간이 경과되거나 및 크기가 초과되기 전까지 메시지가 지속적으로 저장된다.

메시지의 replay가 필요로한 서비스에 적용하기 적합하다.

　
# 프로토콜

## Rabbitmq

`AMQP`, `STOMP`, `MQTT` 등과 같은 여러 표준 프로토콜을 지원해준다. 그리하여 지원하는 표준 프로토콜을 사용하면 대체 가능하다.

## Kafka

TCP / IP  위의 custom 프로토콜을 사용한다. 표준화된 프로토콜을 사용하지 않기에 쉽게 대체하기 힘들다.

　
# 라우팅

## Rabbitmq

`Direct`, `Fanout`, `Topic`, `Headers`의 라우팅 옵션을 제공하여 유연하게 라우팅이 가능하다.

`Direct`는 unicast 방식처럼 라우팅 키가 정확히 일치하는 큐에 메시지를 라우팅해준다. `Fanout`은 Broadcast 방식처럼 라우팅 키와 상관없이 모든 큐에 메시지를 라우팅해준다. `Topic`은 라우팅 키의 패턴(Regex)에 부합하는 큐에 메시지를 라우팅해준다. `Headers`는 `x-match` 속성인 [any, all]을 통해서 메시지 헤더의 키와 값의 일치 정도에 따라 라우팅해준다.

## Kafka

기본기능으로 라우팅에 대해서 지원하지 않는다. Kafka Streams를 활용하여 동적라우팅을 구현할 수 있다.

　
# 메시지 우선 순위

## Rabbitmq

priority queue를 지원하여 `x-max-priority`  argument를 통해서 우선 순위에 따라서 우선 처리가 가능하다.

## Kafka

priority queue를 지원하지 않으며, 변경 불가능한 시퀀스 큐로, 시간 순서를 보장한다.

　
# 응답

## Rabbitmq

producer의 ack를 지원하여 해당 브로커에 메시지가 안전하게 producing되었는 지를 알 수있다. consumer도 마찬가지로 메시지가 안전하게 클라이언트에게 전달되었는지를 확인할 수 있다. 추가적으로 수동 ack를 활용하면 메시지에 대한 완전한 처리도 관리할 수 있다.

## Kafka

Rabbitmq와 마찬가지로 producer의 ack를 지원하며, consumer도 offset을 통해 메시지의 처리 를 추적할 수 있으며, 수동으로 제어도 가능하다.

　
# 스케일링

## Rabbitmq

많은 활성화된 consumer들이 경쟁을 통해서 메시지를 처리가능하며, 단순히 consumer의 추가 및 제거로 관리할 수 있다. 브로커는 수평적으로 확장을 하더라도 항상 더 나은 성능을 제공해주지 못 한다.

## Kafka

consumer들을 하나 이상의 파티션에 할당하여 스케일링하며, 브로커는 클러스터에 더 많은 노드를 추가하거나 토픽에 더 많은 파티션을 추가하여 확장 할 수 있다.  Rabbitmq보다 더 큰 규모의 확장이 이뤄질 수 있다.

　
# 참고문헌

[https://www.rabbitmq.com/](https://www.rabbitmq.com/)

[https://kafka.apache.org/](https://kafka.apache.org/)

[https://www.confluent.io/blog/kafka-fastest-messaging-system](https://www.confluent.io/blog/kafka-fastest-messaging-system/)

[https://www.cloudamqp.com/blog/2019-12-12-when-to-use-rabbitmq-or-apache-kafka.html](https://www.cloudamqp.com/blog/2019-12-12-when-to-use-rabbitmq-or-apache-kafka.html)