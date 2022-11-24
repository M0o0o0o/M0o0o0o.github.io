---
layout: post
title:  "프로타입 스코프"
date:   2022-11-23 19:00:00 +0900
categories: Spring 
---

## 빈 스코프
- 빈을 생성할 때 별도의 스코프를 지정해주지 않으면 기본적으로 싱글톤으로 생성한다.
- 싱글톤 이외에도 스프링은 다양한 빈의 스코프 방식을 지원하는데 다음과 같다.

---

- singleton
  - 기본 스코프로 스프링 컨테이너의 전체 라이프사이클동안 하나의 인스턴스만 생성돼 관리해주는 가장 넓은 범위의 스코프다.
- prototype 
  - 프로토타입 빈의 경우 스프링 컨테이너는 빈의 생성과 의존관계 주입만 해주는 역할을 한다.
- request 
  - 하나의 웹 요청의 라이프사이클동안 유지되는 스코프다.
- session
  - 웹 세션이 생성되고 종료될 때 까지 유지되는 스코프다.
- application
  - 웹의 서블릿 컨테이너의 생명주기동안 유지되는 스코프다.
- websocket
  - 웹 소켓 세션의 생명주기와 동일한 스코프다.

> 이렇게 다양한 스코프들이 존재하는데, 여기서는 싱글톤과 프로토타입 빈의 비교를 통해 차이점을 알아보려 한다.


## Singleton Bean
- 싱글톤으로 등록된 빈은 스프링 컨테이너의 시작과 종료까지 단 하나의 인스턴스만 생성해서 관리되는 스코프다. 
- 그 과정을 코드로 알아보자.


```java
@Component
public class SingletonBean {
}
```

```java
@Service
@RequiredArgsConstructor
public class Service_1 {
    public final SingletonBean singletonBean;
}
```

```java
@Service
@RequiredArgsConstructor
public class Service_2 {
    public final SingletonBean singletonBean;
}
```


```java
@SpringBootTest
@Slf4j
public class SingletonBeanTest {
  @Autowired
  Service_1 service1;

  @Autowired
  Service_2 service2;


  @Test
  void checkSingletonBeanbyEquals() {
    log.info("service1.singletonBean={}", service1.singletonBean);
    log.info("service2.singletonBean={}", service2.singletonBean);
    assertThat(service1.singletonBean.equals(service2.singletonBean)).isTrue();
  }
}
```


**<span style="color:red;">실행결과</span>**

<img src="/public/img/1123singleton.png" width="" height="" alt="" />

> 위 결과를 보면 싱글톤 빈은 딱 하나의 인스턴스만을 생성해 관리해준다는 것을 확인할 수 있다.

## 프로토타입 빈 
- 프로토타입 스코프를 갖는 빈을 스프링 컨테이너에 조회하면 항상 새로운 인스턴스를 생성해 반환해준다.
- 즉, 스프링 컨테이너는 생성 후 의존관계 주입까지만 관여한다.
- 코드를 통해 확인해보자.

```java
@Scope("prototype")
@Component
public class PrototypeBean {
}
```
- **@Scope annotation을 사용해서 스코프를 지정할 수 있다.**

```java
@Service
@RequiredArgsConstructor
public class ServicePrototype_1 {
    public final PrototypeBean prototypeBean;
}
```

```java
@Service
@RequiredArgsConstructor
public class ServicePrototype_2 {
    public final PrototypeBean prototypeBean;
}
```

```java
@SpringBootTest
@Slf4j
public class PrototypeBeanTest {

    @Autowired
    ServicePrototype_1 service1;

    @Autowired
    ServicePrototype_2 service2;

    @Test
    void checkPrototypeBeanbyEquals() {
        log.info("ServicePrototype_1.prototypeBean={}", service1.prototypeBean);
        log.info("ServicePrototype_2.prototypeBean={}", service2.prototypeBean);
        assertThat(service1.prototypeBean.equals(service2.prototypeBean)).isFalse();
    }
}
```

**실행결과**

<img src="/public/img/1123proto.png" width="" height="" alt="" />

- 결과를 보면 service1가 참조하고 있는 ProtoTypeBean의 주소와 service2가 참조하고 있는 ProtoTypeBean의 주소값이 다른 것을 볼 수 있다.

## 프로토타입 사용 예시
- 프로토타입 공부하면서 실제 쓰이는 예시가 궁금해서 찾아봤다.
- Spring Security는 필터 기반의 인증, 인가 기능을 제공하는데, 개발자가 다양한 필터들을 커스텀하게 추가할 수 있는데 이때 여러 필터들을 빈으로 등록하기 위해서 프로토타입 빈을 사용한다.

<img src="/public/img/1123prototypeEx.png" width="" height="" alt="" />

## 프로토타입 빈과 싱글톤 빈
- 싱글톤 빈에서 프로토타입 빈을 사용할 때 주의할 점이 하나 있다. 
- 우선 프로토타입을 사용하는 이유는 사용할 때 마다 새로운 객체가 필요하기 때문이다.
  - 그렇지만 싱글톤 빈에서 프로토타입을 의존받으면 싱글톤 빈이 생성되는 시점에 스프링 컨테이너가 최초 한번 프로토타입 빈을 생성해서 주입해주기 때문에 이후의 해당 싱글톤 빈은 계속해서 같은 프로토타입 빈을 사용학 ㅔ된다.

> 즉, 해당 프로토타입 빈을 사용할 때 마다 새로운 인스턴스가 필요한데, 이미 스프링에서는 필요한 기능을 제공한다.

### ObjectProvider
> 코드를 통해 ...

```java
@Scope("prototype")
@Component
public class PrototypeBean {
}
```

```java
@Slf4j
@Service
public class BeanWithPrototype {

    @Autowired
    private ObjectProvider<PrototypeBean> prototypeBeanProvider;

    public void logic() {
        PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
        log.info("prototypeBean={}", prototypeBean);
    }
}
```

```java
@SpringBootTest
@Slf4j
public class PrototypeBeanTest {

    @Autowired
    ServicePrototype_1 service1;

    @Autowired
    ServicePrototype_2 service2;

    @Test
    void checkPrototypeBeanbyEquals() {
        log.info("ServicePrototype_1.prototypeBean={}", service1.prototypeBean);
        log.info("ServicePrototype_2.prototypeBean={}", service2.prototypeBean);
        assertThat(service1.prototypeBean.equals(service2.prototypeBean)).isFalse();
    }
}
```

