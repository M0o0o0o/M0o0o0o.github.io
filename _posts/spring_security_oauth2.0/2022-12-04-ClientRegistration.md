---
layout: post
title:  "Spring Security OAuth2.0 ClientRegistration란?"
date:   2022-12-04 13:00:00 +0900
categories: Spring-Security-OAuth2.0
---

## OAuth2.0 Client란
- OAuth2.0 인가 프레임워크의 역할 중 인가서버 및 리소스 서버와의 통신을 담당하는 클라이언트의 필터 기반으로 구현한 모듈
- 간단한 설정만으로 OAuth2.0 인증 및 리소스 접근 권한, 인가서버 엔드 포인트 구성 등의 구현이 가능하다.

### OAuth2.0 Login
- 애플리케이션의 사용자를 외부 OAuth2.0 Provider나 OpenID Connect 1.0 Provider 계정으로 로그인할 수 있는 기능을 제공.
- Authorization Code 방식을 사용한다. 

### OAuth2.0 Client
- OAuth2.0 인가 프레임워크에 정의된 클라이언트 역할을 지원.

---

## 클라이언트 권한 부여 요청 흐름
1. 클라이언트가 인가서버로 권한 부여 요청을 하거나 토큰 요청을 할 경우 클라이언트 정보 및 엔드포인트 정보를 참조해서 전달한다.
2. application.yml(=properties)에 클라이언트 설정과 엔드포인트 설정.
3. 초기화가 진행되면 application.yml에 있는 클라이언트 및 엔드포인트 정보가 **<span style="color:red;">OAuth2ClientProperties</span>**의 각 속성에 바인딩 된다.
4. OAuth2ClientProperties에 바인딩 되어 있는 속성의 값은 **<span style="color:red;">ClientRegistration</span>**클래스픠 필드에 저장된다.
5. OAuth2Client는 ClientRegistration를 참조해서 권한부여 요청을 위한 매개변수를 구성하고 인가서버와 통신한다.

<img src="/public/img/1204_clientregistration.png" width="500" object-fit="cover" alt="" />

### references
[스프링 시큐리티 OAuth2 - Spring Boot 기반으로 개발하는 Spring Security OAuth2](https://www.inflearn.com/course/정수원-스프링-시큐리티)