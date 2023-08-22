---
title: "[모던 자바 인 액션] 11. null 대신 Optional 클래스"
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

모든 자바 개발자가 만나는 `NullPointerException`은 초급자, 중급자, 상급자를 가리지 않는다

`null`은 컴파일러의 자동 확인 기능으로 모든 참조를 안전하게 사용할 수 있을 것을 목표로, `null` 참조 및 예외로 값이 없는 상황을 가장 단순하게 구현할 수 있다는 판단 아래 탄생했다.

최근 수십 년간 탄생한 대부분의 언어는 `null` 참조 개념을 포함하는데, 예전 언어와의 호환성 문제도 있지만 언어를 구현하기 쉽기 때문에 `null` 참조 개념을 포함했을 가능성이 높다

# 📌 값이 없는 상황을 어떻게 처리할까?

```java
public class Person {

	private Car car;

	public Car getCar() {
		return car;
	}

}

public class Car {

	private Insurance insurance;

	public Insurance getInsurance() {
		return insurance;
	}

}

public class Insurance {

	private String name;

	public String getName() {
		return name;
	}

}

// 이 코드는 문제가 있나?
new Person().getCar().getInsurance().getName();
```

<center><img src="/assets/images/modernjavainaction/ch11/1.png"></center>

위 코드를 보면 아무런 문제가 없는 것처럼 보일 수 있다. 하지만 실제 세계에서는 모든 사람이 차를 소유하지 않았고, 또 차를 소유한 사람 중 보험을 들지 않은 사람도 있다.

대부분의 프로그래머는 `null` 참조를 반환하는 방식으로 값이 없는 상황을 표현할 것이다. 그러면 런타임에 `NullPointerException`이 발생하면서 프로그램이 종료된다.

## 보수적인 자세로 NullPointerException 줄이기

예기치 않은 `NullPointerException`을 피하려면 어떻게 해야 하나? 대부분 프로그래머는 `null` 확인 코드를 추가해 문제를 해결할 것이다.

- **깊은 의심(Deep Doubt)**: 변수가 `null`인지 의심하므로 변수를 접근할 때마다 중첩된 if 문이 추가되면서 메서드의 들여쓰기 레벨이 증가하는 반복 패턴 코드
  - 변수를 참조할 때마다 `null`을 확인
  - 코드의 구조가 엉망이 되고 가독성이 떨어짐

```java
// null 안전 시도 1: 깊은 의심, 필요한 곳에 null 체크 코드를 넣는다
public String getCarInsuranceName(Person person) {
	if (person != null) {
		Car car = person.getCar();
		if (car != null) {
			Insurance insurance = car.getInsurance();
			if (insurance != null) {
				return insurance.getName();
			}
		}
	}

	return "Unknown";
}
```

여기서 보험 회사의 이름은 반드시 존재하기 때문에, 우리가 확실히 알고 있는 영역을 모델링할 때는 `null` 확인을 생략할 수 있다.

하지만 데이터를 자바 클래스로 모델링할 때는 반드시 존재하는지 없는지 사실을 단정하기는 어렵다.

- **너무 많은 출구(early return)**: 변수가 있으면 즉시 `Unknown`을 early return 하는 방식으로 중첩 if 블록을 없애 들여쓰기 레벨을 줄였다

```java
// null 안전 시도 2: 너무 많은 출구
public static String getCarInsuranceName2(Person person) {
	if (person == null) {
		return "Unknown";
	}

	Car car = person.getCar();
	if (car == null) {
		return "Unknown";
	}

	Insurance insurance = car.getInsurance();
	if (insurance == null) {
		return "Unknown";
	}
	return insurance.getName();
}
```

하지만 이 방식도 여러 출구가 생겨 유지보수가 어렵다. 또 쉽게 에러를 일으킬 수 있다.

- `null`**로 값의 존재 유무를 표현하는 것은 좋은 방법이 아니다. 따라서 값의 존재 유무를 표현할 수 있는 다른 방법이 필요하다.**

## null 때문에 발생하는 문제

Java에서 `null` 참조를 사용하면 발생할 수 있는 문제는 다음과 같다

- **에러의 근원**: `NullPointerException`은 가장 많이 발생하는 에러 중 하나다
- **더러운 코드를 만든다**: 중첩된 `null` 확인 코드로 가독성이 떨어진다
- **아무 의미가 없다**: `null`은 아무 의미도 표현하지 않는다. 특히 정적 형식 언어에서 값이 없음을 표현하는 방법으로 적절하지 않다
- **Java 철학에 위배된다**: Java는 개발자로부터 포인터를 숨기도록 설계된 언어이다. 하지만 예외가 있는데 그것이 `null` 포인터다
- **형식 시스템에 구멍을 만든다**: `null`은 무형식에 정보를 포함하고 있지 않아 모든 참조 형식에 할당 가능하다. 이런 식으로 `null`이 할당되기 시작하면 시스템의 다른 부분으로 `null`이 퍼졌을 때 어떤 의미로 사용되었는지 알 수 없다

