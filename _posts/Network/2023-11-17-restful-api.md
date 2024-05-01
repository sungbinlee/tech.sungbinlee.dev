---
title: "RESTful API란 무엇인가요?"
categories:
  - Network
tags:
  - RESTful API
  - REST API
  - Backend
  - Networking
toc: true
toc_sticky: true
toc_label: "RESTful API란?"
toc_icon: "internet"
---

## RESTful API란 무엇인가요?
> "Representational State Transfer(REST)는 API 작동 방식에 대한 조건을 부과하는 소프트웨어 아키텍처입니다." - 출처: aws

REST는 웹 아키텍처 스타일 중 하나로, 네트워크 기반의 소프트웨어 아키텍처를 지칭합니다. REST는 자원(Resource)을 URI(Uniform Resource Identifier)로 표현하고, HTTP 프로토콜을 통해 자원에 대한 행위를 정의하는 방법론입니다.

RESTful API는 REST의 원칙을 따라 설계된 API를 지칭합니다. 이를 위해서는 몇 가지 기본적인 원칙을 준수해야 합니다.

### REST의 개념과 RESTful API의 기본 원칙 설명


### HTTP Methods
#### GET vs POST
의미와 차이점:
- **GET**: 정보를 요청하기위한 메서드로, 서버로부터 데이터를 요청할 떄 사용됩니다. URL에 파라미터를 포함하여 데이터를 전송합니다.
- **POST**: 서버에 새로운 리소스를 생성하기 위해 사용되며, 데이터를 서버로 제출합니다. POST 요청은 body에 데이터를 담아 전송합니다.
  
특별한 구분점:
- POST는 보안 측면에서 GET보다 안전합니다. POST 요청은 body에 데이터를 포함하기 때문에, URL에 노출되지 않아야 하는 민감한 정보를 전송할 수 있습니다.

#### DELETE
의미와 전달 방법:
- **DELETE**: 리소스를 삭제하기 위해 사용됩니다. DELETE 요청은 특정 리소스를 식별하는 URL을 통해 전달됩니다. `/articles/{id}` 와 같은 형식으로 사용됩니다.

#### PUT vs PATCH
의미와 차이점:
- **PUT**: 리소스를 업데이트 할때 사용되며 전체 엔티티를 업데이트 합니다. 요청된 리소스가 존재하지 않으면 새로운 리소스를 생성할 수 있습니다.
- **PATCH**: 리소스를 부분적으로 업데이트할 때 사용됩니다. PUT과 달리 전체 엔티티를 보내는 것이 아니라, 일부만을 보내서 업데이트할 수 있습니다.

권한 검증:
- 업데이트 권한을 백엔드에서 검증하기 위해, 보통 미들웨어를 활용합니다. 이를 통해 권한이 있는지 확인하고, 요청이 수정 권한에 적합한지 검사합니다. 이렇게 작성된 미들웨어는 여러 엔드포인트에서 재사용될 수 있습니다.


## RESTful API 디자인 가이드

### 리소스 네이밍과 URI 설계
- **고유하고 직관적인 URI**: 자원을 나타내는 URI는 명확하고 직관적이어야 합니다. `/users`, `/articles`와 같이 복수형 명사를 사용하는 것이 일반적입니다.
- **명확한 계층 구조**: URI는 계층 구조로 표현되어야 하며, 관련성 있는 자원 간의 관계를 잘 표현해야 합니다.
  
### HTTP 메서드와 리소스 상태 코드 활용 방법
- **적절한 HTTP 메서드 사용:** GET은 조회, POST는 생성, PUT 또는 PATCH는 업데이트, DELETE는 삭제에 사용됩니다.
- **상태 코드 활용:** 적절한 상태 코드(200, 201, 404, 403 등)를 반환하여 요청의 성공 또는 실패를 클라이언트에게 알려줍니다.
