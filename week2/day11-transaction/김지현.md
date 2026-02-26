# Day 11: 트랜잭션(Transaction)과 ACID

> 📅 2025.02.23 | 📁 Week 2

---

## 🧠 학습 질문

-   [ ] 트랜잭션이란 무엇이며 왜 필요한가?
-   [ ] ACID 속성(원자성, 일관성, 격리성, 지속성)을 각각 설명하면?
-   [ ] 트랜잭션 격리 수준(Read Uncommitted, Read Committed, Repeatable Read, Serializable)의 차이는?
-   [ ] Dirty Read, Non-Repeatable Read, Phantom Read란 무엇인가?
-   [ ] 각 격리 수준에서 어떤 문제가 발생하고 방지되는가?
-   [ ] 락(Lock)의 종류(공유 락, 배타 락)와 역할은?
-   [ ] 2PL(Two-Phase Locking) 프로토콜이란?

## 1. 트랜잭션(Transaction)이란 무엇이며 왜 필요한가?

**트랜잭션(Transaction)**
→ 데이터베이스에서 **하나의 논리적 작업 단위(Logical Unit of Work)**

예:

-   계좌 이체

    1. A 계좌에서 10,000원 차감
    2. B 계좌에 10,000원 증가

이 두 연산은 **반드시 함께 성공하거나 함께 실패**해야 한다.

### 왜 필요한가?

1. **데이터 무결성 보장**
2. **동시성 환경에서 일관성 유지**
3. **장애 발생 시 복구 가능**

트랜잭션이 없다면:

-   차감만 되고 입금은 실패 → 데이터 불일치
-   동시 접근 시 값이 꼬임

## 2. ACID 속성

### 1) Atomicity (원자성)

> All or Nothing

-   트랜잭션은 전부 성공하거나 전부 실패해야 한다.
-   일부만 반영되는 것은 허용되지 않는다.
-   실패 시 Rollback

### 2) Consistency (일관성)

-   트랜잭션 전후에 **DB는 항상 유효한 상태**
-   제약조건(Primary Key, Foreign Key, Unique 등)을 위반하지 않는다.

예:

-   잔액이 음수가 되지 않는다
-   FK 참조 무결성 유지

### 3) Isolation (격리성)

-   동시에 실행되는 트랜잭션들이 서로 간섭하지 않도록 보장
-   마치 **순차적으로 실행된 것처럼 보이게 한다**

### 4) Durability (지속성)

-   Commit된 결과는 시스템 장애가 나도 보존된다.
-   WAL, Redo Log, fsync 등을 통해 보장

---

## 3. 트랜잭션 격리 수준 (Isolation Level)

### 1) Read Uncommitted

-   다른 트랜잭션의 **커밋되지 않은 데이터도 읽을 수 있음**
-   Dirty Read 발생 가능

가장 낮은 격리 수준

### 2) Read Committed

-   **Commit된 데이터만 읽을 수 있음**
-   Dirty Read 방지
-   Non-Repeatable Read 가능

PostgreSQL 기본값

### 3) Repeatable Read

-   한 트랜잭션 내에서 **같은 row는 항상 동일한 값으로 읽힘**
-   Dirty Read 방지
-   Non-Repeatable Read 방지
-   Phantom Read 가능 (DBMS에 따라 다름)

MySQL(InnoDB) 기본값

### 4) Serializable

-   모든 트랜잭션이 **완전히 순차 실행된 것처럼 동작**
-   모든 이상현상 방지
-   성능 가장 낮음

## 4. 이상 현상 (Anomalies)

### 1) Dirty Read

-   커밋되지 않은 데이터를 읽는 것
-   나중에 롤백되면 잘못된 값을 읽은 것

### 2) Non-Repeatable Read

-   같은 row를 두 번 읽었는데 값이 달라짐
-   중간에 다른 트랜잭션이 수정 후 commit

### 3) Phantom Read

-   같은 조건으로 조회했는데
-   두 번째 조회에서 **행(row)이 추가/삭제됨**

예:

```
SELECT * FROM users WHERE age > 20;
```

중간에 다른 트랜잭션이 age=25 데이터 insert

## 5. 격리 수준별 발생 가능 문제

| 격리 수준        | Dirty Read | Non-Repeatable | Phantom |
| ---------------- | ---------- | -------------- | ------- |
| Read Uncommitted | O          | O              | O       |
| Read Committed   | X          | O              | O       |
| Repeatable Read  | X          | X              | O       |
| Serializable     | X          | X              | X       |

## 6. Lock의 종류

DB는 **동시성 제어를 위해 Lock 사용**

### 1) Shared Lock (공유 락, S-lock)

-   읽기용 락
-   여러 트랜잭션이 동시에 획득 가능
-   다른 트랜잭션이 쓰기 불가

→ 읽기-읽기 가능
→ 읽기-쓰기 불가

### 2) Exclusive Lock (배타 락, X-lock)

-   쓰기용 락
-   한 트랜잭션만 획득 가능
-   다른 모든 접근 차단

→ 쓰기-읽기 불가
→ 쓰기-쓰기 불가

### Lock 호환성

| 요청 ↓ / 보유 → | S    | X    |
| --------------- | ---- | ---- |
| S               | 가능 | 불가 |
| X               | 불가 | 불가 |

## 7. 2PL (Two-Phase Locking)

**Two-Phase Locking Protocol**

트랜잭션이 다음 두 단계로 나뉨:

### 1단계: Growing Phase

-   Lock 획득만 가능
-   Lock 해제 불가

### 2단계: Shrinking Phase

-   Lock 해제만 가능
-   새로운 Lock 획득 불가

즉:

> Lock을 더 이상 획득하지 않는 순간부터는 해제만 가능

### 왜 필요한가?

2PL을 따르면 **Conflict-Serializable** 스케줄이 보장됨
→ 직렬 실행과 동일한 결과 보장

### Strict 2PL

-   모든 X-lock을 Commit 시점까지 유지
-   Cascading Abort 방지
-   대부분의 DBMS가 채택

---

## 📎 참고 자료

## <!-- 공부하면서 참고한 링크를 여기에 추가해주세요 -->

## 💬 토론 포인트

<!-- PR 리뷰 또는 스터디 중 나온 추가 질문이나 논의 사항을 기록해주세요 -->