## 다른 언어는 null 대신 무엇을 사용할까?

그루비(Groovy) 언어는 안전 내비게이션 연산자(Safe Navigation Operator) `?..` 를 도입해 `null` 문제를 해결했다.

```groovy
def carInsuranceName = person?.car?.insurance?.name
```

안전 내비게이션 연산자를 이용하면 `null` 참조 예외 걱정 없이 객체에 접근 가능하다. 이때 호출 체인에 `null`인 참조가 있으면 결과로 `null`이 반환된다

하스켈, 스칼라 등의 함수형 언어는 `null` 문제를 완전 다른 관점에서 접근한다. 하스켈은 Maybe라는 **선택형 값(Optional Value)** 형식을 제공해 `null` 참조 개념을 자연스럽게 없앴다. 선택형 값은 값을 가질수도 있고 아무 값도 갖지 않을 수 있다. 스칼라도 Option[T]라는 구조를 제공해 null 관련 문제가 일어날 가능성을 줄였다.

그렇다면 Java 8은 어떤 기능을 제공할까?

Java 8은 **선택형 값** 개념의 영향을 받아 `java.util.Optional<T>`라는 새로운 클래스를 제공한다.

<br>

# 📌 Optional 클래스

- `java.util.Optional<T>`: 선택형 값을 캡슐화하는 클래스

Java 8은 하스켈과 스칼라의 영향을 받아 선택형 값 클래스를 제공한다. 예를 들어 어떤 사람이 차를 소유하고 있지 않다면 `null`을 할당하지 않고 `Optional<Car>`로 설정할 수 있다.

**Optional 클래스는 더 좋은 API를 설계하고 NullPointerException을 줄이는 데 도움을 준다**

<center><img src="/assets/images/modernjavainaction/ch11/2.png"></center>

값이 있으면 Optional 클래스는 값을 감싼다. 반면 값이 없으면 `Optional.empty()`로 Optional을 반환한다.

`Optional.empty()`는 특별한 싱글턴 인스턴스를 반환하는 정적 팩토리 메서드로 의미상 `null` 참조와 비슷하지만 실제로는 차이점이 많다.

`null`을 참조하려 하면 `NullPointerException`이 발생하지만, `Optional.empty()`는 Optional 객체이므로 이를 다양한 방식으로 활용할 수 있다.

<center><img src="/assets/images/modernjavainaction/ch11/3.png"></center>

Optional 클래스를 사용하면 값이 없을 수 있음을 명시적으로 보여줄 수 있다. 반면 일반 객체를 사용하면 해당 객체에 `null` 참조가 할당될 수 있는데 이것이 올바른 값인지 아니면 잘못된 값인지 판단할 근거가 없다.

Optional을 이용하면 값이 없는 상황이 우리 데이터에 문제가 있는 것인지 아니면 알고리즘의 버그인지 명확하게 구분할 수 있다. 그렇다고 **모든 null 참조를 Optional로 대치하는 것은 바람직하지 않다.**

Optional의 역할은 더 **이해하기 쉬운 API를 설계하도록 돕는 것**이다. 즉, 메서드의 시그니처만 보고도 선택형 값인지 여부를 구분할 수 있다. Optional이 등장하면 이를 벗겨 값이 없을 수 있는 상황에 적절하게 대응하도록 강제하는 효과가 있다.

<br>

# 📌 Optional 적용 패턴

Optional 형식을 이용하면 도메인 모델의 의미를 더 명확하게 만들 수 있고 null 참조 대신 값이 없는 상황을 표현할 수 있다. 그렇다면 Optional로 감싼 값은 어떻게 사용할 수 있을까?

## Optional 객체 만들기

Optional을 사용하려면 Optional 객체를 생성해야 한다. Optional 객체는 다양한 방법으로 생성할 수 있다

### 빈 Optional

- 정적 팩토리 메서드 `Optional.empty()`로 빈 Optional 객체를 얻을 수 있다

```java
Optional<Car> optCar = Optional.empty();
```

### null이 아닌 값으로 Optional 생성

- 정적 팩토리 메서드 `Optional.of()`로 null이 아닌 값을 포함하는 Optional 객체를 생성
- 이때 null 값을 넣으면 `NullPointerException`이 발생

