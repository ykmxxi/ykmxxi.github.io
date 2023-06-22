---
title: "[자바] 가변인자(Varargs, Variable Arguments)"
categories: [자바]
tag: ["자바", "Java"]
toc: true
toc_sticky: true
author_profile: false
sidebar:
    nav: "counts"
---

> [자바의 정석, 도우출판, 남궁성 저](https://www.yes24.com/Product/Goods/24259565)
>
> [Varargs in Java, Baeldung](https://www.baeldung.com/java-varargs)

<br>

# 📌 가변인자(Varargs)란?

자바의 가변인자(Varargs, Variable Arguments)는 JDK 1.5 에서 도입된 기능으로 메서드의 **매개변수의 개수를 동적으로 지정**해줄 수 있게 만들어준다.

Java 5 이전에는 동일한 메서드에 대해 필요한 파라미터 개수를 정해 메서드를 오버로딩해 사용해 왔다.

> 메서드 오버로딩의 조건은 메소드 이름이 같아야하고 매개변수의 개수 또는 타입이 달라야 가능하다.

하지만 format, print 같은 기능들은 사용자에 따라 사용하는 매개변수의 개수와 타입이 무한히 달라질 수 있는데, 이 경우 일일이 모두 오버로딩하는 것은 매우 비효율적이다.

그래서 도입된 기능이 **가변인자**다.

```java
public String format() {}

public String format(String str) {}

public String format(String str1, String str2) {}
```

가변인자가 도입되면서 임의 개수의 매개변수를 자동으로 처리할 수 있는 메서드 선언이 가능해져 중복된 코드 작성을 방지할 수 있게 되었다.

```java
public String format(String... strs) {}
```

가변인자는 내부 배열을 사용해 여러 개수의 매개변수를 처리하는데 사용할 때는 일반 배열을 사용할때와 같이 가변인자를 사용하면 된다.

```java
public String format(String... strs) {

	for (String str : strs) {
		...
	}

}
```

실제로 자바의 `printf()` 메서드는 가변인자를 사용해 구현되었다.

<center><img src="/assets/images/java/varargs/1.png"></center>

<br>

# 📌 가변인자 사용법과 주의점

## 가변인자 사용법

- 가변인자 선언: `타입... 변수명`
- 메서드 선언시 가변인자는 매개변수 중에서 제일 마지막에 선언해야 한다
  - 컴파일 에러 발생 : 컴파일러가 가변인자를 구분할 수 없다

```java
public PrintStream printf(String format, Object... args) { ... }

// Compile Error
public PrintStream printf(Object... args, String format) { ... }
```

- 각 메서드는 하나의 가변인자만 가질 수 있다.

<center><img src="/assets/images/java/varargs/2.png"></center>

- 제네릭과 함꼐 사용할 수 있다

```java
<T> void varargsWithGenerics(T... objects) {}

void varargsWithGenerics2(List<String>... data) {}
```

## 가변인자 주의점

가변인자로 선언된 함수를 호출하면 컴파일러가 배열을 함수 실행 직전에 생성하도록 코드를 변경하는데, 이것이 문제를 일으킬 수 있다.

- 가변인자는 내부적으로 배열을 이용한다. 그래서 가변인자가 선언된 메서드를 호출할 때마다 배열이 새로 생성되는데, 편리성을 주지만 비효율이 숨어있으므로 꼭 필요한 경우 가변인자를 사용한다.
- 매개변수의 타입을 배열로 하면, 반드시 인자를 지정해 줘야하기 때문에 가변인자와 차이점이 발생한다. 가변인자는 인자를 생략할 수 있지만, 배열은 인자 생략이 불가능하다.
- 제네릭과 함꼐 사용할 때 치명적인 런타임 예외가 발생할 수 있다. 자바 컴파일러는 안전하지 않은 가변인자 사용에 대해 경고를 준다.
- 힙 오염(Heap Pollution)이 발생할 수 있다
  - 힙 오염(Heap Pollution): JVM의 힙 영역(heap area)이 오염된 상태. 즉, 어떠한 이유로 힙 영역에 문제가 생긴 상태
  - 힙 오염을 해결하기 위해서는 제네릭 타입제한을 이용해 함수를 사용하면 된다

```java
public class Main {

	public static void main(String[] args) {

		String one = firstOfFirst(Arrays.asList("one", "two"), Collections.emptyList());
	}

	static String firstOfFirst(List<String>... strings) {
		List<Integer> ints = Collections.singletonList(42);
		Object[] objects = strings;
		objects[0] = ints; // Heap pollution

		return strings[0].get(0); // ClassCastException
	}

}
```

- 가능하면 가변인자를 사용한 메서드는 오버로딩하지 않는다. 아래 메서드를 실행해 보면 컴파일러는 어떤 메서드를 사용해야 하는지 구분하지 못한다.

```java
void format(String... strings) {}

void format(String s, String... strings) {}
```

<center><img src="/assets/images/java/varargs/3.png"></center>