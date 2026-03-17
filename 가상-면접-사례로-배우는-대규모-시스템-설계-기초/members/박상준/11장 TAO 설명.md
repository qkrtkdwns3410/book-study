# 11장 TAO 설명

## TAO 가 뭐냐

- 메타(페이스북)에서 소셜 그래프 처리용으로 만든 저장 계층임
- MySQL 위에 올린 `관계 조회 특화 캐시 + 저장소` 임

## 왜 필요했음

- 페이스북은 관계 조회가 엄청 많음 (친구 목록, 좋아요 목록 등)
- 일반 key-value 캐시만으로는 감당이 안 됐음

## 데이터 2종류

### object

- 사용자, 포스트, 댓글 같은 실체 데이터임

### association

- 엔티티 간 연결 정보임 (친구, 좋아요, 댓글 연결 등)
- `(id1, type, id2, time)` 구조로 저장됨

## 주요 연산

- object 읽기 (사용자 정보 조회)
- association 확인 (A가 B를 친구 추가했는지)
- association 목록 조회 (친구 목록, 좋아요 목록)
- 개수 세기 (좋아요 수)
- 범용 그래프 DB가 아니라 `SNS 빈출 조회 패턴만 빠르게 처리`하는 쪽임

## 구조

```
앱 서버 → Follower cache → Leader cache → MySQL
```

- follower cache: 앱 서버가 제일 먼저 붙는 캐시, 읽기 요청을 여기서 최대한 끝냄
- leader cache: follower에 없을 때 가는 2차 캐시, DB에 더 가까운 계층
- 2단계로 나눈 이유: `DB 보호` + `캐시 자체 병목 완화`

## MySQL은 버린게 아님

- MySQL이 영구 저장소 역할임
- TAO는 그 앞단에서 캐시 + 관계 조회 가속 + 데이터 접근 단순화 해주는 계층임

## 뉴스 피드 연결

- 뉴스 피드도 결국 관계 조회(친구/팔로우 목록)가 핵심이라 TAO 구조가 잘 맞음

## 샤딩

- 데이터를 여러 shard로 쪼개서 분산 저장함
- 한 서버에 몰리는 걸 방지하는 용도임

## 한 줄 정리

> TAO = 페이스북에서 친구/좋아요 같은 연결 정보를 빠르게 읽으려고 만든 관계 중심 저장/캐시 계층

## 참고 링크

- [TAO: The power of the graph - Meta Engineering](https://engineering.fb.com/2013/06/25/core-infra/tao-the-power-of-the-graph/)
- [TAO: Facebook's Distributed Data Store for the Social Graph - USENIX](https://www.usenix.org/system/files/conference/atc13/atc13-bronson.pdf)
