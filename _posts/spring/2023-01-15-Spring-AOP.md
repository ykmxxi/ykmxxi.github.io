---
title: "[Spring] AOP (Aspect Oriented Programming)"
categories: [Spring]
tag: ["Spring", "SpringBoot"]
toc: true
toc_sticky: true
author_profile: false
sidebar:
    nav: "docs"
---

# 📌 AOP(관점 지향 프로그래밍)

특정 함수 호출 전이나 호출 후에 **공통적인 처리가 필요한 상황에서 AOP**를 사용한다. 즉, 특정 시점, 부분에 원하는 기능을 사용하고 싶을 때 사용한다.

- 로깅(logging)
  - 특정 함수 호출시 log를 사용하고 싶을 때 사용
- 트랜잭션(transaction) 관리: AOP 적용의 대표적인 예제
- 인증(certification)

OOP로 처리하기에는 다소 까다로운 부분을 AOP 처리 방식을 도입해 손쉽게 공통 기능을 추가, 수정, 삭제 할 수 있도록 한다.

- OOP의 모듈화 핵심 단위가 class이면 **AOP의 모듈화 핵심 단위는 관점(Aspect)**이다.
- AOP는 Spring IoC 컨테이너를 보완해 매우 유능한 미들웨어 솔루션을 제공한다.
  - 미들웨어(middleware): OS와 애플리케이션의 중간에서 조정과 중개의 역할을 수행하는 SW, 양 쪽을 연결해 데이터를 주고 받을 수 있도록 매개 역할을 한다.

<br>

# 📌 AOP의 기본 개념

## Aspect

- 여러 클래스나 기능, 객체에 걸쳐 존재하는 관심사(문제), 이 관심사들을 모듈화함
- AOP 중에서 가장 많이 활용되는 부분은 `@Transactional`(트랜잭션 관리) 기능
- 공통 관심 사항(cross-cutting concern)과 핵심 관심 사항(core concern)을 분리
  - 트랜잭션 관리 기능은 공통 관심 사항(crosscutting concern)의 대표적인 예제
- `@Aspect` 애노테이션을 달아 선언

## Advice

- AOP에서 실제로 적용하는 기능(로깅, 트랜잭션 인증 등)을 뜻함.
- 특정 Join Point에서 Aspect가 취하는 행동을 의미한다.

## Join Point

- 모듈화된 특정 기능이 실행될 수 있는 연결 포인트(연결 가능 지점)
- 메서드 실행 또는 예외 처리와 같은 프로그램 실행 중 포인트 지점, Spring AOP에서는 항상 메서드 실행을 나타낸다.

## Pointcut

- Join Point 중 해당 관점(Aspect)을 적용할 대상을 뽑는 조건식
- Advice와 연관되어 Pointcut과 일치하는 모든 Join Point에서 Advice가 일어난다.
- **AOP의 핵심이며 Spring은 기본적으로 AspectJ 표현식을 사용한다.**

## Target Object

- Advice가 적용될 대상 객체(Object)
- Spring AOP는 Runtime Proxy를 사용해 구현되므로 Target Object는 항상 프록시된(proxied) 객체이다.

## AOP Proxy

- Target Object에 관점을 적용하는 경우 Advice를 덧붙이기 위해 하는 작업을 Proxy라고 한다.
- 주로 CGLIB(Code Generation Library) Proxy를 사용해여 프록싱 처리를 한다.
  - CGLIB: 실행 중에 실시간으로 코드를 생성하는 라이브러리

## Weaving

- Advice를 비즈니스 로직 코드에 삽입하는 것을 말한다.
- AspectJ 컴파일러를 사용하면 컴파일 시간에 위빙을 수행한다.
  - load time이나 run time중에도 위빙이 수행될 수 있다.
  - Pure Java AOP, Spring AOP는 run time에 위빙을 수행한다.

<br>

# 📌 AspectJ 지원

