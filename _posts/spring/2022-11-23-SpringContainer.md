---
title:  "스프링 컨테이너(ApplicationContext)"
categories: 스프링
toc: true
toc_sticky: true
toc_label: "Contents"
---

<span style="color:red;">ApplicationContext</span>를 스프링 컨테이너라 하는데, 이번에는 ApplicationContext의 구현체들과 ApplicationContext의
부모 interface인 BeanFactory에 대해 알아보려 한다.

## BeanFactory
- BeanFactory는 ApplicationContext의 상위 인터페이스다.
- 스프링 컨테이너의 최상위 인터페이스로서 스프링 빈을 관리하고 조회하는 역할을 담당한다.
- BeanFactory는 ApplicationContext의 부모 인터페이스로 기본적인 IoC container 기능을 제공한다.

```java
public interface BeanFactory {
    Object getBean(String name) throws BeansException;
    Object getBean(String name) throws BeansException;
    // ...
}
```

- 위의 코드는 BeanFactory의 일부를 가져온건데, 우리가 지금까지 ApplicationContext를 이용해서 getBean을 통해 조회했던 메서드는 BeanFactory가  제공하는 기능이다.

## ApplicationContext
- ApplicationContext는 BeanFactory를 상속받은 인터페이스로 기본적인 IoC container 기능 + 추가적인 기능들을 제공하는데,
해당 하는 기능들은 다음과 같다.   
   
1. 메시지소스 기능
   - Locale에 따라 다양한 언어 지원을 편리하게 해주는 기능
2. 환경변수
   - 로컬, 개발, 운영 환경을 구분해서 처리할 수 있다.
3. 리소스 조회
   - 파일, 클래스패스 등의 리소스를 편리하게 조회할 수 있게 해준다.
4. 애플리케이션 이벤트 
   - 이벤트 발행/구독 모델을 편리하게 지원한다.

위의 기능들은 ApplicationContext의 코드를 보면 알 수 있다.

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver {

	@Nullable
	String getId();


	String getApplicationName();


	String getDisplayName();


	long getStartupDate();

	@Nullable
	ApplicationContext getParent();

	AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;

}

```

> 정리하자면 ApplicationContext는 BeanFactory를 상속받아 스프링 컨테이너의 기능과 더불더 메시지소스와 같은 애플리케이션에 필요한 
> 다양한 기능들을 추가적으로 제공해준다.

## ApplicationContext의 다양한 구현체
- 보통 빈을 등록할 때 애노테이션을 활용하는데, xml 등의 방법으로도 빈을 등록할 수 있는 방법이 있다.
- 그래서 스프링은 다양한 방식으로 ApplicationContxet를 구성할 수 있는 다양한 구현체를 만들어 두었다.
  - AnnotationConfigApplicationContext.class
  - GenericXmlApplicationContext.class
  - etc...

> 스프링은 다양한 설정 형식들을 적용할 수 있도록 구현해놨는데, 그건 <span style="color:red;">BeanDefinition</span>이라는 한단계 더 
> 추상화를 했기 때문에 가능하다.

즉, ApplicationContext의 각 구현체들은 Annotation 또는 XML로 읽어들인 설정들을 BeanDefinition으로 만들어서 반환하고 
Spring Container는 이 BeanDefinition으로 컨테이너를 구성한다.