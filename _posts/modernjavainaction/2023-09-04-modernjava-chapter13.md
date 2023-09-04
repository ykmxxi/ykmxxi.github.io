---
title: "[모던 자바 인 액션] 13. 디폴트 메서드(Default Method)"
categories: [Modern Java in Action]
tag: ["Java"]
author_profile: false
sidebar:
    nav: "counts"
---

> Reference
>
> [모던 자바 인 액션: 람다, 스트림, 함수형, 리액티브 프로그래밍으로 새로워진 자바 마스터하기](https://www.yes24.com/Product/Goods/77125987?pid=123487&cosemkid=go15646485055614872&gclid=Cj0KCQjw2qKmBhCfARIsAFy8buL-QGAkh5m5YK8mj5vAgRgCKdSRIHzDDWdAYOLPFSuQNlzVVcFFdbYaAsh1EALw_wcB)

<br>

전통적인 자바에서 인터페이스와 관련 메서드는 한 몸처럼 구성된다. 인터페이스를 구현한 콘크리트 클래스는 인터페이스에 정의된 모든 메서드를 구현하거나 슈퍼 클래스의 구현을 상속받아야 한다.

평소에는 아무 문제 없지만 여기서 인터페이스에 새로운 메서드를 추가하면 해당 인터페이스를 구현한 모든 콘크리트 클래스를 변경해야 한다.

Java 8에서는 이 문제를 해결하는 새로운 기능을 제공한다. Java 8은 기본 구현을 포함하는 인터페이스를 정의하는 두 가지 방법을 제공한다

1. 인터페이스 내부에 **정적 메서드(static method)**를 사용
   - 유틸리티 클래스를 따로 작성할 필요 없이 인터페이스 내부에서 해결 가능
2. 인터페이스의 기본 구현을 제공할 수 있도록 **디폴트 메서드(default method)** 기능을 사용

즉, Java 8은 메서드 구현을 포함하는 인터페이스를 정의할 수 있다. 결과적으로 기존 인터페이스를 구현한 클래스는 자동으로 추가된 새로운 메서드의 디폴트 메서드를 상속받게 된다.

List 인터페이스의 sort 메서드는 Java 8에서 새로 추가된 메서드다. default 키워드가 해당 메서드가 디폴트 메서드임을 가리키는 키워드이다.

<center><img src="/assets/images/modernjavainaction/ch13/1.png"></center>

그렇다면 디폴트 메서드가 추가되면서 인터페이스와 추상 클래스의 경계가 모호해진 것은 아닌가? 인터페이스와 추상 클래스는 같은 점이 많아 졌지만 여전히 다른 점(인스턴스 필드의 존재 유무, 상속 문제)도 존재한다.

디폴트 메서드를 사용하는 이유는 Java API의 호환성을 유지하면서 라이브러리를 바꿀 수 있기 때문이다. 디폴트 메서드를 사용하면 인터페이스의 기본 구현을 그대로 상속하므로 인터페이스에 자유롭게 메서드를 추가할 수 있게 된다. 또 디폴트 메서드는 다중 상속 동작이라는 유연성을 제공해 프로그램 구성에도 도움을 준다. 이제 클래스는 여러 디폴트 메서드를 상속받을 수 있게 되었다.

> **정적 메서드(static method)와 인터페이스**
>
> 보통 자바는 인터페이스와 인터페이스의 인스턴스를 활용할 수 있는 다양한 정적 메서드를 정의한 유틸리티 클래스를 활용한다. 예를 들어 Collections 클래스는 Collection 객체를 활용할 수 있는 유틸리티 클래스다.
>
> Java 8부터는 인터페이스에 직접 정적 메서드를 선언할 수 있어 유틸리티 클래스를 없애고 직접 인터페이스 내부에 정적 메서드를 구현할 수 있다.

<br>

# 📌 변화하는 API

Java 그리기 라이브러리 설계자가 되었다고 가정한다. 해당 라이브러리에는 모양의 크기를 조절하는 데 필요한 메서드들이 있는 Resizable 인터페이스가 있다. 그뿐 아니라 정사각형, 직사각형 같은 위 인터페이스를 구현한 클래스도 제공한다. 라이브러리가 인기를 얻어 Ellipse 라는 타원형 관련 클래스를 구현하기로 결정되었다.

또 몇 가지 기능이 부족해 기능을 추가하기로 결정이 났다. 새로운 기능을 추가해 라이브러리를 재배포하면 모든 문제가 해결될까? 라이브러리 자체의 문제는 해결되었지만 사용자가 라이브러리 인터페이스를 구현했다면 문제가 발생한다.

- 라이브러리의 초기 버전: v1

```java
public interface Resizable extends Drawable {

	int getWidth();

	int getHeight();

	void setWidth(int width);

	void setHeight(int height);

	void setAbsoluteSize(int width, int height);
	
}
```

- 개선된 라이브러리 버전: v2

```java
public interface Resizable extends Drawable {

	int getWidth();

	int getHeight();

	void setWidth(int width);

	void setHeight(int height);

	void setAbsoluteSize(int width, int height);
	
	void setRelativeSize(int width, int height); // 추가된 기능

}
```

## 사용자가 겪는 문제

먼저 Resizable 인터페이스를 구현한 모든 클래스는 추가된 `setRelativeSize()`메서드를 구현해야 한다. 인터페이스에 새로운 메서드를 추가하면 **바이너리 호환성**은 유지된다. 바이너리 호환성은 새로 추가된 메서드를 호출하지만 않으면 새로운 메서드의 구현 없이 기존 클래스 파일 구현이 잘 동작한다는 의미다. 하지만 언젠가 추가된 메서드가 호출되는 일이 발생하므로 문제가 발생한다.

또 전체 애플리케이션을 재빌드하면 컴파일 에러가 발생한다. 공개된 API가 수정되면 기존 버전과의 호환성 문제가 발생한다. 이런 이유 때문에 기존 API를 고치는 것이 어렵다.

디폴트 메서드를 사용하면 이 모든 문제를 해결할 수 있다. 디폴트 메서드를 이용해 API를 바꾸면 새롭게 바뀐 인터페이스에서 자동으로 기본 구현을 제공하므로 기존 코드를 고치지 않아도 된다.

## 바이너리 호환성, 소스 호환성, 동작 호환성

Java 프로그램을 바꾸는 것과 관련된 호환성 문제는 크게 바이너리 호환성, 소스 호환성, 동작 호환성 세 가지로 분류할 수 있다.

인터페이스에 메서드를 추가했을 때는 바이너리 호환성을 유지하지만 인터페이스를 구현하는 클래스를 재컴파일하면 에러가 발생한다. 즉, 다양한 호환성이 있다는 사실을 이해해야 한다.

- 바이너리 호환성: 변경 이후에도 에러 없이 기존 바이너리가 실행될 수 있는 상황
  - 바이너리 실행에는 인증(verification), 준비(preparation), 해석(resolution) 등의 과정이 포함됨
  - 인터페이스에 메서드를 추가했을 때 추가된 메서드를 호출하지 않는 한 문제가 일어나지 않는 경우가 바이너리 호환성임
- 소스 호환성: 코드를 고쳐도 기존 프로그램을 성공적으로 재컴파일할 수 있는 상황
- 동작 호환성: 코드를 바꾼 다음에도 같은 입력값이 주어지면 프로그램이 같은 동작을 실행하는 상황
  - 인터페이스에 메서드를 추가해도 프로그램에서 추가된 메서드를 호출할 일이 없으면 동작 호환성이 유지됨

<br>

# 📌 디폴트 메서드(Default Method)란?

- **디폴트 메서드**: 호환성을 유지하면서 API를 바꿀 수 있도록 하는 기능

인터페이스는 디폴트 메서드를 도입해 구현 클래스에서 메서드를 구현하지 않을 수 있는 새로운 메서드 시그니처를 제공한다. 인터페이스 자체에서 기본으로 제공하기 때문에 디폴트 메서드라 한다.

**default**라는 키워드로 시작하면 디폴트 메서드이고, 다른 클래스에서 선언된 메서드처럼 메서드 바디를 포함하고 있다. 디폴트 메서드가 있으면 해당 인터페이스를 구현하는 모든 클래스는 디폴트 메서드의 구현도 상속받아 **소스 호환성이 유지**된다.

이제 디폴트 메서드를 이용해 그리기 라이브러리의 문제를 해결한 v3를 구현할 수 있다. 이제 API 설계자들은 사용자들의 호환성을 유지하면서 고칠 수 있고, 사용자들은 인터페이스를 구현한 클래스를 고칠 필요가 없어졌다.

- 디폴트 메서드를 적용한 라이브러리 버전: v3

```java
public interface Resizable extends Drawable {

	int getWidth();

	int getHeight();

	void setWidth(int width);

	void setHeight(int height);

	void setAbsoluteSize(int width, int height);

	default void setRelativeSize(int wFactor, int hFactor) {
		setAbsoluteSize(getWidth() / wFactor, getHeight() / hFactor);
	}

}
```

Java 8의 API에서 디폴트 메서드가 상당히 많이 활용된 것을 확인할 수 있다. 새롭게 추가된 기능들은 디폴트 메서드로 작성되었고, 함수형 인터페이스도 다양한 디폴트 메서드를 포함하고 있다. **디폴트 메서드는 추상 메서드에 해당하지 않으므로** 함수형 인터페이스에 추가해도 된다.

> 함수형 인터페이스에 디폴트 메서드와 정적 메서드를 추가해도 상관 없음. 추상 메서드(abstract method)가 하나만 존재하면 됨

만약 모든 컬렉션에서 사용할 수 있는 `removeIf()` 메서드를 추가해달라는 요청이 오면 다음과 같이 구현하면 된다.

```java
public interface Collection<E> extends Iterable<E> {

	...

	default boolean removeIf(Predicate<? super E> filter) {
		boolean removed = false;
		final Iterator<E> each = iterator();

		while (each.hasNext()) {
			if (filter.test(each.next())) {
				each.remove();
				removed = true;
			}
		}

		return removed;
}
```

## 추상 클래스(abstract class) vs Java 8 인터페이스

추상 클래스와 인터페이스의 차이점은 무엇일까? 둘 다 추상 메서드와 바디를 포함하는 메서드를 정의할 수 있어 큰 차이점이 없어 보인다. 하지만 두 가지 큰 차이점이 존재한다.

1. 클래스는 **하나의 추상 클래스만 상속**받을 수 있지만 **인터페이스는 여러 개를 구현**할 수 있다
2. 추상 클래스는 **인스턴스 변수(필드)로 공통 상태**를 가질 수 있지만 **인터페이스는 인스턴스 변수를 가질 수 없다**.

<br>

# 📌 디폴트 메서드 활용 패턴

디폴트 메서드를 이용하는 **선택형 메서드(optional method)**와 **동작 다중 상속(multiple inheritance of behavior)** 두 가지 방식을 살펴본다.

## 선택형 메서드(Optional Method)

인터페이스를 구현하는 클래스에서 메서드의 내용이 비어있는 상황이 있다. 예를 들어 Java의 Iterator 인터페이스는 `hasNext()`, `next()`, `remove()`를 제공하고 있었다. 그런데 remove 기능은 잘 사용하지 않아 Java 8 이전에는 해당 기능을 무시했다. 결과적으로 Iterator를 구현한 많은 클래스에서는 remove에 빈 구현을 제공했다.

디폴트 메서드를 이용하면 remove 같은 메서드에 기본 구현을 제공할 수 있어 구현 클래스에서 빈 구현을 제공할 필요가 없다. 실제로 Java 8에서는 remove를 디폴트 메서드로 변경해 제공하기 시작했다.

<center><img src="/assets/images/modernjavainaction/ch13/2.png"></center>

- **기본 구현이 제공되어 구현 클래스는 불필요한 코드를 줄일 수 있다**

## 동작 다중 상속**(Multiple Inheritance of Behavior)**

디폴트 메서드를 이용하면 기존에 불가능했던 동작 다중 상속 기능도 구현할 수 있다. 클래스는 다중 상속을 이용해 기존 코드를 재사용할 수 있다.

Java에서 클래스는 한 개의 다른 클래스만 상속할 수 있지만 인터페이스는 여러 개 구현할 수 있다.

### 다중 상속 형식

<center><img src="/assets/images/modernjavainaction/ch13/3.png"></center>

ArrayList는 한 개의 클래스를 상속 받고 여섯 개의 인터페이스를 구현한다. 따라서 ArrayList는 AbstracList, List, RandomAccess 등의 **서브형식(subtype)**이 된다. 따라서 디폴트 메서드를 사용하지 않아도 다중 상속을 활용할 수 있다.

Java 8은 인터페이스가 구현을 포함할 수 있어 한 클래스가 여러 인터페이스에서 동작(구현 코드)을 상속받을 수 있다. 즉, 중복되지 않는 최소한의 인터페이스를 유지하면 동작을 쉽게 재사용하고 조합할 수 있다

기능이 중복되지 않는 인터페이스들을 조합해서 한 클래스가 상속 받으면 원하는 기능을 따로 구현할 필요 없이 사용할 수 있다. 또 어떤 기능을 좀 더 효율적으로 고친다면 따로 구현 클래스에서 오버라이딩 하지 않은 이상 자동으로 변경한 코드를 상속받아 사용할 수 있다. 즉, 변화에 쉽게 대응할 수 있게 된다

### 옳지 못한 상속

상속으로 모든 코드 재사용 문제를 해결할 수 있는 것은 아니다. 만약 한 개의 메서드를 재사용하기 위해 100개의 메서드와 필드가 정의되어 있는 클래스를 상속받는 것은 좋은 상속이 아니다.

이럴 때는 델리게이션(delegation), 즉 멤버 변수를 이용해 클래스에서 필요한 메서드를 직접 호출하는 메서드를 작성하는 것이 좋다. (상속 보다는 위임을 사용한다)

종종 final로 선언된 클래스를 볼 수 있는데, 이런 클래스는 상속을 허용하지 않아 원래 동작이 바뀌지 않길 원한다.

<center><img src="/assets/images/modernjavainaction/ch13/4.png"></center>

- Java의 String 클래스가 final로 선언해 핵심 기능을 바꾸지 못하도록 제한

이런 규칙을 디폴트 메서드에도 적용할 수 있다. 필요한 기능만 포함하도록 인터페이스를 최소한으로 유지한다면 필요한 기능만 선택할 수 있어 쉽게 기능을 조립할 수 있다.

<br>

# 📌 해석 규칙

만약 어떤 클래스가 같은 디폴트 메서드 시그니처를 포함하는 두 인터페이스를 구현하는 상황이라면 어떻게 될까? 클래스는 어떤 인터페이스의 디폴트 메서드를 사용하게 되는 것일까.

이런 상황이 자주 일어나지는 않지만 이런 문제를 해결할 수 있는 규칙이 필요하다. Java 컴파일러가 이러한 문제를 어떻게 해결하는지 알아본다.

아래 문제는 C++의 유명한 다이아몬드 문제이다. 같은 시그니처를 갖는 두 메서드를 상속받는 클래스가 존재하는데, 이때 어떤 메서드가 사용될까?

```java
public interface A {

	default void hello() {
		System.out.println("Hello From A");
	}

}

public interface B {

	default void hello() {
		System.out.println("Hello From B");
	}

}

public class C implements B, A {

	public static void main(String[] args) {
		new C().hello();
	}

}
```

## 세 가지 해결 규칙

다른 클래스나 인터페이스로부터 같은 시그니처를 갖는 메서드를 상속받을 때 세 가지 규칙을 따라야 한다

1. 클래스가 항상 이긴다. 클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖는다
   - 클래스의 메서드 구현 > 디폴트 메서드
2. 1번 규칙 이외의 상황에서는 서브인터페이스가 이긴다. 상속관계를 갖는 인터페이스에서 같은 시그니처를 갖는 메서드를 정의할 때는 서브인터페이스가 이긴다. 즉, 위 예제에서 B가 A를 상속받는다면 B가 A를 이긴다
3. 디폴트 메서드의 우선순위가 결정되지 않았다면 여러 인터페이스를 상속받는 클래스가 **명시적으로 디폴트 메서드를 오버라이드하고 호출**해야 한다

## 디폴트 메서드를 제공하는 서브인터페이스가 이긴다

```java
public interface A {

	default void hello() {
		System.out.println("Hello From A");
	}

}

public interface B extends A {

	default void hello() {
		System.out.println("Hello From B");
	}

}

public class C implements B, A {

	public static void main(String[] args) {
		new C().hello();
	}

}
```

위 예제에서는 2번 규칙에 따라 서브인터페이스의 디폴트 메서드가 호출된다. 즉, B가 A를 상속받았으므로 컴파일러는 B의 메서드를 선택한다.

<center><img src="/assets/images/modernjavainaction/ch13/5.png"></center>

그렇다면 여기서 인터페이스 A를 구현한 클래스 D를 추가하고, C가 D클래스를 상속받는다면 어떻게 될까?

```java
class D implements A {}

class C extends D implements B, A {}
```

1번 규칙에서 클래스의 메서드 구현이 이긴다고 설명하는데, D는 hello() 메서드를 오버라이드 하지 않았고 단순히 A 인터페이스를 구현했다. 따라서 D는 인터페이스 A의 디폴트 메서드 구현을 상속받아 1번 규칙 이외의 상황이 된다. 클래스나 슈퍼클래스에 메서드 정의가 없어 서브인터페이스가 선택되어 이번에도 B의 메서드가 호출된다.

<center><img src="/assets/images/modernjavainaction/ch13/6.png"></center>

여기서 만약 D가 인터페이스 A의 메서드를 오버라이드한다면 D의 메서드가 호출된다.

```java
public class D implements A {

	@Override
	public void hello() {
		System.out.println("Hello From D");
	}

}
```

<center><img src="/assets/images/modernjavainaction/ch13/7.png"></center>

## 충돌, 명시적인 문제 해결

만약 1번, 2번 규칙으로 문제를 해결할 수 없는 상황은 어떻게 될까? 인터페이스 B가 A를 상속받지 않았다고 가정해 본다. 규칙을 적용할 수 없어 컴파일러는 어떤 메서드를 호출해야 할지 알 수 없어 에러가 발생한다.

```java
public interface A {

	default void hello() {
		System.out.println("Hello From A");
	}

}

public interface B {

	default void hello() {
		System.out.println("Hello From B");
	}

}

public class C implements B, A {

	public static void main(String[] args) {
		new C().hello();
	}

}
```

<center><img src="/assets/images/modernjavainaction/ch13/8.png"></center>

- 유일한 해결책은 클래스 C에서 메서드를 오버라이드한 다음 호출하려는 메서드를 명시적으로 선택해야 한다
  - `X.super.m()`: X는 호출하려는 메서드 m의 슈퍼인터페이스

```java
public class C implements B, A {

	@Override public void hello() {
		B.super.hello(); // 명시적으로 인터페이스 B의 메서드를 선택
	}

}
```

## 다이아몬드 문제

```java
public interface A {

	default void hello() {
		System.out.println("Hello From A");
	}
}

public interface B extends A {

	default void hello() {
		System.out.println("Hello From B");
	}
}

public interface C extends A {

	// 추상 메서드 hello(), 디폴트 메서드 XX
	void hello();
}

public class D implements B, C {

	public static void main(String[] args) {
		new D().hello();
	}

}
```

다음처럼 인터페이스 C에 추상 메서드 hello()가 존재하면 어떻게 될까? C는 A를 상속받으므로 C의 추상 메서드 hello가 A의 디폴트 메서드보다 우선권을 갖는다. 따라서 D의 추상 메서드가 우선권을 갖는데, 여기서 컴파일 에러가 발생한다. 클래스 D가 어떤 hello를 사용할지 명시적으로 선택해서 해결해야 한다.

<br>

# 📌 Summary

- Java 8의 인터페이스는 구현 코드를 포함하는 디폴트 메서드, 정적 메서드를 정의할 수 있다
- 디폴트 메서드의 정의는 default 키워드로 시작하며 일반 클래스 메서드처럼 바디를 갖는다
- 공개된 인터페이스에 추상 메서드를 추가하면 소스 호환성이 깨진다
- 디폴트 메서드 덕분에 라이브러리 설계자가 API를 바꿔도 기존 버전과 호환성을 유지할 수 있다
- 선택형 메서드와 동작 다중 상속에도 디폴트 메서드를 사용할 수 있다
- 클래스가 같은 시그니처를 갖는 여러 디폴트 메서드를 상속하면서 생기는 충돌 문제를 해결하는 규칙이 있다
- 클래스나 슈퍼클래스에 정의된 메서드가 다른 디폴트 메서드 정의보다 우선한다. 이 외의 상황에서는 서브인터페이스의 디폴트 메서드가 선택된다
- 두 메서드의 시그니처가 같고, 상속관계로도 충돌 문제를 해결할 수 없을 때는 디폴트 메서드를 사용하는 클래스에서 메서드를 오버라이드해서 어떤 디폴트 메서드를 호출할지 명시적으로 결정해야 한다