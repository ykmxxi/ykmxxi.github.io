---
title: "[Spring] IoC(Inversion of Control), DI(Dependency Injection)"
categories: [Spring]
tag: ["Spring", "SpringBoot"]
author_profile: false
---

# 📌 IoC(Inversion of Control), DI

- **IoC(Inversion of Control)는 DI(Dependency Injection,의존성 주입)라고 많이 불린다.**

기존의 코드는 Main에서 프로그래머 코드가 모든 제어를 가지도록 구현을 했다면, 스프링은 프레임워크 위에서 스프링이 제어권을 가지고 프로그래머는 그 위에 조립(구현)해 애플리케이션을 만들면 되는 구조를 제공한다. 스프링이 레고처럼 레고판을 제공해주고 사용자가 판 위에서 레고(애플리케이션)를 조립 하는 것과 비슷하다.

즉, **클라이언트(프로그래머) 코드가 가지고 있던 제어권을 프레임워크가 일부 가져가 제어 역전(Inversion of Control)이 일어났다고 말한다.**

IoC는 객체가 생성자 인수, factory method에 대한 인수 또는 객체 인스턴스가 factory method로부터 생성되거나 반환된 후 객체 인스턴스에 설정된 속성을 통해서만 객체의 종속성(작업하는 다른 객체)을 정의하는 프로세스이다. 그런 다음 컨테이너는 Bean을 만들어 종속성을 주입한다. 이 프로세스는 **클래스의 구성이나 Service Locator Pattern과 같은 메커니즘을 사용해 Bean 스스로 의존성의 위치를 제어하거나 인스턴스화를 제어하기 때문에 IoC(Inversion of Control)라 불린다.**

## **Service Locator Pattern vs. DI**

**Service Locator Pattern**: 요청이 발생하면 서비스 인스턴스를 반환하는 패턴

- Client: Service Locator에 요청을 호출할 책임을 가지는 소비자
- Service Locator: Cache에서 서비스를 반환하기 위한 통신 진입점
- Cache: 서비스 참조를 저장해 나중에 재사용하기 위한 객체
- Initializer: Cache에 서비스에 대한 참조를 생성하고 등록
- Service: Service와 Service Implementation을 나타냄

Service Locator Pattern과 DI(의존성 주입)는 유사하게 보일 수 있다. 두 개념 모두 제어 반점 개념(Inversion of Control)을 구현하기 때문이다.

차이점은 **Service Locator Pattern은 클라이언트 객체가 종속성을 생성**한다는 점이다. 이 종속성은 **Locator 객체에 대한 참조**를 위해 사용된다. 이에 비해 **DI는 클래스에 종속성이 부여**된다. 프로그램이 시작할 때 **클래스에 종속성을 주입하기 위해 Injector가 한 번만 호출**된다.

<br>

# 📌 Bean

## JavaBean

- 데이터를 저장하기 위한 구조체로 자바 빈 규약을 따르는 구조체
- private property, getter/setter 로만 데이터를 접근

```java
public class JavaBeanExample {
    private String id;
    private Integer count;

    public JavaBeanExample() {
    }

    public String getId() {
        return id;
    } 

    public void setId(String id) {
        this.id = id;
    }

    public Integer getCount() {
        return count;
    }

    public void setCount(Integer count) {
        this.count = count;
    }
}
```

## **SpringBean**

- **Spring IoC Container에 의해 인스턴스화, 생성(조립)되고 관리되는 객체**
- annotation을 사용해 등록.
- 자바와 달리 new 키워드로 생성하지 않는다.
- 각각의 **Bean들 끼리는 서로를 편리하게 의존(사용)**할 수 있다. 스프링 컨테이너가 빈 설정을 통해 자동으로 연결해 주기 때문이다.

<br>

# 📌 Spring IoC Container

`org.springframework.context.ApplicationContext` 인터페이스를 통해 제공되는 스프링 컨테이너는 Bean 객체의 생성 및 Bean들의 조립(상호 의존성 관리)및 인스턴스화를 담당한다. 쉽게 말해 Bean의 생성부터 퐈괴 까지 모든 것을 관리하는 큰 용기라 생각하면 된다.

- ApplicationContext 인터페이스: BeanFactory 인터페이스를 확장한 기능으로 Bean을 등록, 생성, 조회, 반환 관리하는 기능과 각종 부가 기능을 제공한다.
  - 부가기능
    - Text Message 관리 기능
    - 파일 자원(이미지 등)을 로드할 수 있는 포괄적인 방법 제공
    - Listener로 등록된 Bean에게 이벤트 발생을 알려주는 기능

## Bean 등록

- 과거에는 XML 설정으로 관리해 등록했지만, 현재는 

  annotation 기반으로 Bean 등록

  - 애노테이션: `@Bean`, `@Controller`, `@Service`

## Bean 등록 시 정보

- XML 설정으로 관리할 때는 Class 경로, Bean의 이름등이 필요했지만 현재는 annotation만 달아주면 스프링이 자동으로 등록
- Class Path
- Bean의 이름
  - 기본적으로는 원 Class 이름에서 첫 문자만 소문자로 변경  → accountService, userDao
  - 원하는 경우 변경 가능
- Scope: Bean을 생성하는 규칙
  - singleton: 컨테이너에 단일로 생성, 가장 많이 사용됨
  - prototype: 작업 시마다 Bean을 생성하고 싶은 경우 사용
  - request: http 요청마다 새롭게 Bean을 생성하고 싶은 경우 사용
  - session: http 세션의 수명 주기에 대한 단일 Bean 생성
    - web-aware Spring ApplicationContext에서만 사용
  - application: ServeletContext의 수명주기에 의해 Bean을 생성하고 싶은 경우 사용
    - web-aware Spring ApplicationContext에서만 사용
  - websocket: WebSocket의 생명주기에 의해 Bean을 생성하고 싶은 경우 사용.
    - web-aware Spring ApplicationContext에서만 사용

## Bean LifeCycle callback

- callback: 어떤 이벤트가 발생하는 경우 호출되는 메서드
- lifecycle callback: Bean을 생성하고 초기화하고 파괴하는 등 특정 시점에 호출되도록 정의된 함수
- 많이 사용되는 callback
  - `@PostConstruct`: Bean 생성 시점에 필요한 작업을 수행
  - `@PreDestroy`: Bean 파괴(주로 애플리케이션 종료) 시점에 필요한 작업을 수행

<br>

> Reference
>
> [Service Locator Pattern](https://www.baeldung.com/java-service-locator-pattern)
>
> [Spring.io/IoC & DI](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans)