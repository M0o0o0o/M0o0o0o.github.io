---
title:  "스프링 시큐리티의 아키텍처"
categories:
- spring security

toc: true
toc_sticky: true
toc_label: "Contents"
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

> 인증 흐름에서 사용되는 필터들과 흐름에 대해 알아보자.

## Authentication

Authentication의 정의는 [Oauth0](https://auth0.com/intro-to-iam/what-is-authentication)에 따르면 다음과 같다.
> Authtication is typically associated with proving a user's identity. Usually, a User proves their identity by providing
> their credentials, that is, and agreed piece of information shared between the user and the system.

- 즉, Authentication은 사용자의 정보를 통해 시스템에 인증하는 것을 말한다.
- 인증을 위한 정보는 대표적으로 (아이디 + 비밀번호), 스마트폰, 생체 정보를 이용할 수 있다.

---

Spring Security에서 Authentication는 사용자의 인증 정보를 저장하는 토큰 개념이다.
   
구조는 다음과 같다. 
1. principal
   - 사용자 아이디 혹은 User 객체를 저장
2. credentials
   - 사용자 비밀번호
3. authorities
   - 사용자의 권한 목록
4. details
   - 부가 정보
5. Authenticated
   - 인증 여부

![스크린샷 2022-12-25 오후 1 03 22](https://user-images.githubusercontent.com/121086012/209456625-918ee134-4696-4266-9bc7-3e209679237d.png)

---

## Authentication Flow(인증 흐름)

<img width="1318" alt="스크린샷 2022-12-25 오후 1 06 21" src="https://user-images.githubusercontent.com/121086012/209456664-8ec8c5ba-be8c-4eea-be14-8050bf4b3757.png">

인증 흐름에 사용되는 필터는 다음과 같다.
1. AuthenticationFilter
   - 인증 요청이 들어오면 request의 URL과 입력에 대한 검증을 담당한다.
   - 요청이 올바르게 들어오면 AuthenticationManager에게 인증을 위임한다. 
2. AuthenticationManager
   - AuthenticationManger 목록에서 현재 요청에 대한 요건과 일치하는 AuthenticationProvider를 찾아 인증처리를 위임한다.
   - 즉 실제 인증 역할을 하지 않고 인증의 전반적인 관리를 한다.
3. AuthenticationProvider
   - 실제 인증 역할을 수행하며 ID 검증, Password 검증과 같은 인증 과정을 거친다.
4. UserDetails
   - UserDetails는 사용자의 정보를 담는 인터페이스다. 
   - UserDetails가 필요한 이유는 유저를 구현하는 방식은 어플리케이션마다 모두 다르기 때문에 이를 추상화해서 spring security에서 통일된 방식으로 
   관리하기 위해서다. 
   - UserDetails의 구성은 다음과 같다.
   
   ![스크린샷 2022-12-25 오후 1 20 52](https://user-images.githubusercontent.com/121086012/209456863-f5f98549-4b8d-4557-95cd-432f82a8a68f.png)

---

## Authorization
- 인가는 인증 후 특정 자원에 대한 **권한**이 있는지 확인하는 것을 말한다. 
> 예를 들어 특정 유저가 관리자 게시판에 접근을 시도할 경우 해당 유저에게 관리자 권한이 있는지 확인하는 일련의 과정을 말한다. 

   
- 스프링 시큐리티는 다양한 방식의 권한 계층 방식을 지원한다.
  - 웹 계층(URL 요청에 따른 보안 )
  - 서비스 계층(AOP를 활용한 메서드 단위의 보안)

## AccessDeniedManager
- 인증 정보, 요청 정보, 권한 정보를 이용해 사용자의 접근 여부를 최종 결정한다.
- 여러 개의 Voter로 구성되며 Voter로부터 접근에 대한 응답을 통해 최종 결정

접근결정의 세 가지 유형
1. AffirmativeBased
   - 여러개의 Voter 클래스 중 하나롣 접근 허가로 결론을 내면 접근을 허가한다.
2. ConsensusBased
   - 다수결에 의해 인가 결정을 판단한다.
3. UnanimousBased
   - 모든 Voter의 승인이 있어야 인가 허가가 가능하다.

## AccessDecisonVoter
- 인가의 승인 여부를 판단하는 인터페이스.
- Voter는 다음의 정보들로 인가 결정을 내린다.
  - Authenticaton : 인증 정보
  - FilterInvocation : 요청 정보(antMatcher("/admin"))
  - ConfigAttributes : 권한 정보

---

## FilterSecurityInterceptor
- 필터 체인의 마지막에 위치한 필터로써 인증된 사용자의 승인 혹은 거부 여부를 최종적으로 결정한다.
- 인증객체 없이 보호자원에 접근할 경우 AuthenticationException 예외를 발생시킨다.
- 인증 후 자원 접근 권한이 존재하지 않는 경우 AccessDeniedException 예외를 발생시킨다.
  
<img width="1267" alt="스크린샷 2022-12-25 오후 1 37 37" src="https://user-images.githubusercontent.com/121086012/209457095-4f029557-f82f-4679-9fda-97a4d4b9676d.png">


### references
[스프링 시큐리티 공식문서](https://docs.spring.io/spring-security/reference/index.html)
[스프링 시큐리티 - 인프런](https://www.inflearn.com/course/코어-스프링-시큐리티/dashboard)