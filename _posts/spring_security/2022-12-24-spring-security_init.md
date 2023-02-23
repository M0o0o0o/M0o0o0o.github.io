---
title: "스프링 시큐리티란?"
categories: 
  - spring security
toc: true
toc_sticky: true
toc_label: "Contents"
---


```text
스프링 시큐리티가 무엇인지 간략하게 알아보고   
애플리케이션에 스프링 시큐리티의 기본 기능을 알아보자.
```
# 스프링 시큐리티란? 
> 스프링 시큐리티 공식문서에 따르면...

- 스프링 시큐리티는 커스터마이징이 자유롭게 가능한 인증 및 인가를 위한 프레임워크다.
- 사실상 스프링 기반 애플리케이션에서 <span style='background-color:#fff5b1'>표준 기술</span>로 사용된다.

```text
스프링 시큐리티는 자바 application의 인증과 인가 기능을 제공한다.
```

## Spring Security's features
1. 폭넓고 확장 가능한 인증 및 인가를 제공한다.
2. session fixation, clickjacking, csrf와 같은 공격을 위한 기본적인 보안 기능을 제공한다.
3. Servlet API integration
4. 스프링 Web MVC와 통합이 가능한다. 

```text
공식문서에 따르면 스프링 시큐리티는 인증과 인가 처리를 위한 기능을 제공해주며,   
사용자가 확장해서 커스텀하게 인증 및 인가 기능을 사용할 수 있다.   
```

---

# 스프링 시큐리티 의존성 추가
> 스프링 시큐리티 의존성만 추가해도 시큐리티의 기본 초기화 작업 및 보안 설정이 이루어잔다.

<span style="color:red;">즉, 별도의 설정이나 구현을 하지 않아도 웹 보안 기능이 작동한다.</span>   

**제공되는 기본 기능은 다음과 같다.**
1. 모든 요청은 인증이 되어야 자원에 접근이 가능
2. 인증 방식은 폼 요청과 httpBasic 로그인 방식을 제공
3. 기본 로그인 페이지 제공
4. 기본 계정 한 개 제공

## 필터 기반의 스프링 시큐리티
스프링은 필터 기반으로 작동한다.   
따라서 위의 기능들을 제공하는데 필요한 스프링 시큐리티의 필터 목록은 다음과 같다. 

<img width="393" alt="스크린샷 2022-12-24 오후 7 51 45" src="https://user-images.githubusercontent.com/121086012/209432641-c3558c16-60fa-4be5-8077-3f076221643b.png">


> 스프링 시큐리티의 의존성을 추가하는 것만으로도 위와 같은 기본 기능을 제공해주지만, 
> 실제 애플리케이셔은 권한 관리, 계정 관리 등 복잡하고 다양한 기능이 필요하기 때문에 스프링 스큐리티의 기능을 이용해
> 상황에 맞게 커스텀해야 한다. 

---

### references
[스프링 시큐리티 공식문서](https://docs.spring.io/spring-security/reference/index.html)
[스프링 시큐리티 - 인프런](https://www.inflearn.com/course/코어-스프링-시큐리티/dashboard)