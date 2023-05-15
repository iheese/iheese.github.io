---
layout: post
title: '[INFRA] Redis Stream vs Kafka'
subtitle: 'Redis, Redis Stream, Kafka'
date: 2023-05-15 11:00:00 +0900
categories: 'INFRA'
background: '/img/posts/etc/datastructure.jpg'
---

## Redis Stream
- Redis 5.0 부터 추가된 데이터 스트리밍 시스템
- 시계열 데이터의 시퀀스를 유지하는 추가 전용(append only) 로그 데이터 구조
- 데이터 일관성, 안정성 보장, 대용량 데이터 스트림 실시간 처리
- 초당 수백만 개의 메시지를 처리할 수 있는 대기 시간이 짧은 고가용성 시스템

> - In-memory Data Storage: Redis Stream는 메모리(RAM)에 데이터를 저장하므로 디스크 스토리지를 사용하는 다른 데이터 스트리밍 시스템보다 빠름
> - 내장 데이터 처리 기능: Redis Stream에는 필터링, 집계 및 패턴 일치와 같은 여러 데이터 처리 기능이 포함
> - 여러 클라이언트 지원: Redis Streams는 여러 클라이언트를 지원하므로 다른 시스템과 쉽게 통합 가능

<br>

## Apache Kafka
- 대용량 데이터 스트림을 처리할 수 있는 분산 데이터 스트리밍 시스템
- 초당 수백만 개의 데이터를 처리할 수 있고 디스크 저장소를 사용하므로 일부 대기 시간 문제가 발생할 수 있다. 
- Producer가 kafka topic에 데이터 Publish하고 해당 topic을 Subscribe하는 Consumer가 존재하는 구조

> - 분산 아키텍처: Apache Kafka는 여러 노드에서 실행할 수 있는 분산 시스템으로 가용성이 높고 내결함성이 있음
> - 확장성: Apache Kafka는 클러스터에 더 많은 노드를 추가하여 수평으로 확장할 수 있음
> - 데이터 보존: Apache Kafka는 구성 가능한 기간 동안 데이터를 보존할 수 있으므로 소비자가 데이터 스트림을 재생할 수 있음

<br>

## Redis vs Kafka

1 . 원산지
- Redis : 실시간 웹로그 분석을 위해 Salvatore Sanfilippo가 개발
- Kafka : LinkedIn에서 개발한 오픈 소스 소프트웨어 플랫폼

2 . 구독 방식
- Redis : Push 기반 구독, Redis에 게시된 메시지가 구독자에게 자동으로 전달됨
- Kafka : Pull 기반 구독, Kafka에 게시된 메시지는 소비자가 Topic을 구독하고 메시지를 처리할 준비가 되었을 때 메시지를 가져옴

3 . 병렬성
- Redis : 병렬성 개념을 지원하지 않음

> - Redis의 consumer group
> > -  Redis와 Kafka의 consumer group는 다른 개념임을 알아야 한다.
> > - Redis의 메시지는 모두 다른 소비자에게 제공된다. 그 소비자들을 묶어놓은 것이 consumer group이다.
> > - 1,2,3,4,5,6,7 메시지 데이터가 스트림에 들어있고, C1, C2, C3 소비자가 있을 때 아래와 같은 처리를 만들기 위해 consumer group을 개념을 도입하였다. 

```
1 -> C1
2 -> C2
3 -> C3
4 -> C1
5 -> C2
6 -> C3
7 -> C1
```

> > - consumer group은 소비되지 않은 메시지들을 추적하고 첫 번쨰 ID의 개념을 가지고 있다. 

- Kafka : 병렬성 개념 지원, 여러 Counsumer 가 Consumer group에 속하여 동시에 로그 데이터를 분할하여 병렬 처리함

> - kafka의 consumer group 
> > - 한 소비자가 장애가 나서 다운되더라도 다른 소비자가 존재하면 장애를 대응할 수 있다. 
> > - 같은 토픽을 구독하고 있더라도 그룹들은 각각 offset를 가지고 있어서 자신이 어디까지 소비하였는지 알 수 있다.

4 . 메시지 보존
- Redis : 소비자가 메시지를 보내면 메시지가 나중에 제거됨, 메모리 저장
- Kafka : 로그이기 때문에 항상 메시지가 존재하고 메시지에 대한 보존 정책을 설정하여 모니터링 가능, 디스크 저장

5 . 속도
- Redis : 메모리 저장 기반이므로 메시지 전달 속도가 빠름
- Kafka : 메시지를 전달한 후에도 메시지가 유지되고 디스크 기반이므로 비교적 느림

6 . 데이터 양
- Redis : 메모리에 저장되므로 많은 양의 데이터를 오래 저장할 수 없음
- Kafka : 디스크에 저장되므로 대량의 데이터를 처리할 수 있고 필요한 만큼 서버를 추가 할 수 있음, 그러나 디스크를 사용하므로 로드 속도가 느릴 수  있음

7 . 범주
- Redis : 인메모리 데이터베이스
- Kafka : 메시지 큐

8 . 사용 사례 
- Redis : Session Cache, Full Page Cache (FPC), Leader Boards/Counting, Pub s/Sub, Queues 등 다양한 사용 사례 존재
- Kafka : Kafka에는 메시징, 웹사이트 활동 추적, 로그 집계, 스트림 처리, 지표, 이벤트 소싱, 커밋 로그와 같은 다양한 사용 사례 존재

<br>

Reference : 
- [Redis Stream (레디스 스트림) 기본 정리](https://kingjakeu.github.io/page2/)
- [kafka-vs-redis](https://logz.io/blog/kafka-vs-redis/)
- [Kafka 운영자가 말하는 Kafka Consumer Group](https://www.popit.kr/kafka-consumer-group/)
- [redis 공식문서](https://redis.io/docs/data-types/streams-tutorial/#consumer-groups)
