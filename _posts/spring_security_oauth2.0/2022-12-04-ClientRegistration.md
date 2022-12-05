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

<img src="/public/img/1204_clientregistration.png" width="900" object-fit="cover" alt="" />


## ClientRegistration
- OAuth2.0 또는 OpenID connect 1.0 Provider에서 클라이언트의 등록 정보
- ClientRegistration은 OpenId Connect Provider의 설정 엔드포인트나 인가 서버의 메타데이터 엔드포인트를 찾아 초기화할 수 있다.

```java
public final class ClientRegistration implements Serializable {

    private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

    private String registrationId;

    private String clientId;

    private String clientSecret;

    private ClientAuthenticationMethod clientAuthenticationMethod;

    private AuthorizationGrantType authorizationGrantType;

    private String redirectUri;

    private Set<String> scopes = Collections.emptySet();

    private ProviderDetails providerDetails = new ProviderDetails();

    private String clientName;

    public class ProviderDetails implements Serializable {

        private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

        private String authorizationUri;

        private String tokenUri;

        private UserInfoEndpoint userInfoEndpoint = new UserInfoEndpoint();

        private String jwkSetUri;

        private String issuerUri;

        private Map<String, Object> configurationMetadata = Collections.emptyMap();
    }
    
    public class UserInfoEndpoint implements Serializable {

        private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

        private String uri;

        private AuthenticationMethod authenticationMethod = AuthenticationMethod.HEADER;

        private String userNameAttributeName;
    }
}
```

> ClientRegistration의 코드를 보면 ProviderDetails 클래스를 정의하고 있고 ProviderDetails는 UserInfoEndpoint를 정의하고 있다.

<img src="/public/img/1204_bach_code.png" width="900" object-fit="cover" alt="" />


- registrationId : ClientRegistration을 식별할 수 있는 ID 값(유니크)
- clientId : 클라이언트 식별자
- clientSecret : 클라이언트 secret
- clientAuthenticationMethod : provider에서 클라이언트를 인증할 때 사용할 메서드로서 basic, post, public을 지원한다.
- authroizationGrantType : OAuth2.0 인가 프레임워크에서 지원하는 권한 부여 타입
- redirectUri : 클라이언트에 등록한 리다이렉트 URI로 사용자의 인증으로 클라이언트에 접근 권한을 부여하고 나면, 인가 서버가 해당 URL로 브라우저를 리다이렉트 시킨다.
- ccopes : 클라이언트가 요청한 openid, 이메일, 닉네임 등의 scope
- clientName : 클라이언트를 나타내는 이름으로 자동 생성되는 로그인 페이지 등에서 사용한다.
- authorizationUri : 인가 서버의 엔드포인트 URI
- tokenUri : 인가 서버의 토큰 엔드포인트 URI
- jwtsetUri : 인가 서버에서 JSON 키 셋을 가져올 때 사용할 URI.
- configurationMetatdata : OpenID Provider 설정 정보로서 application.properties에 spring.security.oauth2.client.provider.[providerId].issuerUri를 설정했을 때만 사용할 수 있다.
- (userInfoEndpoint)uri : 인증된 최종 사용자의 클레임/속성에 접근할 때 사용하는 UserInfo 엔드포인 트 URI.
- (userInfoEndpoint)authenticationMethod : UserInfo 엔드포인트로 액세스 토큰을 전송할 때 사용 할 인증 메소드. header, form, query 를 지원한다.
- userNameAttributeName : UserInfo 응답에 있는 속성 이름으로, 최종 사용자의 이름이나 식별자에 접근할 때 사용한다

### references
[스프링 시큐리티 OAuth2 - Spring Boot 기반으로 개발하는 Spring Security OAuth2](https://www.inflearn.com/course/정수원-스프링-시큐리티)