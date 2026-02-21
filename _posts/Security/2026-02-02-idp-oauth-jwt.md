---
title: "IDP · OAuth2 · JWT 정리"
date: 2026-02-02
categories:
  - Security
tags:
  - OAuth2
  - JWT
  - IDP
toc: true
toc_sticky: true
---

IDP, OAuth2, JWT는 현대 인증/인가 구조의 핵심 요소다. 로그인 구현이나 MSA 환경에서 자주 사용되지만, 개념이 혼동되거나 잘못 적용되는 경우가 많다.

이 글에서는 인증과 인가의 구분, OAuth2 흐름, JWT의 본질, 그리고 실무 보안 관점으로 정리해 보았다.
## 1. 인증(Authentication) vs 인가(Authorization)

* **인증**: 누군지 확인
* **인가**: 무엇을 할 수 있는지 판단

항상 순서:

> 인증 → 인가

인가만 단독으로 존재할 수 없다.

## 2. OAuth 주요 Grant

### Authorization Code Grant 

* 사용자(User)가 로그인
* 브라우저는 로그인 UI만 담당
* authorization code는 임시 표식
* code → token 교환은 반드시 백엔드에서 수행

목적:

* 토큰을 브라우저 JS로부터 분리
* 보안 강화

공개 클라이언트(SPA 등):

> 반드시 PKCE 적용

### Client Credentials Grant

* 사용자 없이 서버 간 인증
* `client_id` + `client_secret` 사용
* 백엔드 ↔ 백엔드 통신

## 3. OAuth 역할 구분

* **Authorization Server**: 토큰 발급  
  예: Keycloak

* **Client**: 토큰 요청 및 사용

* **Resource Server**: 토큰 검증 및 리소스 제공

## 4. Authorization Code Grant 흐름 / [Authorization Code Grant (RFC 6749 §4.1)](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1)

1. 브라우저 → 로그인 페이지 이동
2. 로그인 성공 → authorization code 수신
3. 백엔드 → code 전달
4. Authorization Server → 토큰 발급
5. 백엔드 → 토큰 저장

권장:

* HTTP-only Cookie
* 또는 서버 세션(opaque id)

## 5. 브라우저의 역할

브라우저는 최대한 단순하게 유지한다.

* JWT 저장하지 말 것
* localStorage 저장하지 말 것
* Authorization 헤더 직접 구성하지 말 것

쿠키 자동 전송만 사용한다.

## 6. JWT를 브라우저 JS에서 다루면 안 되는 이유

JavaScript 실행 환경은 신뢰할 수 없다.

위험 요소:

* XSS → 토큰 탈취
* localStorage → 항상 노출
* 악성 스크립트 삽입

결론:

> JWT 관리는 백엔드 책임

## 7. 권장 아키텍처 — BFF / Gateway

```

Browser
↓ (HTTP-only Cookie)
Backend / BFF
↓ (Authorization: Bearer <token>)
Resource Servers

```

JWT는 백엔드만 처리한다. 브라우저는 세션과 유사한 UX로 동작한다.

## 8. JWT 구조와 본질

구조:

```

header.payload.signature

```

header / payload:

* Base64URL 인코딩
* 암호화 아님
* 누구나 디코딩 가능

따라서:

* 민감 정보 저장 금지
* 개인정보 저장 금지

signature:

* 위조 방지 목적
* 암호화 아님

JWT는 숨기는 값이 아니다.

## 9. JWT 검증 — 실무 체크리스트

서명 검증만 수행하면 부족하다.

필수 검증:

* signature 유효성
* exp (만료)
* nbf (사용 가능 시점)
* iss (발급자)
* aud (대상)

인가 판단:

* scope
* roles / authorities

Payload 설계 원칙:

* 암호화되지 않음을 전제
* 민감 정보 저장 금지
* 개인정보 저장 금지
* 최소 정보만 포함

매 요청마다 검증 및 인가를 수행한다.

