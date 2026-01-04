---
title: "Keycloak + OAuth2 인증 흐름 살펴보기"
categories:
  - Security
tags:
  - Keycloak
  - OAuth2
  - IDP
  - network
toc: true
toc_sticky: true
---
회사에서 IDP(Identity Provider) 관련 작업을 하면서 Keycloak을 본격적으로 사용하게 됐다. 처음에는 그냥 **인증 서버** 정도로만 이해하고 있었는데, 실제로 여러 서비스가 붙고 환경이 복잡해지다 보니 개념을 제대로 이해하지 않으면 구조가 머릿속에서 정리가 안 됐다. 이 글은 그런 상황에서 Keycloak과 OAuth2 구조를 이해하려고 정리한 내용이다.

## 왜 IDP가 필요했나

서비스가 하나일 때는 인증을 서비스 내부에서 처리해도 큰 문제가 없다. 하지만 서비스가 늘어나고, 서로 다른 시스템들이 인증을 공유해야 하는 상황이 되면 이야기가 달라진다. 사용자 정보, 권한, 토큰 관리까지 각 서비스가 따로 가져가면 유지보수가 거의 불가능해진다.

그래서 인증을 전담하는 **IDP**가 필요했고, 우리 쪽에서는 그 역할을 **Keycloak**이 맡고 있다. Keycloak은 사용자 인증, 토큰 발급, 권한 관리까지 전부 중앙에서 처리하는 역할을 한다.

## Keycloak에서 Realm이란 개념

Keycloak을 처음 보면 가장 먼저 나오는 개념이 **Realm**이다. 처음엔 이게 뭔지 감이 잘 안 왔는데, 지금은 이렇게 이해하고 있다.

- Realm = 인증과 권한의 독립된 공간  
- 하나의 Realm 안에는 사용자(User), 애플리케이션(Client), 역할(Role)이 묶여 있다  
- Realm 간에는 기본적으로 서로 영향을 주지 않는다  

보통은 하나의 **Master Realm**이 있고, 이 Realm에서는 다른 Realm들을 관리만 한다. 실제 서비스 사용자나 애플리케이션은 각각의 서비스용 Realm 안에서 관리된다. 이 구조 덕분에 서비스 간 인증 경계를 깔끔하게 나눌 수 있다.

![keycloak-realm](/assets/images/security/keyclaokrealm.png)

> *Realm 구조 예시*

## OAuth2 흐름을 정리한 그림

OAuth2는 등장하는 역할이 여러 개라서, 결국 누가 뭘 하는지부터 정리가 필요했다.

![oauth2-sequence](/assets/images/security/keycloak-sequence.png)

### 1) Resource Requester

리소스를 요청하는 쪽이다. 상황에 따라 여러 가지가 될 수 있다. UI 앱(웹/모바일)일 수도 있고, Postman/Swagger UI 같은 테스트 클라이언트일 수도 있고, 서버 간 통신을 하는 다른 백엔드 서비스일 수도 있다. 즉 **“토큰을 가지고 리소스를 요청하는 주체”**라고 보면 된다.

### 2) Authorization Server (IDP)

인증과 토큰 발급을 담당하는 서버다. Keycloak 같은 IDP가 여기 들어간다. 사용자를 인증하고, 정상적인 요청이라면 **Access Token**을 발급한다.

### 3) Resource Server

실제 비즈니스 로직과 데이터를 가지고 있는 서버다. Django, Spring Boot 같은 서버들이 여기에 해당한다. Resource Server는 사용자를 직접 인증하지 않는다. 대신 요청에 포함된 토큰을 검증해서, 이 요청이 유효한지 판단한다.

## [Authorization Code Grant (RFC 6749 §4.1)](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1)

Authorization Code Grant는 브라우저 리다이렉트 기반으로 동작하고, 토큰을 프론트에 직접 노출하지 않고 서버에서 교환하는 흐름을 만들 수 있다.

- 사용자가 브라우저로 로그인 요청을 시작한다  
- Authorization Server(Keycloak)에서 사용자 인증이 진행된다  
- 인증이 끝나면 Authorization Code를 redirect로 돌려준다  
- Client(서버)가 그 code를 가지고 token endpoint로 요청해서 Access Token(필요하면 Refresh Token도)을 받는다  
- 이후부터는 Access Token을 붙여서 Resource Server(API)를 호출한다  

## 토큰 검증 방식에 대한 이해

토큰 검증 방식도 상황에 따라 나뉜다.

- 매 요청마다 Authorization Server에 토큰 검증 요청을 보내는 방식(Introspection)  
- 공개키(JWK)를 받아서 Resource Server 내부에서 토큰을 검증하는 방식(JWT 서명 검증)  

두 번째 방식이 네트워크 부담도 적고, 실제 운영 환경에서는 더 많이 쓰이지 않나 싶다.

![token-validation](/assets/images/security/keycloak-validate-token.png)

## 정리

Keycloak이나 OAuth2 자체가 어려운 개념이라기보다는, **등장하는 역할이 많아서 처음에 헷갈렸던 것 같다**. Resource Requester, Authorization Server, Resource Server 이 세 가지만 명확히 나누고 나니까 구조가 한 번에 정리됐다.
