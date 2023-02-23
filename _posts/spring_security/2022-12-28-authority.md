---
title:  "Authority(Spring Security)"
categories:
- spring security
  
toc: true
toc_sticky: true
toc_label: "Contents"
---

## Authorities

Authentication 섹션에선 어덯게 모든 Authentication 구현체가 GrantedAuthority 객체 리스트를 저장하는지를 설명한다. GrantedAuthority 객체는 principal에게 부여한 권한을 나타낸다. AuthenticationManager가 GrantedAuthority 객체를 Authentication 객체에 삽입하며, 이후 권한을 결정할 때 AccessDecisionManager가 GrantedAuthority를 읽어간다.

스프링 시큐리티는 한 가지 GrantedAuthority 구현체 , SimpleGrantedAuthority를 제공한다. 이 클래스는 사용자가 지정한 String을 GrantedAuthority로 변환해 준다. 시큐리티 아키텍처에 속한 모든 AuthenticationProvider는 Authentication 객체에 값을 채울 때 SimpleGrantedAuthority를 사용한다.

## Pre-invocation Handling

스프링 시큐리티는 method invocaion이나 웹 요청같은 보안 객체에 대한 접근을 제어하는 인터셉터를 제공한다ㅏ. 호출을 허용할지 말지를 결정하는 pre-invocation 결정은 AccessDecisionManager에서 내린다.

AccessDecisionManager는 AbstractSEcurityinterceptor에서 호출하며, 최종적인 접근 제어를 결정한다.

### Authorize HttpServletRequest With FilterSecuritiyInterceptor

1. 먼저 FilterSecurityInterceptor가 SecurityContextHolder에서 Authentication을 조회한다.
2. FilterSecurityInterceptor가 넘겨받은 HttpSevletRequest, Response, FilterChain으로 FilterInvocation을 만든다.
3. 그 다음  FilterInvocation을 SecurityMetadataSource로 넘겨 ConfigAttribute 컬렉션을 가져온다.
4. 마지막으로 Authenticatio, FilterInvocation, ConfigAttributeㅋ을 AccessDecisionManager로 넘긴다.
5. 인가를 거절하면 AccessDeniedException을 던진다.
6. 인가를 승인하면 FilterSecurityInterceptor는 일반적인 어플리케이션 프로세스를 실행할 수 있도록 FilterChain을 이어간다.

```java
protected void configure(HttpSecurity http) throws Exception {
    http
        // ...
        .authorizeRequests(authorize -> authorize // (1)
            .mvcMatchers("/resources/**", "/signup", "/about").permitAll() // (2)
            .mvcMatchers("/admin/**").hasRole("ADMIN") // (3)
            .mvcMatchers("/db/**").access("hasRole('ADMIN') and hasRole('DBA')") // (4)
            .anyRequest().denyAll() // (5)         
        );
}
```