## 10. JWT 탈취의 본질

JWT는 bearer token이다.

의미:

* 토큰을 가진 사람이 사용자로 인정됨
* 사용자 비밀번호 불필요

따라서:

> JWT의 위험은 위조가 아니라 탈취

JWT는 현금과 유사하다. 유효하면 누구나 사용 가능하다.

## 11. JWT 탈취 대응 전략

대응 전략은 두 가지 관점으로 나눌 수 있다.

1. 훔치지 못하게 한다
2. 훔쳐도 쓸모없게 한다

## 12. 훔치지 못하게 하는 방법

### HTTPS 사용

* 스니핑 방지
* 중간자 공격 방지

토큰 기반 인증에서는 필수 조건이다.

### 토큰 저장 위치

위험:

* localStorage
* sessionStorage

이유:

* JavaScript 접근 가능
* XSS 시 탈취 가능

권장:

* HTTP-only Cookie

장점:

* JS 접근 불가
* XSS로 직접 탈취 어려움

### SameSite 설정

쿠키 기반 구조에서 필수.

* Strict
* Lax
* None (+ Secure 필수)

### 로그 보안

* Authorization 헤더 로그 금지
* 토큰 마스킹 처리
* 로그 접근 통제

로그를 통한 토큰 유출 사고 빈번.

## 13. 훔쳐도 쓸모없게 하는 방법

### Access Token 수명 단축

전략:

* Access Token → 짧게
* Refresh Token → 상대적으로 길게

탈취 피해 시간 최소화 목적.

### Refresh Token Rotation

동작:

1. 갱신 시 새 Refresh Token 발급
2. 이전 토큰 무효화
3. 이전 토큰 재사용 감지

효과:

* 탈취 탐지 가능

### Reuse Detection

이미 사용된 Refresh Token 재사용 시:

* 모든 토큰 폐기
* 세션 종료
* 재로그인 요구

### JTI 기반 블랙리스트

JWT의 jti 활용:

* 로그아웃 시 저장
* 재사용 시 차단

저장소 예:

* Redis

단점:

* 완전 Stateless 포기

### SID + JTI 세션 추적

전략:

* SID → 세션 식별자
* JTI → 토큰 식별자

구조:

```

Key: SID
Value: 현재 유효 JTI
TTL: Refresh Token 만료 시간

```

효과:

* 토큰 재사용 탐지
* 세션 강제 폐기 가능

### 환경 바인딩 (보조 수단)

예:

* User-Agent 해시
* 디바이스 정보

완벽한 방어는 아니며 이상 징후 탐지용.

### 중요 기능 추가 인증

대상:

* 비밀번호 변경
* 결제
* 송금
* 권한 변경

전략:

* 재인증
* MFA / 2FA

## 14. Stateless vs Stateful 현실적 선택

| 전략 | 특징 |
|------|------|
| 순수 JWT | 확장성 높음 / 통제 약함 |
| Refresh 서버 관리 | 통제 강함 / 상태성 증가 |

실무에서 목적에 따라 trade off가 필요하다

## 15. 실무 권장 조합

일반 서비스 기준:

* HTTPS
* HTTP-only Cookie
* SameSite 설정
* Access Token 수명 단축
* Refresh Rotation
* 중요 기능 재인증

보안 민감 서비스:

* Refresh Token 서버 저장
* Reuse Detection
* 세션 추적

## 16. 핵심 요약

* JWT는 bearer token이다
* 위조보다 탈취가 위험하다
* 저장 전략이 보안의 핵심이다
* 짧은 수명 + Rotation이 기본 대응이다
* 완전 Stateless에 집착하지 않는다

> 보안은 기술 문제가 아니라 설계 문제

**같이보면 좋은 글**
- [Keycloak + OAuth2 인증 흐름 살펴보기]({{ site.url }}{{ site.baseurl }}/security/keycloak-oauth/) 
