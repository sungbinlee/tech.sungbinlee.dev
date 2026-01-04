---
title: "NginX + Keycloak으로 레거시 MSA에 보안 레이어 얹기"
categories:
  - Security
tags:
  - Keycloak
  - OAuth2
  - JWT
  - OpenResty
  - Nginx
  - Lua
  - MSA
  - network
toc: true
toc_sticky: true
---
회사에서 운영 중인 주요 솔루션은 오랫동안 폐쇄망 환경에서만 돌아가고 있었다. 국내 고객사 대부분이 외부와 완전히 차단된 네트워크를 쓰다 보니, API 보안에 대해서는 크게 고민할 일이 없었다. 서비스가 내부에서만 호출되니 인증/인가 로직 없이도 그냥 잘 굴러갔던 거다.

문제는 해외 고객사에서 API 보안에 대한 요구사항이 생기면서부터였다. 이제는 오픈된 인터넷 환경에서 서비스가 돌아가야 하고, 누구든 API를 두드릴 수 있는 구조가 된 것이다. 자연스럽게 “인증된 사용자만 API를 호출할 수 있어야 한다”는 요구가 붙었다.

## 문제

문제는 이 솔루션이 MSA(Microservice Architecture) 구조라는 점이었다. 서비스가 여러 개로 쪼개져 있는데, 각 서비스마다 보안 로직을 붙이면 관리가 복잡해지고 유지보수도 어려워진다.

그리고 무엇보다 이미 오랫동안 안정적으로 운영해온 서비스라는 점이 제일 걸렸다. 지금까지는 폐쇄망에서 잘 굴러가던 걸, 각 마이크로서비스에 인증/인가 로직을 붙이기 시작하면 수정 범위가 너무 커지고 영향도 예측하기 어렵다. 서비스가 여러 개라 한 군데만 바꿔도 배포/검증 포인트가 확 늘고, 어디서 사이드 이펙트가 터질지도 모른다. 결국 레거시는 최대한 건드리지 않고, 앞단에서 보안을 한 번에 처리할 수 있는 방식이 필요했다.

## 접근 방식

결론적으로 선택한 방법은 **Proxy 레이어에서 JWT 기반 인증을 처리하는 구조**였다. 기존 레거시 MSA 솔루션은 최대한 건드리지 않고, 앞단에 **OpenResty(Nginx + Lua)** 기반 Proxy(Key Validator)를 세워서 보안 레이어 역할을 하게 했다. JWT 발급은 Keycloak이 맡고, Proxy는 Keycloak의 공개키(JWKS)를 이용해 토큰을 검증한다.

## 전체 아키텍처

![proxy-keycloak](/assets/images/security/key-validator.png)

> *Proxy(OpenResty)에서 JWT를 검증하고 통과한 요청만 내부 MSA로 전달하는 구조*

흐름은 단순하게 가져갔다.

1. 클라이언트(웹/REST API Caller)가 Keycloak에 인증 요청을 보내고 JWT를 발급받는다  
2. 클라이언트는 Private URI 호출 시 JWT를 포함해서 Proxy(OpenResty)로 요청한다  
3. Proxy는 Keycloak의 공개키(JWKS)로 토큰을 검증한다  
4. 검증에 성공하면 내부 마이크로서비스(Private Services)로 요청을 전달한다  
5. Public URI는 JWT 없이도 Proxy를 거쳐 내부 서비스로 들어간다  

핵심은 이거다. **서비스 코드에는 보안 로직을 넣지 않는다.** 보안은 Proxy에서 끝내고, 내부 서비스는 원래 하던 비즈니스 로직만 하게 만들었다.

## Public / Private URI 분리

POC 단계에서는 Proxy에서 URI를 두 가지로 나눠서 갔다. 이유는 단순하다. 기존 레거시 MSA 안에서도 일부 서비스는 이미 보안이 적용돼 있었고, 어떤 건 아예 내부 호출만 전제로 만들어져 있어서 한 번에 전부 “무조건 JWT”로 묶기엔 부담이 컸다. 그래서 우선 경로 기준으로 정책을 분리해서, 적용 범위를 통제할 수 있게 만들었다.

- **Public URI**: 인증 없이 접근 가능한 엔드포인트  
- **Private URI**: 반드시 JWT가 있어야 접근 가능한 엔드포인트  

예시는 이런 느낌이다.

- `/publicURI/**` → JWT 검사 없이 내부 Public 서비스로 전달  
- `/privateURI/**` → JWT가 없거나 유효하지 않으면 Proxy에서 바로 차단  

## OpenResty(Lua)로 JWT 검증

JWT 검증은 OpenResty에서 Lua 스크립트로 처리했다. 요청 헤더에서 토큰을 꺼내고, 서명 검증에 실패하거나 토큰이 유효하지 않으면 Proxy 단계에서 바로 401/403으로 끊는 방식이다. 레거시 서비스들에 인증 로직을 직접 넣지 않고도 앞단에서 공통으로 막을 수 있다는 점이 컸다. 아래 코드는 구현 예시이다.

> Nginx에 이런 JWT검증 로직을 넣는 방법이 몇 가지 있었는데, 하나는 Nginx Plus(유료)를 쓰는 방식이고, 다른 하나는 OpenResty를 쓰는 방식이었다. OpenResty는 LuaJIT이 내장된 Nginx 기반 플랫폼이라, 프록시 레벨에서 Lua로 인증/검증 로직을 비교적 유연하게 붙일 수 있었다.

```lua
access_by_lua_block {
    local jwt = require("resty.jwt")
    local auth_header = ngx.var.http_Authorization

    if not auth_header then
        return ngx.exit(401)
    end

    local token = string.gsub(auth_header, "Bearer ", "")
    local jwt_obj = jwt:verify("my-secret-key", token)

    if not jwt_obj.verified then
        return ngx.exit(403)
    end
}
```

실제로는 secret을 하드코딩하지 않고, Keycloak이 제공하는 **JWKS(공개키 세트)**를 받아서 서명 검증을 하도록 구성했다. 중요한 건 “검증에 실패하면 내부 서비스까지 요청이 도달하지 않는다”는 점이다.

## Keycloak 인증 서버

Proxy만 있다고 끝나는 건 아니고, JWT를 발급해줄 인증 서버가 필요하다. 여기서는 Keycloak을 테스트 환경에 구축했다.

* Keycloak에서 사용자 등록 → 로그인 → 토큰 발급
* Keycloak은 JWKS 공개키 제공
* Proxy는 JWKS를 가져다가 JWT 서명을 검증

## 결과

이 구조를 통해서 기존 레거시 MSA 시스템에 보안 레이어를 얹을 수 있었다.

* 서비스 코드에는 보안 로직을 넣을 필요가 없다
* Proxy에서 JWT 검증을 통과한 요청만 내부 서비스로 들어온다
* 보안 정책을 중앙에서 관리할 수 있어서 운영 효율이 올라간다

폐쇄망에만 머물렀다면 이런 고민이 필요 없었겠지만, 오픈된 환경에서 서비스를 운영하려면 API 보안은 결국 필수 과제가 된다. OpenResty 기반 Proxy Key Validator + Keycloak 조합으로 기존 구조를 크게 바꾸지 않고도 요구사항을 맞출 수 있었다.

