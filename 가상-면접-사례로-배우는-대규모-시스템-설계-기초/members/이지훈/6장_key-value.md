## 1. Key-Value Store란 무엇인가

Key-Value Store는 데이터를 (key, value) 쌍으로 저장하는 가장 단순한 형태의 NoSQL 저장소이다.  
데이터 접근은 오직 key를 통해 이루어지며, 기본 연산은 다음 두 가지다.

- put(key, value)
- get(key)

이 단순한 모델 덕분에 복잡한 조인이나 스키마 제약이 없고,  
대규모 트래픽 환경에서 매우 빠른 읽기/쓰기를 제공할 수 있다.


---

## 2. 단일 서버 Key-Value Store의 한계

가장 단순한 구현은 한 대의 서버에 해시 테이블로 데이터를 저장하는 방식이다.

장점:
- 매우 빠른 응답 속도
- 구현이 단순함

한계:
- 메모리 용량 한계
- 서버 장애 시 전체 서비스 중단
- 트래픽 증가에 따른 수평 확장 불가

---

## 3. 분산 Key-Value Store의 핵심 문제

분산 환경에서 Key-Value Store를 설계할 때 반드시 해결해야 하는 문제는 다음과 같다.

1. 데이터를 어떤 노드에 저장할 것인가 (Partitioning)
2. 노드 장애 시 데이터는 어떻게 보호할 것인가 (Replication)
3. 데이터 일관성과 가용성 중 무엇을 우선할 것인가 (CAP)
4. 노드가 추가/제거될 때 데이터 재배치는 어떻게 최소화할 것인가

DynamoDB는 이를 실제 시스템으로 구현한 사례다.

---

## 4. DynamoDB 

DynamoDB는 AWS가 제공하는 완전관리형 Key-Value / Document NoSQL 데이터베이스다.

특징:
- 자동 파티셔닝
- 자동 복제
- 서버리스 운영
- 선택 가능한 일관성 모델

DynamoDB는 Amazon Dynamo 논문에서 제시한 개념을 기반으로,  
실제 상용 서비스에 맞게 고도화된 시스템이다.

---

## 5. 데이터 분산: Partition Key와 해시 기반 파티셔닝

DynamoDB는 테이블의 Partition Key를 내부 해시 함수에 입력하여  
해당 아이템이 저장될 파티션을 결정한다.

흐름:
1. Partition Key 값 입력
2. 내부 해시 함수 계산
3. 해시 결과에 따라 파티션 결정
4. 해당 파티션에 데이터 저장

---

## 6. 파티션 내부 구조

DynamoDB는 단순히 해시만 사용하는 것이 아니라,  
파티션 내부에서는 B-Tree 계열 구조를 사용해 데이터를 관리한다.
- https://adamrackis.dev/blog/dynamo-introduction

구조 요약:
- 외부: 해시 기반 파티셔닝
- 내부: 정렬/검색 효율을 위한 트리 구조

이를 통해 단일 파티션 내에서도 빠른 검색과 범위 처리가 가능하다.

---

## 7. 복합 키 설계: Partition Key + Sort Key

DynamoDB는 다음 두 가지 Primary Key 구조를 지원한다.

1. Partition Key 단일 키
2. Partition Key + Sort Key 복합 키

복합 키를 사용하면:
- 같은 Partition Key 내에서 여러 아이템 저장 가능
- Sort Key 기준 정렬 및 범위 조회 가능

 예시 1: Partition Key만 사용하는 경우

```
Table: Users
Partition Key: userId
```

```
userId = 1001 → 사용자 정보
userId = 1002 → 사용자 정보
```

이 구조의 특징:
- Partition Key 값은 유일해야 한다.
- 같은 userId로는 하나의 아이템만 저장 가능하다.
- 단일 객체(사용자, 설정 값 등) 저장에 적합하다.
- 한 사용자 아래에 여러 데이터를 저장하기에는 부적합하다.

예시 2: Partition Key + Sort Key를 사용하는 경우

```
Table: Orders
Partition Key: userId
Sort Key: orderId
```

```
(userId=1001, orderId=20240101) → 주문 A
(userId=1001, orderId=20240115) → 주문 B
(userId=1001, orderId=20240203) → 주문 C
```

이 구조의 특징:
- 같은 Partition Key(userId) 아래에 여러 아이템 저장 가능
- Sort Key(orderId)가 다르면 서로 다른 아이템으로 저장된다
- Sort Key 기준으로 내부 정렬이 유지된다


Sort Key를 활용한 범위 조회 예시

```
userId = 1001
AND orderId BETWEEN 20240101 AND 20240131
```

이 쿼리는 다음을 의미한다:
- 특정 사용자(userId)의 데이터 중
- 특정 범위의 Sort Key(orderId)에 해당하는 데이터만 조회

이러한 범위 조회는 Partition Key 단일 구조에서는 불가능하다.



---

## 8. 복제와 장애 대응

DynamoDB는 각 파티션을 여러 노드에 복제하여 저장한다.

- 기본적으로 3개 이상의 복제본 유지
- 하나의 노드 장애 시에도 서비스 지속 가능

쓰기 요청은 리더 노드에 먼저 반영된 후,
다른 복제 노드로 전파된다.

이는 6장에서 설명하는 Replication 기반 가용성 확보 전략의 실제 구현이다

---

## 9. 일관성 모델과 CAP 정리

DynamoDB는 두 가지 읽기 일관성 옵션을 제공한다.

Eventually Consistent Read (기본)

Strongly Consistent Read

이는 CAP 정리 관점에서 다음과 같이 해석할 수 있다.

Partition Tolerance(P)는 분산 시스템 특성상 항상 유지된다

따라서 Consistency와 Availability 중 하나를 선택해야 하며,
이 선택을 사용자에게 위임한다

DynamoDB는 리전 내 다수의 노드에 데이터를 분산 저장하는 구조이기 때문에
네트워크 분리(Partition)는 필연적으로 발생할 수밖에 없다.
즉, P는 설계상 고정값이다..

그 결과 DynamoDB는 읽기 시점마다:

- 가용성을 우선해 항상 응답을 보장할 것인지 (Eventually Consistent Read)
- 일관성을 우선해 최신 데이터를 보장할 것인지 (Strongly Consistent Read)

를 선택하도록 설계되었다.

정리하자면, P는 못바꾸니 성능 중시할래, 일관성 중시할래. 어쩔수 없는 형태.

---

## 10. 자동 확장과 운영 단순화

6장에서는 분산 시스템의 운영 복잡성이 큰 문제라고 지적한다.

DynamoDB는 이를 다음 방식으로 해결한다.

- 파티션 자동 분할
- 트래픽 증가 시 자동 확장
- 노드 장애 자동 복구
- 운영자 개입 최소화

이에따라, 개발자의 개입이 적기 때문에 운영 복잡성 측에선 매우 큰 이점을 가진다.

