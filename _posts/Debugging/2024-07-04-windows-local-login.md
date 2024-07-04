---
title: "Windows 11 회사/학교 계정에서 로컬 계정으로 로그인하는 방법"
categories:
  - Debugging
tags:
  - Windows
toc: true
toc_sticky: true
---
본 글은 도메인 컨트롤된 컴퓨터에서 로컬 계정으로 접속하는 과정과 Windows 11이 기본적으로 온라인 계정을 사용하도록 설정되어 있어 발생할 수 있는 문제를 다룬다.

## **로컬 계정으로 로그인하기**
   - 시작 화면에서 `.\<로컬 계정 이름>`을 입력한다. 예를 들어, 로컬 계정 이름이 `User1`인 경우 `.\<User1>`
   - 이 과정을 통해 로컬 사용자로 로그인할 수 있다.

## 이유  
`.\`를 사용하는 이유는 로컬 계정과 도메인 계정을 구분하기 위해서다. Windows는 도메인 환경에서 도메인 계정과 로컬 계정을 구분할 필요가 있다.

- 도메인 계정
  - 만약 컴퓨터가 도메인에 속해 있다면, 기본적으로 입력한 사용자 이름은 도메인 계정으로 인식된다.
  - `username@domain`
- 로컬 계정
  - 로컬 계정을 사용하여 로그인하려면, Windows가 이 계정이 로컬 계정임을 알 수 있도록 .\를 사용한다.
  - `.\username`

## 참고 자료
- [Microsoft Learn - How to login to Windows 11 with a local account](https://learn.microsoft.com/en-us/answers/questions/663885/how-to-login-to-windows-11-with-a-local-account)