```java
Optional<Car> optCar = Optional.of(new Car());
```

<center><img src="/assets/images/modernjavainaction/ch11/4.png"></center>

<center><img src="/assets/images/modernjavainaction/ch11/5.png"></center>

### null 값으로 Optional 생성

- 정적 팩토리 메서드 `Optional.ofNullable()`로 null값을 저장할 수 있는 Optional 객체를 생성
  - null 값을 넣으면 `Optional.empty()`와 같음
  - null이 아닌 값을 넣으면 `Optional.of()`와 같음
  - `return value == null ? empty() : of(value)`

```java
Optional<Car> optCar = Optional.ofNullable(null);
```

<center><img src="/assets/images/modernjavainaction/ch11/6.png"></center>

Optional은 `get()` 메서드를 통해 값을 가져올 수 있는데, 비어있을 때 호춯하면 예외가 발생한다. 즉, Optional을 잘못 사용하면 결국 null을 사용했을 때와 같은 문제가 발생한다. 따라서 먼저 Optional로 명시적인 검사를 제거할 수 있는 방법을 알아야 한다.

## map으로 Optional의 값을 추출하고 변환하기

보통 객체의 정보를 추출할 때 Optional을 많이 사용한다

```java
String name = null;
if (insurance != null) {
	name = insurance.getName();
}
```

이런 유형의 패턴에 사용할 수 있도록 Optional은 `map()` 메서드를 지원한다

```java
Optional<Insurace> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

Optional의 map 메서드는 스트림의 map 메서드와 개념적으로 비슷하다. 스트림의 map은 스트림의 각 요소에 제공된 함수를 적용하는 연산이다.

여기서 Optional 객체를 최대 요소의 개수가 한 개 이하인 데이터 컬렉션으로 생각할 수 있는데, Optional이 값을 포함하면 map의 인수로 제공된 함수가 값을 바꾸고, 비어있으면 아무 일도 일어나지 않는다.

<center><img src="/assets/images/modernjavainaction/ch11/7.png"></center>

## flatMap으로 Optional 객체 연결

만약 여러 메서드를 안전하게 호출하기 위해서는 어떻게 해야 할까?

```java
person.getCar().getInsurance().getName();

// map을 이용하기: 컴파일 에러 발생
Optional<Person> optPerson = Optional.of(new Person());
Optional<String> name =
	optPerson.map(Person::getCar)
		.map(Car::getInsurance)
		.map(Insurance::getName);