- **AspectJ**: AOP를 제대로 사용하기 위해 꼭 필요한 라이브러리.

기본적으로 제공되는 Spring AOP는 다양한 기법의 AOP를 사용할 수 없어 꼭 필요한 라이브러리이다.

## Aspect의 생성

- `@Aspect` 애노테이션을 붙여야 한다.

```java
package org.xyz;
import org.aspectj.lang.annotation.Aspect;

@Aspect
@Component  // Component를 붙인 것은 해당 Aspect를 스프링의 Bean으로 등록해서 사용하기 위함
public class UsefulAspect {

}
```

## Pointcut 선언

- 해당 Aspect의 Advice가 적용될 Join Point를 찾기 위한 패턴(또는 조건 생성)

```java
package org.xyz;
import org.aspectj.lang.annotation.Aspect;

@Aspect
@Component
public class UsefulAspect {

	@Pointcut("execution(* transfer(..))") // Pointcut 표현
	private void anyOldTransfer() {} // Pointcut signature
}
```

## Pointcut 결합

```java
package org.xyz;
import org.aspectj.lang.annotation.Aspect;

@Aspect
@Component
public class UsefulAspect {

	@Pointcut("execution(public * *(..))")
	private void anyPublicOperation() {} //public 메서드 대상 포인트 컷

	@Pointcut("within(com.xyz.myapp.trading..*)")
	private void inTrading() {} // 특정 패키지 대상 포인트 컷, 메서드 실행이 com.xyz.myapp.trading 밑에 있으면 일치
	
	@Pointcut("anyPublicOperation() && inTrading()")
	private void tradingOperation() {} // 위의 두 조건을 and(&&) 조건으로 결합한 포인트 컷
}
```

- execution: 메서드 실행 Join point를 일치시키기 위해 사용. 기본 Pointcut 지정자
- within: 특정 유형 내의 Join point에 대한 일치를 제한시킴.

<br>

# 📌 Advice 정의

Pointcut을 활용해 사용 전(Before), 후(After), 주변(Around)에서 실행될 액션(기능)들을 정의함.

- Spring을 포함한 많은 AOP 프레임워크는 Advice를 인터셉터(Interceptor)로 모델링하고 Join Point 주위에 인터셉터 체인(Chain of Interceptors)을 유지한다.

## Before Advice

- 미리 정의된 Pointcut의 바로 전에 실행되는 Advice
  - dataAccessOperation() 이 실행되기 전에 doAccessCheck()가 실행 됨.

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("com.xyz.myapp.CommonPointcuts.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }
}
```

## After Returning Advice

- 미리 정의된 Pointcut에서 return이 발생된 후 실행되는 Advice
  - dataAccessOperation()에서 return이 발생하면 doAccessCheck()가 실행 됨.

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning("com.xyz.myapp.CommonPointcuts.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }
}
```

## Around Advice

- 미리 정의된 Pointcut의 전, 후에 필요한 동작을 추가
  - businessService()의 실행 전, 후에 doBasicProfiling()가 실행 됨.
- Join Point로 진행할지, 자체 반환 값을 반환하거나 예외를 던져 실행을 단축할지의 여부를 선택하는 책임을 가진 중요한 Advice.

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.ProceedingJoinPoint;

@Aspect
public class AroundExample {

    @Around("com.xyz.myapp.CommonPointcuts.businessService()")
    public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
        // start stopwatch
        Object retVal = pjp.proceed();
        // stop stopwatch
        return retVal;
    }
}
```

> 이외에도 After throwing advice, After (finally) advice가 있다.
>
> - After throwing advice: 미리 정의된 Pointcut이 예외를 발생시켜 종료하는 경우 실행 됨.
> - After (finally) advice: 미리 정의된 Pointcut이 정상(returning) 또는 예외를 반환하는 상황과 관계없이 실행 됨.

<br>

> Reference
>
> [Spring.io/AOP](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop)