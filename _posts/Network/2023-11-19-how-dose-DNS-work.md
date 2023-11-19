---
title: "DNS는 어떻게 동작하는가?"
categories:
  - Network
tags:
  - Internet
  - Networking
  - DNS
  - DNS records
  - DNS resoultion
toc: true
toc_sticky: true
toc_label: "DNS는 어떻게 동작하는가?"
toc_icon: "internet"
---

## DNS란?
DNS(Domain Name System)는 인터넷에서 도메인 이름을 해당하는 IP 주소로 변환해주는 시스템입니다.

네트워크 상에서 통신하기 위해 다른 장치들을 식별하기 위해 IP 주소가 사용됩니다. 그러나 우리는 도메인 이름을 통해 웹사이트에 접속하고자 합니다. 이때 DNS는 사용자가 입력한 도메인 이름을 해당하는 IP 주소로 해석하여 컴퓨터가 서버에 접근할 수 있도록 도와줍니다.

## DNS는 어떻게 동작하는가?
<img alt="image" src="https://github.com/sungbinlee/sungbinlee.github.io/assets/52542229/96a6df53-8da9-4911-84dd-e588d5e669ed">

> 그림 출처: https://www.youtube.com/watch?v=Wj0od2ag5sk 

DNS Resoultion 과정
1. 사용자 입력: 사용자가 브라우저에 도메인을 입력합니다.
2. 브라우저 동작:
- 브라우저는 해당 도메인의 IP 주소를 찾기 시작합니다.
- 먼저 브라우저 캐시를 확인하여 이전 방문에서의 웹사이트 IP 주소를 찾습니다.
- 캐시에서 찾지 못하면 DNS 캐시를 검사합니다.
- 여전히 찾지 못하면 Hosts 파일을 확인합니다.
3. 캐시 및 파일 검색 실패:
- 이 단계에서도 찾지 못하면, ISP(인터넷 서비스 제공업체)의 Recursive DNS 서버를 찾습니다.
- 이 서버에서 다른 사용자가 동일한 웹사이트를 방문했을 수 있기 때문에 정보를 찾아 반환합니다.

4. 최종 검색 실패:
- 앞선 과정에서도 IP 주소를 찾지 못하면, Root DNS 서버에 쿼리를 보냅니다.
- TLD(Top-Level Domain) 정보를 반환합니다.
- TLD의 NS(네임 서버) DNS 정보를 받아서 권한 있는 네임 서버(Authoritative Name Server)에 쿼리를 보냅니다.
- 여기서 도메인의 IP 주소를 반환합니다.
  
## DNS 레코드란?

**A Record:**
- 정의: A 레코드는 도메인 이름과 해당하는 IPv4 주소를 매핑하는 데 사용됩니다.
- 용도: 도메인의 웹사이트를 호스팅하는 서버의 IP 주소를 나타냅니다.
  
**CNAME Record:**
- 정의: CNAME 레코드는 도메인 이름을 다른 도메인 이름으로 매핑하는 데 사용됩니다.
- 용도: 하나의 도메인이 다른 도메인으로 별칭을 지정할 때 사용되며, 주로 리디렉션을 구현하는 데 활용됩니다.
  
**MX Record:**
- 정의: MX 레코드는 도메인의 메일 서버 정보를 지정하는 데 사용됩니다.
- 용도: 도메인에서 이메일을 처리하는 메일 서버의 우선 순위와 호스트 이름을 지정합니다. 여러 메일 서버가 있을 때 우선순위를 부여할 수 있습니다.
  
**TXT Record:**
- 정의: TXT 레코드는 텍스트 정보를 포함하는 레코드로, 주로 특정 도메인과 관련된 부가적인 정보를 제공하는 데 사용됩니다.
- 용도: SPF(Sender Policy Framework)나 DKIM(DomainKeys Identified Mail)과 같은 이메일 보안 관련 정보, 도메인 인증 등을 저장합니다.
  
**NS Record:**
- 정의: NS 레코드는 도메인의 네임 서버 정보를 나타냅니다.
- 용도: 도메인을 관리하는 네임 서버의 정보를 저장하며, 해당 도메인의 DNS 쿼리를 처리할 수 있는 네임 서버를 지정합니다.

## DNS 레코드 검사하기
- host: 주어진 도메인에 대한 DNS 정보를 확인하는 명령어입니다.
- dig: DNS에 대한 상세한 정보를 조회하는 명령어로, 도메인의 레코드 유형 및 기타 정보를 제공합니다.
- nslookup: 주어진 도메인의 IP 주소나 DNS 정보를 조회하는 명령어입니다.

> 예시 1. $ host -t txt naver.com
<img alt="image" src="https://github.com/sungbinlee/sungbinlee.github.io/assets/52542229/e6a6a915-60d5-4807-b510-d0f662bfc16f">

해당 도메인과 관련된 모든 TXT 레코드 반환. 첫번째는 구글 인증 관련, 두번째는 아마 이메일 인증관련, 마지막은 페이스북 인증관련

> 예시 2. $ host -t a naver.com
<img width="582" alt="image" src="https://github.com/sungbinlee/sungbinlee.github.io/assets/52542229/40a1aa57-f052-4883-a1ac-3e7d6c32992d">

해당 도메인과 관련된 모든 A 레코드 반환. 네이버의 IP 주소들
