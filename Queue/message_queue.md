# Message Queue

- 데이터를 전달 및 처리할 때 중간에 임시 공간을 두어 데이터를 쌓아두고 처리하는 것
- 큐에 데이터를 넣어두고 비동기로 처리하기 때문에 바로 처리가 필요하지 않은 대규모 데이터, 트래픽 처리에 적합
- 메시지 지향 미들웨어 (MOM: Message Oriented Middleware)를 구현한 시스템으로 프로그램 간의 데이터를 교환할 때 사용하는 기술
- 메시지 큐 시스템 : Kafka, RabbitMQ, Redis Pub/Sub 등

## 메시지 브로커, 이벤트 브로커

### 메시지 브로커

- `publisher`가 생산한 메시지를 메시지 큐에 저장하고 저장된 데이터를 `consumer`가 가져갈 수 있도록 중간 다리 역할을 해주는 브로커
- 서로 다른 시스템 사이에서 데이터를 비동기 형태로 처리하기 위해서 사용함
- `consumer`가 큐에서 데이터를 가져가게 되면 즉시 혹은 짧은 시간 내에 큐에서 데이터가 삭제됨
- ex. Redis, RabbitMQ, SQS 등

### 이벤트 브로커

- 메시지 브로커의 큐 기능들을 가지고 있어서 메시지 브로커 역할도 가능함
- `publisher`가 생산한 이벤트를 처리 후에 바로 삭제하지 않고 저장, 이벤트 시점이 저장되어 있어서 `consumer`가 특정 시점부터 이벤트를 다시 consume할 수 있음
- 대용량 처리에 있어서는 메시지 브로커보다 더 많은 양의 데이터를 처리할 수 있음
- ex. kafka 등

## Pub/Sub 모델

- 비동기 메시징 전송 방식
- 발신자의 메시지에는 수신자가 정해져있지 않은 상태로 publish함 → 이를 Subscribe(구독)한 수신자만 정해진 메시지를 받을 수 있음

→ 높은 확장성을 확보

### Redis

- 구성
  - `publisher`: 메세지를 게시(pub)
  - channel: `publisher`가 메시지를 발행하고, `subscriber`가 해당 메시지를 실시간으로 수신하기 위한 통신 경로
  - `subscriber`: 메세지를 구독(sub)
- 동작
  - `publisher`가 channel에 메세지 게시
  - 해당 채널을 구독하고 있는 `subscriber`가 메세지를 sub해서 처리함
- 특징
  - 채널은 이벤트를 저장하지 않음
  - 채널에 이벤트가 도착했을 때 해당 채널의 `subscriber`가 존재하지 않는다면 이벤트가 사라짐
  - `subscriber`는 동시에 여러 채널을 구독할 수 있으며 특정 채널을 지정하지 않고 패턴을 설정하여 해당 패턴에 맞는 채널을 구독할 수 있음
- 장점
  - 빠른 처리 속도
  - 캐시 역할도 가능
  - 명시적으로 데이터 삭제 가능
- 단점
  - 메모리 기반으로 서버가 다운되면 Redis 내의 모든 데이터가 사라짐
  - 이벤트 도착 보장 불가

### RabbitMQ

- AMQP 프로토콜을 구현한 메시지 브로커
- 구성 요소
  - producer: 메세지를 보냄
  - exchange: 메세지를 목적지(큐)에 맞게 전달
  - queue: 메세지를 쌓아둠
  - consumer: 메세지를 받음
- 메세지 처리 과정
  - `producer` 가 `broker`로 메세지를 보냄
  - `broker` 내 Exchange(메세지 교환기) 에서 해당하는 key에 맞게 큐에 분배. (Binding or Routing 이라고 함)
    - topic 모드 : Routing Key가 정확히 일치하는 Queue에 메세지 전송 (Unicast)
    - direct 모드 : Routing Key 패턴이 일치하는 Queue에 메세지 전송 (Multicast)
    - headers 모드 : [Key:Value] 로 이루어진 header값을 기준으로 일치하는 Queue에 메세지 전송 (Multicast)
    - fanout 모드 : 해당 Exchange에 등록도니 모든 Queue에 메세지 전송 (Broadcast)
  - 해당 큐에서 Consumer가 메세지를 받는다.
- 장점
  - `broker` 중심적인 형태로 `publisher`와 `consumer` 간의 보장되는 메시지 전달에 초점을 맞추고 복잡한 라우팅 지원
  - 클러스터 구성이 쉽고 Manage UI가 제공되며 플러그인도 제공되어 확장성이 뛰어남
  - 20kb/sec정도의 속도
  - 데이터 처리 보단 관리적 측면이 다양한 기능 구현을 위한 서비스를 구축할 때 사용
- 단점
  - MQ Server가 종료 후 재가동 되면 기본적으로 Queue 내용이 모두 제거됨
  - 성능
  - `producer와` `consumer` 결합도가 높음

### kafka

- 구성 요소
  - Event: kafka에서 `producer`와 `consumer`가 데이터를 주고받는 단위. 메세지
  - Producer: kafka에 이벤트를 게시(post, pop)하는 클라이언트 어플리케이션
  - Consumer: Topic을 구독하고 이로부터 얻어낸 이벤트를 받아(Sub) 처리하는 클라이언트 어플리케이션
  - Topic: 이벤트가 모이는 곳. `producer`는 topic에 이벤트를 게시하고, `consumer`는 topic을 구독해 이로부터 이벤트를 가져와 처리. 게시판 같은 개념
  - Partition: Topic은 여러 Broker에 분산되어 저장되며, 이렇게 분산된 topic을 partition이라고 함
  - ZooKeeper: 분산 메세지의 큐의 정보를 관리
- 동작 원리
  - `publisher`는 전달하고자 하는 메세지를 topic을 통해 카테고리화 한다.
  - `subscriber`는 원하는 topic을 구독(=subscribe)함으로써 메시지를 읽어온다.
  - `publisher`와 `subscriber`는 오로지 topic 정보만 알 뿐, 서로에 대해 알지 못한다.
  - kafka는 broker들이 하나의 클러스터로 구성되어 동작하도록 설계
  - 클러스터 내, broker에 대한 분산처리는 ZooKeeper가 담당한다.
- 장점
  - 대규모 트래픽 처리 및 분산 처리에 효과적
  - 클러스터 구성, Fail-over, Replication 같은 기능이 있음
  - 100Kb/sec 정도의 속도 (다른 메세지 큐 보다 빠름)
  - 디스크에 메세지를 특정 보관 주기동안 저장하여 데이터의 영속성이 보장되고 유실 위험이 적음. 또한 Consumer 장애 시 재처리가 가능
