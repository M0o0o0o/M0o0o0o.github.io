---
layout: post
title:  "스프링 시큐리티의 아키텍처"
date:   2022-12-25 11:47:29 +0900
categories: spring-security 
---

> 스프링 시큐리티의 전체적인 아키텍처에서 각 필터의 역할과 인증 및 인가 흐름에 대해 알아보자.

**시큐리티의 아키텍처는 다음과 같다**

<img width="1340" alt="스크린샷 2022-12-25 오전 11 49 33" src="https://user-images.githubusercontent.com/121086012/209455586-58802da2-50e0-41cc-bd8c-1ee70d248959.png">

[참조](https://www.inflearn.com/course/코어-스프링-시큐리티)

## DelegatingFilterProxy와 FilterChainProxy의 관계

스프링 시큐리티는 필터 기반으로 인증 및 인가 기능을 제공한다.   
서블릿 필터는 서블릿 컨테이너에서 실행되기 때문에 스프링 컨테이너에서 생성 및 관리하는 빈을 사용할 수 없다.
그래서 스프링 시큐리티는 애플리케이션이 실행되면 <span style="color:red;">'springSecurityFilterChain'</span>의 이름으로 생성되는 필터 빈을 찾아 위임하는 역할을 한다.
   
> 위의 내용을 코드를 통해 살펴보자.

![스크린샷 2022-12-25 오후 12 00 58](https://user-images.githubusercontent.com/121086012/209455714-2331d151-af97-407b-a102-71475288c3d4.png)   

> 실제 처리를 위임하기 위해 필요한 targetBeanName과 ApplicationContext를 설정하는 것을 확인할 수 있다.

![스크린샷 2022-12-25 오후 12 01 24](https://user-images.githubusercontent.com/121086012/209455724-fd698d97-6dc2-4806-8726-758e53dda7f7.png)   


---

## SecurityContext & SecurityContextHolder 

### SecurityContext
- Authentication 객체가 저장되는 저장소다.
- ThreadLocal에 저장되어 아무 곳에서나 참조가 가능
- 인증이 완료되면 HttpSession에 저장되어 어플리케이션에 전역 참조가 가능하다.

```java
public interface SecurityContext extends Serializable {
    
	Authentication getAuthentication();

	void setAuthentication(Authentication authentication);
}
```
- securityContext의 구현체는 SecurityContextImpl


### SecurityContextHolder
- SecurityContextHolder는 SecurityContext의 저장 방식과 SecurityContext의 정보를 초기화하는 편의 메서드를 구현하고 있다. 
- SecurityContextHolder의 주석에 따르면 다음과 같이 설명하고 있다. 

> The purpose of the class is to provide a convenient way to specify the strategy that should be used for a given JVM.   

---

## SecurityContextPersistenceFilter
- SecurityContext 객체의 생성, 저장, 조회 기능을 담당한다.

### 인증 시
- 새로운 SecurityContext 객체를 생성하여 SecurityContextHolder에 저장
- 인증이 최종 완료되면 Session에 SecurityContext를 저장 

### 인증 후
- Session에서 SecurityContext를 꺼내 SecurityContextHolder에 저장
- SecurityContext 안에 Authentication 객체가 존재하면 인증을 계속 유지


---

> 위에서는 
