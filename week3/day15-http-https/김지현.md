# Day 15: HTTP와 HTTPS

> 📅 2025.02.27 | 📁 Week 3

---

## 🧠 학습 질문

-   [x] HTTP는 어떻게 동작하는가? Stateless의 의미는?
-   [x] HTTP 메서드(GET, POST, PUT, PATCH, DELETE)의 차이와 멱등성은?
-   [x] HTTP 상태 코드의 의미는? (2xx, 3xx, 4xx, 5xx) 주요 코드는?
-   [x] HTTP/1.1과 HTTP/2의 차이는? HTTP/2가 더 빠른 이유는?
-   [x] HTTP/3는 무엇이 다른가? QUIC 프로토콜이란?
-   [x] HTTPS는 어떻게 보안을 제공하는가? 대칭키와 비대칭키의 역할은?
-   [x] SSL/TLS 핸드셰이크는 어떻게 동작하는가?

## 1. HTTP는 어떻게 동작하는가? Stateless의 의미는?

### HTTP(HyperText Transfer Protocol) 동작 구조

-   **Application Layer Protocol**
-   기본 구조: **Client–Server 모델**
-   통신 방식: **Request → Response**

#### 동작 흐름

1. Client가 TCP 연결 생성 (기본 포트 80)
2. HTTP Request 전송

    - Start Line (Method + URI + Version)
    - Header
    - Body (선택)

3. Server가 요청 처리
4. HTTP Response 반환

    - Status Line (Version + Status Code)
    - Header
    - Body

HTTP는 텍스트 기반 프로토콜이며, TCP 위에서 동작한다.

---

### Stateless의 의미

HTTP는 **Stateless Protocol**이다.

-   서버는 각 요청을 **독립적인 요청**으로 처리
-   이전 요청의 상태를 기억하지 않음

예시:

```
1. 로그인 요청
2. 상품 조회 요청
```

서버는 2번 요청만 보고는 사용자가 로그인했는지 알 수 없다.

이를 해결하기 위해 사용되는 기술:

-   Cookie
-   Session
-   JWT
-   Token 기반 인증

---

## 2. HTTP 메서드와 멱등성(Idempotency)

### 주요 메서드 비교

| Method | 목적             | Body | 멱등성   |
| ------ | ---------------- | ---- | -------- |
| GET    | 리소스 조회      | 없음 | O        |
| POST   | 리소스 생성      | 있음 | X        |
| PUT    | 리소스 전체 수정 | 있음 | O        |
| PATCH  | 리소스 일부 수정 | 있음 | X (보통) |
| DELETE | 리소스 삭제      | 없음 | O        |

---

### 멱등성(Idempotency)

같은 요청을 여러 번 보내도 결과가 동일한 성질.

예시:

-   PUT /users/1 → 같은 데이터로 여러 번 호출해도 결과 동일
-   DELETE /users/1 → 여러 번 호출해도 삭제 상태 유지

POST는 일반적으로 멱등하지 않다.
→ 여러 번 호출하면 리소스가 중복 생성될 수 있음

---

## 3. HTTP 상태 코드

### 1xx: Informational

-   100 Continue

### 2xx: Success

-   200 OK
-   201 Created
-   204 No Content

### 3xx: Redirection

-   301 Moved Permanently
-   302 Found
-   304 Not Modified

### 4xx: Client Error

-   400 Bad Request
-   401 Unauthorized
-   403 Forbidden
-   404 Not Found

### 5xx: Server Error

-   500 Internal Server Error
-   502 Bad Gateway
-   503 Service Unavailable

---

## 4. HTTP/1.1 vs HTTP/2

### HTTP/1.1 특징

-   요청/응답은 텍스트 기반
-   한 연결에서 하나의 요청만 처리 (순차 처리)
-   Head-of-Line Blocking 발생
-   Keep-Alive로 연결 재사용 가능

문제점:

-   여러 리소스 요청 시 지연 증가
-   병렬 처리 위해 여러 TCP 연결 필요

---

### HTTP/2 특징

-   Binary Protocol
-   Multiplexing 지원
-   Header Compression (HPACK)
-   Server Push 지원

### HTTP/2가 더 빠른 이유

1. 하나의 TCP 연결에서 여러 요청 동시 처리
2. Head-of-Line Blocking 완화
3. 헤더 압축으로 오버헤드 감소
4. 불필요한 TCP 연결 감소

---

## 5. HTTP/3와 QUIC

### HTTP/3 특징

-   TCP 대신 **QUIC(UDP 기반)** 사용
-   연결 설정 속도 개선
-   Head-of-Line Blocking 완전 제거

---

### QUIC 프로토콜

-   UDP 기반 전송 프로토콜
-   TLS 1.3 내장
-   빠른 핸드셰이크 (0-RTT 가능)
-   스트림 단위 독립 처리

TCP는 패킷 하나가 손실되면 전체 대기
QUIC은 스트림 단위로 독립 처리 가능

→ 지연 감소

---

## 6. HTTPS의 보안 구조

HTTPS = HTTP + TLS

보안 제공 요소:

1. 기밀성 (Confidentiality)
2. 무결성 (Integrity)
3. 인증 (Authentication)

---

### 대칭키 vs 비대칭키

#### 비대칭키

-   공개키 + 개인키
-   서버 인증에 사용
-   키 교환에 사용

#### 대칭키

-   동일한 키 사용
-   실제 데이터 암호화에 사용
-   속도가 빠름

흐름:

1. 비대칭키로 대칭키 교환
2. 이후 통신은 대칭키로 암호화

---

## 7. SSL/TLS Handshake 과정

(기본 TLS 1.2 기준)

1. Client Hello

    - 지원 암호 스위트
    - 랜덤 값

2. Server Hello

    - 선택된 암호 스위트
    - 서버 랜덤 값
    - 서버 인증서 전달

3. 서버 인증서 검증

    - CA 검증
    - 공개키 확인

4. Pre-Master Secret 생성

    - 클라이언트가 생성 후 서버 공개키로 암호화

5. 대칭키 생성

    - 양측이 동일한 세션 키 생성

6. Finished 메시지 교환

    - 이후부터 암호화 통신

## 📎 참고 자료

## <!-- 공부하면서 참고한 링크를 여기에 추가해주세요 -->

## 💬 토론 포인트

<!-- PR 리뷰 또는 스터디 중 나온 추가 질문이나 논의 사항을 기록해주세요 -->