```

map을 이용하면 컴파일 에러가 발생한다. 그 이유는 변수 optPerson의 형식은 `Optional<Person>`으로 `map()`을 호출할 수 있지만, `getCar()`는 `Optional<Car>`을 반환하기 때문에 `Optional<Optional<Car>>` 형식의 객체를 반환하기 때문이다.

또 `getInsurance()` 또한 Optional 객체를 반환하기 때문에 중첩 Optional 객체 구조가 발생한다.

스트림의 `flatMap()`을 보면 함수를 인수로 받아서 다른 스트림을 반환한다. 보통 인수로 받음 함수를 스트림의 각 요소에 적용하면 스트림의 스트림이 생성되는데, `flatMap()`은 인수로 받은 함수를 적용해 생성된 각각의 스트림에서 값만 남긴다. 즉, 함수를 적용해 모든 스트림이 하나의 스트림으로 병합되어 평준화된다.

이렇게 `flatMap()`을 사용하면 Optional 객체도 평준화시킬 수 있다. 평준화 과정이란 두 Optional을 합치는 기능을 수행하면서 둘 중 하나라도 null이면 빈 Optional을 생성하는 연산이다.

- 빈 Optional에 `flatMap()`을 호출하면 아무런 일도 일어나지 않는다
- 호출 체인으로 이어진 경우 어떤 메서드가 빈 Optional을 반환하면 전체 결과로 빈 Optiona이 반환

```java
public static String getCarInsuranceName(Optional<Person> person) {
	return person.flatMap(Person::getCar)
		.flatMap(Car::getInsurance)
		.map(Insurance::getName) // getName()은 String을 반환하므로 flatMap을 사용할 필요 X
		.orElse("Unknown"); // 연산 결과 Optional이 비어있으면 기본값을 반환
}
```

Optional을 이용해 값이 없는 상황을 처리하는 것이 어떤 장점을 제공하는지 확인할 수 있다. 즉, null 체크를 위해 분기문을 추가하지 않고도 쉽게 이해할 수 있는 코드를 완성할 수 있다.

Optional을 사용하면 도메인 모델과 관련한 암묵적인 지식에 의존하지 않고 명시적으로 형식 시스템을 정의할 수 있다.

> Java 언어 설계자는 Optional의 용도를 **“선택형 반환값 지원”**라고 명확하게 못박았다.
>
> 따라서 Optional 클래스는 필드 형식으로 사용할 것을 가정하지 않아 `Serializable` 인터페이스를 구현하지 않는다. 따라서 도메인 모델에 Optional을 사용하면 직렬화 모델을 사용하는 도구나 프레임워크레 문제가 생길 수 있다.
>
> 하지만 이런 단점에도 불구하고 Optional을 사용해 도메인 모델을 구성하는 것이 바람직하다고 생각할 수 있다. 특히 객체 그래프에서 일부 또는 전체가 null인 경우라면 더욱 그렇다.
>
> 직렬화 모델이 필요하다면 Optional로 값을 반환받을 수 있는 메서드를 추가하는 방식이 권장된다
>
> ```java
> public class Person {
> 
> 	private Car car;
> 
> 	public Optional<Car> getCarAsOptional() {
> 		return Optional.ofNullable(car);
> 	}
> }
> ```

## Optional 스트림 조작

Java 9는 Optional을 포함하는 스트림을 쉽게 처리할 수 있도록 `stream()` 메서드를 추가했다. 이제 Optional 스트림을 값을 가진 스트림으로 변환할 때 이 기능을 활용할 수 있다

```java
public static Set<String> getCarInsuranceNames(List<Person> persons) {
	return persons.stream()
		.map(Person::getCar) // 사람 목록을 각 사람이 보유한 자동차의 Optional<Car> 스트림으로 변환
		.map(optCar -> optCar.flatMap(Car::getInsurance)) // flatMpa()으로 Optional<Insurance>로 변환
		.map(optIns -> optIns.map(Insurance::getName)) // Optional<Insurance>를 Optional<String>으로 매핑
		.flatMap(Optional::stream) // Stream<String>으로 변환
		.collect(Collectors.toSet()); // 결과 문자열을 Set으로 수집
}
```

보통 스트림 요소를 조작하려면 변환, 필터 등의 긴 체인이 필요한데 Optional로 값이 감싸져 있어 체인 과정이 더 복잡해진다. Optional 객체를 반환하기 때문에 값을 가지고 있을 수도 있고 값이 없을 수도 있기 때문이다.

Optional 덕분에 null 걱정없이 안전하게 처리할 수 있지만 빈 Optional을 제거하고 값을 unwrap해야 결과를 얻을 수 있다는 것이 문제다.

하지만 `flatMap()` 메서드를 사용하면 한 수준인 평면 스트림으로 바꿀 수 있어 한 단계의 연산으로 값을 포함하는 Optional을 unwrap하고 비어있는 Optional은 건너뛸 수 있다

## 디폴트 액션과 Optional Unwrap

- `get()`: Optional 값을 읽는 가장 간단한 메서드
  - 가장 위험한 메서드
  - 값이 있으면 해당 값을 반환
  - 값이 없으면 `NoSuchElementException` 발생
- `orElse(T other)`: Optional이 값이 있으면 해당 값을 반환하고 값을 포함하지 않을 때 기본값을 제공
- `orElseGet(Supplier<? extends T> other)`: orElse 메서드에 대응하는 게으른 버전의 메서드
  - 값이 있으면 해당 값을 반환
  - 값이 없으면 Supplier가 실행
  - 디폴트 메서드를 만드는 데 시간이 걸리거나 Optional이 비어있을 때만 기본값을 생성(기본값이 무조건 필요한 상황)하고 싶을 때 사용
- `orElseThrow(Supplier<? extends X> exceptionSupplier)`: Optional이 비어있을 때 원하는 예외를 발생시는 메서드
- `ifPresent(Consumer<? super T> consumer)`: 값이 존재할 때 인수로 넘겨준 동작을 실행하고 값이 없으면 아무 일도 일어나지 않는 메서드
- `ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)`: Optional이 비어있을 때 실행할 수 있는 Runnable을 인수로 받는다는 점이 `ifPresent()`와 다름
  - Java 9에 추가된 메서드

## filter로 특정 값 거르기

- `public Optional<T> filter(Predicate<? super T> predicate)`: Predicate를 인수로 받아 조건과 일치하면 그 값을 반환하고 그렇지 않으면 빈 Optional을 반환
  - filet를 적용한 Optional에 값이 없으면 아무런 일도 일어나지 않는다
  - Predicate 결과가 true이면 아무 변화도 일어나지 않고 그대로 값을 반환
  - Predicate 결과가 false이면 해당 값은 사라지고 빈 Optional이 반환

```java
Optional<Insurance> optInsurance = Optional.of(insurance);

