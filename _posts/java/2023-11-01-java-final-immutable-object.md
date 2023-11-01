---
title: "[자바] final 키워드 & 불변 객체"
categories: [자바]
tag: ["자바", "Java"]
toc: true
toc_sticky: true
author_profile: false
sidebar:
    nav: "counts"
---

> References
>
> - [[10분 테코톡]시카의 Java final과 불변 객체)](https://www.youtube.com/watch?v=ej-bnXlHk-E&ab_channel=우아한테크)
>
> - [불변 객체(Immutable Object) 및 final을 사용해야 하는 이유 - Mangkyu님 블로그](https://mangkyu.tistory.com/131)

<br>

# 📌 final 키워드

- final 키워드: 한 번만 할당 가능하다는 선언
  - 재할당하려고 하면 컴파일 오류가 발생해 바로 확인이 가능
- 왜 final 키워드를 사용해야 할까?
  - 실수로 값을 재할당해 로직이 꼬여 장애가 발생할 수 있다
  - 좋은 코드는 제약을 두어 의도를 드러내야 한다
    - 마치 기본 생성자를 protected, private 으로 설정해 사용하지 못하게 막는것 처럼
  - 개발은 혼자 하는게 아님. 다른 사람이 객체의 값을 변경하거나, 로직에서 파라미터 값을 임의로 재할당해 장애를 일으킬 수 있다
- 불변을 유지하려고 만들면 객체지향적인 코드가 나오기도 한다
- final 적용 효과
  - 서비스 안정성이 높아짐(버그 발생 가능성이 줄어든다, 코드 품질이 높아진다)

<br>

# 📌 불변 객체(Immutable Object)

- 한 번 생성되면 내부의 상태를 변경(추가, 제거, 수정)할 수 없는 객체
  - 상태를 변경할 수 없다? 이는 Heap 메모리 영역에서 객체가 가리키고 있는 데이터 자체의 변화가 불가능하다는 것을 의미
- 불변 객체는 read-only 메서드만을 제공, 객체의 내부 상태를 제공하는 메서드를 제공하지 않거나 방어적 복사(defensive-copy)를 통해 제공
  - 대표적으로 Java의 String 클래스가 불변 객체
  - 방어적 복사: 참조를 통해 값을 전달하지 않고 내부를 복사해 전달
- 불변 객체는 신뢰할 수 있다

```java
public class Money {

	private final long money;

	private Money(final long money) {
		this.money = money;
	}

	public static Money of(final long money) {
		return new Money(money);
	}

}
```

## 불변 객체의 장점

- 멀티 스레드 동기화 문제 방지
- 불변 객체를 사용해라 성능상의 단점은 미미하다. 현대 컴퓨터의 성능을 무시하지 마라 - 제이슨
- 불변 객체는 간단하고 신뢰성 있는 코드를 만들기 위한 전략이다 - 오라클 자바 튜토리얼
- 불변 객체는 가변 객체보다 설계하고 구현하고 사용하기 쉬우며, 오류도 생길 여지도 적고 훨씬 안전하다 - 이펙티브 자바
- 클래스들은 가변적이여야 하는 매우 타당한 이유가 있지 않는 한 반드시 불변으로 만들어야 한다. 만약 클래스를 불변으로 만드는 것이 불가능하다면, 가능한 변경 가능성을 최소화하라. - 이펙티브 자바

<br>

# 📌 불변 객체와 final을 사용해야 하는 이유

> 아직 성능적인 요소를 고려할 단계는 아니다. 불변 객체가 오류를 줄이고 오용을 막는다는 장점에 먼저 집중하자

- 사이드 이펙트를 줄여 오류가능성을 최소화
- 다른 사람이 객체, 메서드, 클래스를 사용할 때 오용을 막을 수 있다
- GC의 성능을 높일 수 있다
- Thread-safe해 병렬 프로그래밍에 유용하며, 동기화를 고려하지 않아도 된다

<br>

# 📌 불변객체를 만드는 방법

Java는 불변성을 제공하기 위해 final 키워드를 제공하고 있다. 변수에 final 키워드를 붙이면 참조값을 변경하지 못하게 만들어 불변성을 확보할 수 있다

1. 클래스를 final로 선언한다
2. 모든 클래스 변수를 private final로 선언한다
3. setter를 사용하지 않고 생성자를 사용한다
4. 참조에 의해 변경가능성을 막기 위해 방어적 복사와 깊은 복사를 이용한다

## primitive type 불변객체 생성

- setter를 생성하지 않는다
- 생성자 주입, 정적 팩토리 메서드로만 필드값을 설정한다

## 참조 타입(reference type) 불변객체 생성

- 생성자를 통해 값을 전달받을 때 new ArrayList<>(cars)를 통해 새로운 값을 참조하도록 복사(방어적 복사)
- getter에서는 Collections의 unmodifiableList() 메서드를 이용해 반환

```java
public class Cars {
	private final List<Car> cars;

	public Cars(List<Car> cars) {
		this.cars = new ArrayList<>(cars);
	}

	public List<Car> getCars() {
		return Collections.unmodifiableList(cars);
	}

}
```