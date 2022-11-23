---
layout: post
title:  "@PostConstruct, @PreDestroy"
date:   2022-11-21 19:31:29 +0900
categories: Spring
---

## 빈 생명주기
- 스프링 컨테이너가 관리해주는 빈은 다음과 같은 라이프사이클을 갖는다.

1. 스프링 컨테이너 생성
2. 스프링 빈 생성
3. 의존관계 주입
4. 스프링 종료

> 위와 같은 라이프사이클에서 추가적으로 스프링은 의존관계 주입이 완료되면(빈을 사용할 수 있는 시점) 
> <span style="color:red;">빈의 생성 후, 종료 직전에 콜백 메서드를 제공한다.</span>

스프링은 크게 3가지 방법의 빈 생명주기 콜백을 지원한다.
- @PostConstruct, @PreDestroy 애노테이션 지원
- 인터페이스(InitializingBean, DisposableBean)
- 설정 정보에 초기화 메서드, 종료 메서드 지정 

> 이 중 많이 쓰이는 Annotation 방식의 @PostConstruct, @PreDestroy를 코드로 살펴보자.


```java
@SpringBootTest
@Slf4j
public class BeanCallBackTest {

    @Autowired
    BeanTest beanTest;

    @Test
    void test() {
        beanTest.logic();
    }

    @TestConfiguration
    static class Config{
        @Bean
        BeanTest beanTest() {
            return new BeanTest();
        }
    }


    static class BeanTest {
        @PostConstruct
        public void postConstructMethod() {
            log.info("post Construct 호출");
        }


        public void logic() {
            log.info("logic");
        }

        @PreDestroy
        public void preDestroyMethod() {
            log.info("preDestroy 호출");
        }
    }
}
```

**실행 결과는 다음과 같다.**

```text
post Construct 호출
logic
preDestroy 호출
```

- 동작 순서는 다음과 같다.
1. 스프링 컨테이너 생성
2. BeanTest을 빈으로 등록
3. 현재 BeanCallBackTest에 BeanTest를 주입한다.
4. @PostConstruct가 붙은 postConstructMethod() 실행
5. logic() 실행 
6. @PreDestroy가 붙은 preDestroy() 실행 
7. 종료

## @PostConstruct, @PreDestroy 애노테이션 특징
- 스프링 공식문서에 따르면 애노테이션 방식의 콜백 사용을 권장한다.
- 단점은 외부 라이브버리에는 적용하지 못하기 때문에 해당 상황에는 다른 콜백 방식을 사용해야 한다.