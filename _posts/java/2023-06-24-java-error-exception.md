---
title: "[자바] 자바의 Error와 Exception"
categories: [자바]
tag: ["자바", "Java"]
toc: true
toc_sticky: true
author_profile: false
sidebar:
    nav: "counts"
---

> [Java의 Error와 Exception 그리고 예외처리 전략](https://toneyparky.tistory.com/40)
>
> [Exception in Java](https://www.geeksforgeeks.org/exceptions-in-java/)
>
> [Java 예외(Exception) 처리에 대한 작은 생각](https://www.nextree.co.kr/p3239/)

<br>

# 📌 오류(Error)와 예외(Exception)

**오류(Error)**는 컴파일 시 문법적인 오류와 런타임 시 널포인트 참조와 같은 오류로 시스템이 종료되어야 할 수준의 상황과 같이 **수습할 수 없는 심각한 문제**를 의미한다.

- 메모리 부족(OutOfMemory)이나 스택 오버플로우(StackOverflow)같이 발생하면 복구할 수 없는 심각한 상황이 오류이다.
- 애플리케이션이 동작하는 가상머신(JVM)에서 런타임 환경(JRE)문제가 생겼을 때 발생

**예외(Exception)**는 컴퓨터의 시스템 동작 도중 예기치 않았던 이상 상태가 발생해 수행 중인 프로그램이 영향을 받는 것으로 **수습될 수 있는 다소 미약한 문제**를 의미한다. 개발자가 구현한 로직에서 발생한 실수나 사용자의 영향에 의해 발생한다.

- 개발자가 임의로 **예외를 던질 수 있다**
- 프로그래머가 적절히 코드를 작성해주면 비정상적인 종료를 막을 수 있는 상황이 예외 이다.

오류와 예외의 큰 차이는 예상을 할 수 있냐 없냐의 차이다. 오류는 예상하지 못한 상황에서 발생하고 예외는 개발자가 미리 예측해 방지할 수 있기에 상황에 맞는 예외 처리(Exception handling)이 필요하다.

오류는 **발생시점**에 따라 컴파일 오류(compile time error)와 런타임 오류(runtime error)로 나눌 수 있다.

- 컴파일 오류: 클래스 파일 컴파일 시에 발생하는 오류
  - JVM이 던지는 오류
  - 대부분 문법적 오류로 발생한다
  - ClassNotFoundException, IllegalAccessException, NoSuchMethodException
- 런타임 오류: 실행 시에 발생하는 오류
  - 개발자가 직접 오류를 확인해 처리해야 한다
  - NullPointerException, ArithmeticException, IndexOutOfBoundsException
- 논리적 오류(logical error): 실행은 되지만, 의도와 다르게 동작하는 것

소스코드를 컴파일하면 컴파일러가 소스코드에 대해 오타, 구문, 자료형 체크 등의 기본적인 검사를 수행해 오류가 있는지를 알려준다. 이때 발생되는 오류가 컴파일 오류이고, 성공적으로 컴파일 되면 클래스(*.class)파일이 생성되어 클래스 파일을 실행할 수 있게 된다.

컴파일 오류는 컴파일러가 잡아주기 때문에 개발자가 편하게 알아차릴 수 있지만, 런타임 오류는 실행도중에 발생하고 컴파일러가 실행 중 발생하는 잠재적인 오류까지 검사할 수 없기 때문에 개발자는 고려할 수 있는 모든 경우의 수를 고려해 대비해야 한다.

<br>

# 📌 오류와 예외의 계층구조

<center><img src="/assets/images/java/errorexception/1.png"></center>

`Throwable`은 범 예외 클래스들의 공통 조상이다. 모든 예외 클래스들이 가지고 있는 공통된 메서드를 정의하고 있고, `Throwable` 객체에 오류나 예외에 대한 메시지를 담는다. 그리고 예외가 발생했을 때 예외가 발생한 이유나 정보들을 기록하기도 한다. 이 객체가 가진 정보를 이용해 `getMessage()`와 `printStackTrace()`, `toString()`라는 메서드를 사용해 콘솔상 정보를 출력한다.

- `printStackTrace()`: 예외의 이름과 예외가 발생한 이유와 발생 위치를 스택 형식으로 출력
  - 예외 발생시 호출 스택(Call stack)에 있던 메서드의 정보와 예외 메시지를 출력

```java
public class Main {

	public static void main(String[] args) {
		int a = 5;
		int b = 0;

		try {
			System.out.println(a / b);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

}
```

<center><img src="/assets/images/java/errorexception/2.png"></center>

- `getMessage()`: 발생한 예외 클래스의 인스턴스에 저장된 메시지(이유)를 출력

```java
public class Main {

	public static void main(String[] args) {
		int a = 5;
		int b = 0;

		try {
			System.out.println(a / b);
		} catch (Exception e) {
			System.out.println(e.getMessage());
		}
	}

}
```

<center><img src="/assets/images/java/errorexception/3.png"></center>

- `toString()`: 예외의 이름과 발생 이유에 대해서만 출력

```java
public class Main {

	public static void main(String[] args) {
		int a = 5;
		int b = 0;

		try {
			System.out.println(a / b);
		} catch (Exception e) {
			System.out.println(e.toString());
		}
	}

}
```

<center><img src="/assets/images/java/errorexception/4.png"></center>

<br>

# 📌 예외의 종류

Checked Exception과 Unchecked Exception이 존재하며 두 예외를 구분하는 명확한 기준은 **'꼭 처리를 해야 하느냐'**이다

## Checked Exception

RuntimeException을 제외한 Exception의 하위 클래스들이 Checked 예외로 불리며 Compile Exception이라고도 부른다.

Checked 예외는 반드시 예외처리를 해야 하는 예외로 컴파일 시점에 예외를 catch 하는지 정적으로 확인한다, 만약 예외를 처리하지 않으면 컴파일되지 않는다.

트랜잭션 처리 때 예외 발생 시 롤백(rollback)하지 않는다.

주로 JVM 외부와 통신(네트워크, 파일시스템)할 때 주로 발생한다.

- IOException
- SQLException

## Unchecked Exception

RuntimeException과 하위 클래스들이 Unchecked 예외로 불린다. 이 예외들은 반드시 예외처리를 할 필요는 없으며 컴파일 때 체크되지 않고 런타임(Runtime)에 발생하는 예외를 의미한다.

트랜잭션 처리 때 예외 발생 시 롤백(rollback)해야 한다.

예외가 발생하는 곳에서 `throws` 예약어로 예외를 꼭 처리할 필요가 없지만 처리해도 괜찮다. 즉, 명시적으로 예외처리를 강제하지 않는다.

- NullPointerException
- IndexOutOfBoundsException

<br>

# 📌 예외를 처리하는 방법

예외를 처리하는 방법에는 예외 복구, 예외 회피, 예외 전환이 있다

## 예외 복구

예외가 발생하면 다른 작업 흐름으로 유도한다.

- `try~catch` 구문: 예외에 대한 최종적인 책임을 지고 처리한다

```java
int maxretry = MAX_RETRY; // 최대 재시도 횟수

while (maxretry-- > 0) {
	try {
		// 예외가 발생할 가능성이 있는 시도
		return; // 작업 성공시 종료
	} catch (Exception e) {
		// 로그 출력, 정해진 시간만큼 대기
	} finally {
		// 리로스 반납 및 정리 작업
	}

}

throw new RetryFailedException(); // 최대 재시도 횟수를 넘기면 직접 예외 발생
```

예외복구의 핵심은 예외가 발생해도 애플리케이션은 정상적인 흐름으로 진행된다는 것이다.

위 예제는 재시도를 통해 예외를 복구하는 코드다. 네트워크 환경이 좋지 않아 서버 접속이 안되는 상황의 시스템에 적용하면 효율적인데 예외가 발생하면 그 예외를 잡아서 일정 시간만큼 대기하고 다시 재시도를 반복한다. 그리고 최대 횟수를 넘기면 예외를 발생시킨다.

재시도를 통해 정상적인 흐름을 타게 한다거나, 예외가 발생하면 미리 예측해 다른 흐름으로 유도시키도록 구현하면 예외가 발생해도 정상적으로 작업을 종료할 수 있다.

## 예외 회피

예외를 처리하지 않고 호출한 쪽으로 던진다(`throws`).

- `throws Exception`: 발생한 예외의 책임을 호출하는 쪽이 책임지도록 던진다

```java
public void memberJoin() throws SQLException {

}
```

예외가 발생하면 `throws`를 통해 호출한 쪽으로 예외를 던지고 예외 처리를 회피하는 방법이다. 무책임하게 호출한 쪽에 던지는 것은 위험하므로 호출한 쪽에서 다시 예외를 받아 처리하거나, 해당 메서드에서 이 예외를 던지는 것이 최선의 방법일 때 사용해야 한다.

예외를 메서드의 `throws`에 명시하는 것은 예외를 처리하는 것이 아니라, 해당 메서드를 호출한 메서드에게 예외를 전달하여 예외처리를 맡기는 것이다. 이런 식으로 계속 호출스택에 있는 메서드들을 따라 전달되다가 제일 마지막에 있는 `main`에서도 예외 처리가 되지 않으면 프로그램 전체가 종료된다.

## 예외 전환

호출한 쪽으로 던질 때 명확한 의미를 전달하기 위해 다른 예외로 전환해 던진다.

```java
try {
	// 로직
} catch (SQLException e) {
	...
	throw DuplicateUserIdException();
}
```

예외를 잡아서 다른 예외로 던지는 것이 예외 전환이다. 호출한 쪽에서 예외를 받아서 처리할 때 명확하게 인지할 수 있도록 돕기 위한 방법이다. 어떤 예외인지 알아야 처리가 수월해지기 때문이다. 예를 들어 Checked Exception 중 복구가 불가능한 예외가 잡혔다면 이를 Unchecked Exception으로 전환해 다른 계층에서 일일이 예외를 선언할 필요가 없도록 할 수도 있다.

```java
// Checked Exception인 IOException을 Unchecked Exception으로 바꿔 던진다
public String readRequest(BufferedReader bufferedReader) {
	try {
		String request = bufferedReader.readLine();
		if (request == null) {
			throw new NoHttpRequestException("HTTP 요청이 없습니다.");
		}

		return request;
	} catch (IOException e) {
		throw new NoHttpRequestException("입력 값이 잘못되어 요청을 생성할 수 없습니다.");
	}

}
```

예외를 잡고 아무런 처리도 하지 않는 것은 정말 위험한 행위이다. `try-catch` 문으로 예외를 잡고 catch문에서 로그를 찍거나 명확한 예외로 던져줘야 그 원인을 파악하기 쉽다.

어떤 처리를 해야 하는지 모르더라도 무작정 catch하고 무시하거나 throw 해버리는 행위를 할 때는 신중해야 한다.