// AIA 보험인지 확인하기
optInsurance.filter(insurance -> "AIA".equals(insurance.getName()))
	.ifPresent(x -> System.out.println("AIA 보험 입니다."));

// minAge 이상 나이를 가진 사람의 보험 회사 알아오기
Optional<Person> person = Optional.of(personA);
person.filter(p -> p.getAge() >= minAge)
	.flatMap(Person::getCar)
	.flatMap(Car::getInsurance)
	.map(Insurance::getName)
	.orElse("Unknown");
```

<center><img src="/assets/images/modernjavainaction/ch11/8.png"></center>

<br>

# 📌 Optional 실용 예제

Optional 클래스를 효과적으로 이용하려면 잠재적으로 존재하지 않는 값의 처리 방법을 바꿔야 한다. 즉, 코드 구현만 바꾸는 것이 아니라 Native Java API와 상호작용하는 방식도 바꿔야 한다

기존 Java API는 Optional을 적절히 활용하지 않고 있어 Optional을 사용할 수 없다 생각할 수 있는데, 이 부분을 작은 유틸리티 메서드를 추가하는 방식으로 해결할 수 있다.

## 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기

기존 Java API는 null을 반환해 요청한 값이 없거나 문제가 발생해 계산을 실패했음을 알린다. 자료구조 Map의 get(K key) 메서드가 대표적인 예시이다.

```java
Map<String, Object> map = new HashMap<>();

Object value = map.get("key");
```

map의 get 메서드는 해당 key의 값을 가져오는데 만약 key에 대응하는 값이 없으면 null이 반환된다. 이처럼 null이 발생할 수 있는 대상은 Optional로 감싸는 것이 좋다. 기존처럼 if-then-else 같은 조건 분기문을 사용할 수 있지만 Optional을 사용하면 깔끔하게 해결할 수 있다

```java
Optional<Object> value = Optional.ofNullable(map.get("key"));
```

## 예외와 Optional 클래스

Java API는 null을 반환하는 대신 예외를 발생시킬 때도 있다. 대표적으로 `Integer.parseInt(String s)`에서 문자열을 숫자로 변환할 수 없으면 `NumberFormatException`을 발생시킨다.

이를 작은 유틸리티 메서드를 구현해서 Optional을 반환할 수 있도록 만들 수 있다

```java
/**
 * 문자열을 정수로 반환하는 유틸리티 메서드
 * 변환 가능할 때: Integer.parseInt()로 변환해 반환
 * 변환 불가능할 때: 빈 Optional 객체를 반환
 */
public static Optional<Integer> stringToInt(String s) {
	try {
		return Optional.of(Integer.parseInt(s));
	} catch (NumberFormatException e) {
		return Optional.empty();
	}
}
```

## 기본형 특화 Optional은 사용하지 말자

스트림처럼 Optional도 기본형 특화 Optional들이 존재한다.

- `OptionalInt`, `OptionalLong`, `OptionalDouble` 등

스트림은 많은 요소를 가질 때는 기본형 특화 스트림을 이용해서 성능 향상을 일으킬 수 있다. 하지만 Optional은 최대 요소 수는 한 개이므로 기본형 특화 클래스로 성능을 개선할 수 없다.

또 기본형 특화 Optional은 일반 Optional 에서 제공하는 `map()`, `flatMap()`, `filter()`등을 지원하지 않는다. 스트림과 마찬가지로 일반 Optional과 혼용할 수 없어 기본형 특화 Optional은 권장되지 않는다.

<br>

# 📌 Summary

- 프로그래밍 언어는 null 참조로 값이 없는 상황을 표현해왔다
- Java 8에서는 값의 유무를 명확히 표현하는 선택형 반환 값을 지원하는 `java.util.Optional<T>`를 제공
- 정적 팩토리 메서드인 `Optional.empty`, `Optional.of`, `Optional.ofNullable` 등을 이용해 객체를 생성할 수 있다
- Optional 클래스는 스트림과 비슷한 연산을 수행하는 map, flatMap, filter 등의 메서드를 지원한다
- Optional로 값이 없는 상황을 적절하게 처리하도록 강제할 수 있다
  - Optional로 예상치 못한 예외를 방지할 수 있다
- Optional을 활용하면 메서드의 시그니처만으로 Optional값이 사용되거나 반환되는지 예측할 수 있는 좋은 API를 설계할 수 있다