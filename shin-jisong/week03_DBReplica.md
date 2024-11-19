# DB 레플리카 구조 (READ와 WRITE DB 분리)

DB 레플리카 구조는 고가용성, 성능 향상, 데이터 손실 방지 등을 목적으로 데이터베이스의 복제본을 만드는 시스템이다.
이를 통해 애플리케이션에서 더 많은 읽기 요청을 처리할 수 있고, 장애가 발생해도 데이터 손실을 최소화할 수 있다.
주로 **마스터-슬레이브** 구조로 운영되며, 마스터 DB는 쓰기 작업을 담당하고, 슬레이브(또는 레플리카)는 읽기 작업을 담당한다.
이 구조는 수평적 확장성과 데이터 가용성을 높여주며, 대규모 시스템에서 효과적이다.

## 1. DB 레플리카 개념
레플리카 DB는 마스터 DB의 데이터를 실시간으로 복제한 복사본이다.
레플리카는 읽기 전용으로 사용되며, 데이터 일관성을 유지하기 위해 마스터 DB와 동기화된다.
주로 **비동기** 또는 **동기** 복제를 통해 데이터를 동기화한다.

- **동기 복제**는 마스터 DB에서 트랜잭션이 발생하면 슬레이브 DB로 즉시 전파하여, 슬레이브가 변경 내용을 수신한 후 트랜잭션을 완료한다. 이는 데이터 일관성을 보장하지만, 네트워크 지연이나 슬레이브의 상태에 따라 쓰기 작업이 지연될 수 있다.
- **비동기 복제**는 마스터 DB가 트랜잭션을 완료한 후 슬레이브에 데이터를 전송하는 방식이다. 이 방식에서는 데이터 전파가 지연될 수 있지만, 쓰기 성능을 향상시킨다.

## 2. 레플리카의 이점
DB 레플리카 구조는 여러 가지 장점을 가지고 있다:

- **읽기 성능 향상**: 레플리카 DB는 읽기 전용으로 사용되므로, 애플리케이션의 많은 읽기 요청을 마스터 DB 대신 처리할 수 있어 성능을 높인다.
- **고가용성**: 마스터 DB에 장애가 발생했을 때 슬레이브 DB를 마스터로 승격할 수 있어 다운타임을 줄일 수 있다.
- **백업 및 데이터 복구**: 레플리카 DB는 백업을 위한 안정적인 복사본 역할을 하며, 복구 시에도 유용하게 사용된다.

## 3. 레플리카의 동작 방식
레플리카 구조에서 마스터 DB와 슬레이브 DB는 **바이너리 로그(Binary Log)**를 통해 데이터를 동기화한다.
마스터 DB는 쓰기 작업을 바이너리 로그에 기록하고, 슬레이브 DB는 이 로그를 읽어 데이터를 동기화한다.

1. 마스터 DB에서 쓰기 작업이 발생한다.
2. 마스터 DB가 바이너리 로그에 트랜잭션을 기록한다.
3. 슬레이브 DB가 이 로그를 읽어와 동일한 트랜잭션을 실행하여 데이터를 업데이트한다.

## 4. 한계점 및 주의사항
레플리카 구조의 몇 가지 주의점은 다음과 같다:

- **데이터 일관성 문제**: 비동기 복제의 경우, 슬레이브 DB는 마스터 DB보다 최신 상태가 아닐 수 있다.
- **복제 지연**: 네트워크 지연 또는 슬레이브 DB의 성능에 따라 복제 지연이 발생할 수 있다.
- **관리 복잡성 증가**: 마스터 장애 시 슬레이브 승격 등 복잡한 관리 작업이 필요할 수 있다.

## 5. 복제 지연 문제와 해결 방법

**복제 지연(replication lag)**은 마스터 DB와 슬레이브 DB 간의 데이터 전파 속도 차이로 인해 발생하는 문제이다.
이로 인해 슬레이브 DB는 마스터 DB와 일관된 상태를 유지하지 못하게 되어, 데이터 일관성에 문제가 생길 수 있다.
이를 해결하기 위한 여러 가지 방법이 있다:

1. **WRITE DB 직접 조회**  
   복제 지연이 발생할 수 있는 중요한 요청의 경우, **마스터 DB(WRITE DB)**를 직접 조회하여 최신 데이터를 조회하는 방식으로 해결할 수 있다. 예를 들어, 사용자의 주문 처리나 중요한 트랜잭션에는 마스터 DB에서 최신 데이터를 가져와 일관성을 보장할 수 있다.

2. **캐싱(Caching) 활용**  
   자주 조회되는 데이터를 **캐시**에 저장하여 슬레이브 DB의 읽기 부하를 줄이고, 빠르게 응답할 수 있다. 캐시는 실시간으로 데이터를 최신 상태로 유지하기 어렵지만, 캐시 만료 시간과 갱신 전략을 적절히 설정하면 복제 지연 문제를 완화할 수 있다.

3. **복제 주기 조정**  
   슬레이브 DB의 복제 주기를 더 짧게 설정하여 데이터를 더 자주 동기화하도록 함으로써 복제 지연을 줄일 수 있다. 그러나 복제 주기를 너무 짧게 설정할 경우 네트워크 부하가 커질 수 있으므로, 시스템 자원을 고려한 적절한 설정이 필요하다.

4. **일관성 있는 읽기 요청 처리**  
   트랜잭션 내에서 쓰기 작업 후의 읽기 요청은 반드시 마스터 DB에서 처리하도록 하거나, **read-your-writes** 패턴을 적용하여 복제 지연으로 인한 데이터 불일치를 방지할 수 있다.


## 6. 활용 예시
DB 레플리카 구조는 다음과 같은 상황에서 유용하게 사용된다:

- **대규모 애플리케이션**: 많은 사용자가 동시에 접속하는 애플리케이션에서 레플리카 DB는 읽기 부하를 분산 처리해 성능을 향상시킨다.
- **데이터 분석**: 실시간 데이터 조회보다 주기적인 분석이 필요한 경우, 레플리카 DB를 사용하여 분석 작업을 수행하고 마스터 DB의 부하를 줄일 수 있다.
- **장애 복구**: 장애 발생 시 레플리카 DB를 통해 데이터 복구 시간을 최소화할 수 있다.

## 결론
DB 레플리카 구조는 대규모 트래픽을 처리하고, 데이터 가용성을 높이며, 확장성을 제공하는 중요한 시스템 아키텍처다.
하지만 복제 지연과 데이터 일관성 문제를 해결하기 위해 마스터 DB 직접 조회, 캐싱 등의 해결책을 함께 고려해야 한다.