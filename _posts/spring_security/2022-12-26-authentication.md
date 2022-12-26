---
layout: post
title:  "Authentication(Spring Security)"
date:   2022-12-26 11:47:29 +0900
categories: spring-security
---

### SecurityContextHolder

스프링 시큐리티의 인증 모델 중심에는 SecurityContextHolder가 있다. 이 홀더는 SecurityContext를 가지고 있다.


SecurityContextHolder에는 스프링 시큐리티로 인증한 사용자의 상세 정보를 저장한다. 스프링 시큐리티는 SecurityContextHolder에 어떻게 값을 넣는지는 상관하지 않는다. 값이 있다면 현재 인증한 사용자 정보로 사용한다.

> 인증된 주체(principal) 정보를 얻어야 한다면 SecurityContextHolder에 접근하면 된다.
>

### SecurityContext

SecurityContext는 SecurityContextHolder로 접근할 수 있다. SecurityContext는 Authentication 객체를 가지고 있다.

### Authentication

스프링 시큐리티에서 Authentication이 주로 담당하는 일은 다음과 같다.

- AuthenticationManager의 입력으로 사용되어, 인증에 사용할 사용자의 credential을 제공한다. 이 상황에선 isAuthenticated()는 false를 리턴한다.
- 현재 인증된 사용자를 나타낸다.

Authentication는 다음과 같은 정보를 가지고 있다.

- principal
- credentials
- authorities - 사용자에게 부여한 권한은 GrantedAuthority로 추상화한다. 예시로 role이나 scope가 있다.

### GrantedAuthority

사용자에게 부여한 권한은 GrantedAuthority로 추상화한다.

GrantedAuthority는 Authentication.getAuthorities() 메소드로 접근할 수 있다. 이 메소드는 GrantedAuthority 객체의 Collection을 리턴한다. GrantedAuthority는 말 그대로 인증한 주체에게 부여된 권한이다.

이름/비밀번호 기반 인증을 사용한다면 보통 UserDetailServic가 GrantedAuthority를 로드한다.

GrantedAuthority 객체는 일반적으로 어플리케이션 전체에 걸친 권한을 의미한다. 특정 도메인 객체에 국한되지 않는다. 따라서

### Request Credentials With AuthenticationEntryPoint

AuthenticationEntryPoint는 클라이언트의 credential을 요청하느 HTTP 응답을 보낼 때 사용한다.

클라이언트가 리소스를 요청할 때 미리 이름/비밀번호 같은 credential을 함께 보낼 때도 있다. 이럴 때는 credential을 요청하는 HTTP 응답을 만들 필요가 없다.

하지만 어떨 땐 클라이언트가 접근 권한이 없는 리소스에 인증되지 않은 요청을 보내기도 한다. 이때는 AuthenticationEntryPoint 구현체가 클라이언트에 credential을 요청한다. AuthenticationEntryPoint는 로그인 페이지로 리다이렉트 하거나, WWW-Authenticate 헤더로 응답하는 일의 일을 담당한다.