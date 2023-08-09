---
title: "[모던 자바 인 액션] 09. 리팩토링, 테스팅, 디버깅"
categories: [Modern Java in Action]
tag: ["Java"]
author_profile: false
sidebar:
    nav: "counts"
---

>  Reference
>
> [모던 자바 인 액션: 람다, 스트림, 함수형, 리액티브 프로그래밍으로 새로워진 자바 마스터하기](https://www.yes24.com/Product/Goods/77125987?pid=123487&cosemkid=go15646485055614872&gclid=Cj0KCQjw2qKmBhCfARIsAFy8buL-QGAkh5m5YK8mj5vAgRgCKdSRIHzDDWdAYOLPFSuQNlzVVcFFdbYaAsh1EALw_wcB)

<br>

# 📌 리팩토링: 가독성과 유연성 개선

람다 표현식은 익명 클래스보다 코드를 좀 더 간결하게 만든다. 또 인수로 전달하려는 메서드가 있다면 메서드 참조를 이용하는 것이 람다보다 더 간결한 코드를 구현할 수 있다.

그뿐만 아니라 람다 표현식을 이용한 코드는 다양한 요구사항 변화에 대응할 수 있도록 동작을 파라미터화한다.

이 내용을 바탕으로 더 가독성이 좋고 유연한 코드로 리팩토링 하는 방법을 알아본다

## 코드 가독성 개선

코드 가독성이 좋다는 말은 매우 추상적인 의미이다. 일반적으로 코드 가독성이 좋다는 것은 ‘어떤 코드를 다른 사람도 쉽게 이해할 수 있음’을 의미한다. 코드 가독성을 높이려면 문서화를 잘하고, 코딩 컨벤션을 준수하는 노력을 기울여야 한다.

또 Java 8의 새로운 기능인 스트림 API와 람다 표현식을 이용해 코드의 가독성을 높일 수 있다. 코드를 간결하고 이해하기 쉽게 만들고, 메서드 참조와 스트림 API를 이용해 코드의 의도를 명확하게 드러내야 한다.

## 익명 클래스를 람다 표현식으로 리팩토링

- 하나의 추상 메서드를 구현하는 익명 클래스는 람다 표현식으로 리팩토링 가능하다

익명 클래스는 코드를 장황하게 만들고 쉽게 에러를 일으킨다. 이를 람다 표현식으로 리팩토링 하면 간결하고, 가독성이 좋은 코드를 구현할 수 있다

```java
// Before: 익명 클래스
Runnable r1 = new Runnable() {
	public void run() {
		System.out.println("Hello World");
	}
};

// Refactor: 람다 표현식 사용
Runnable r2 = () -> System.out.println("Hello World");
```

하지만, **모든 익명 클래스를 람다 표현식으로 변환할 수 있는 것은 아니다**

- 익명 클래스에서 사용한 `this`, `super`는 람다 표현식에서 다른 의미를 갖는다

  - 익명 클래스에서 `this`: 익명 클래스 자신을 가리킨다
  - 익명 클래스에서 `super`: 익명 클래스가 상속받은 클래스(부모 클래스)의 멤버를 가리킨다
  - 람다 표현식에서의 `this`: 람다를 감싸는 클래스를 가리킨다
  - 람다 표현식에서의 `super`: 람다를 감싸는 클래스의 부모 클래스의 멤버를 가리킨다

- 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다(shadow variable). 하지만, 람다 표현식으로는 변수를 가릴 수 없다

  ```java
  int a = 10; // 전역 변수
  
  Runnable r1 = new Runnable() {
  	public void run() {
  		int a = 2; // 지역 변수 잘 작동함 -> shadow variable
  		System.out.println(a); // 2
  	}
  };
  
  Runnable r2 = () -> {
  	int a = 2; // 컴파일 에러
  	System.out.println(a); // 10
  };
  ```

- 익명 클래스를 람다 표현식으로 바꾸면 Context Overloading에 따른 모호함이 초래될 수 있다

  - 익명 클래스는 인스턴스화할 때 명시적으로 형식이 정해진다
  - 람다의 형식은 Context에 따라 달라진다

  ```java
  interface Task {
  	public void execute();
  }
  
  public static void doSomething(Runnable r) { r.run(); }
  public static void doSomething(Task t) { t.execute(); }
  
  // Task를 구현하는 익명 클래스를 전달
  doSomething(new Task() {
  	public void execute() {
  		System.out.println("Danger danger");
  	}
  });
  
  // 람다 표현식으로 바꾸면 메서드 호출 시 Runnable, Task 모두 대상 형식이 될 수 있음
  // 즉, 모호함이 발생
  doSomething(() -> System.out.println("Danger danger"));
  
  // 위 문제를 명시적 형변환을 이용해 제거할 수 있음
  doSomething((Task)() -> System.out.println("Danger danger"));
  ```

> 인텔리제이, 이클립스 같은 IDE들은 좋은 리팩토링 기능을 제공해 이와 같은 문제를 자동으로 해결해준다.

## 람다 표현식을 메서드 참조로 리팩토링

람다 표현식은 쉽게 전달할 수 있는 짧은 코드이지만 메서드 참조를 이용하면 가독성을 높일 수 있다.

메서드 참조의 메서드명으로 코드의 의도를 명확하게 드러낼 수 있기 때문이다

```java
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream()
		.collect(
			groupingBy(dish -> {
				if (dish.getCalories() <= 400) {
					return CaloricLevel.DIET;
				} else if (dish.getCalories() <= 700) {
					return CaloricLevel.NORMAL;
				} else
					return CaloricLevel.FAT;
			})
		);

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel2 = menu.stream()
		.collect(groupingBy(Dish::getCaloricLevel));
```

람다 표현식을 별도의 메서드로 추출한 다음 메서드 참조를 사용하면 코드가 간결하고 의도가 명확해진다.

또 `comparing()`, `maxBy()` 같은 정적 헬퍼 메서드를 활용하는 것도 좋다. 이들은 메서드 참조와 조화를 이루도록 설계되어있는데, 람다 표현식보다 더 명확한 의도를 드러낼 수 있다.

```java
// 람다 표현식
inventory.sort(
	(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

// 메서드 참조: 코드가 문제 자체를 설명해줌
inventory.sort(comparing(Apple::getWeight));
```

`sum`, `maximum`등 자주 사용하는 리듀싱 연산은 메서드 참조와 함께 사용할 수 있는 내장 헬퍼 메서드를 제공한다

```java
int totalCalories = menu.stream()
	.map(Dish::getCalories)
	.reduce(0, (c1, c2) -> c1 + c2);

// 내장 컬렉터 이용
int totalCalories = menu.stream()
	.collect(summingInt(Dish::getCalories));
```

## 명령형 데이터 처리를 스트림으로 리팩토링

이론적으로 반복자를 이용한 기존 모든 컬렉션 처리 코드는 스트림 API로 바꿔야 한다. 스트림 API는 데이터 처리 파이프라인의 의도를 더 명확하게 보여주고, 강력한 최적화와 멀티코어 아키텍처를 활용할 수 있는 지름길을 제공하기 때문이다.

```java
List<String> dishNames = new ArrayList<>);
for (Dish dish : menu) {
	if (menu.getCalories() > 300) {
		dishNames.add(dish.getName());
	}
}

// 스트림 API 이용
List<String> dishNames = menu.parallelStream()
		.filter(d -> d.getCalories() > 300)
		.map(Dish::getName)
		.collect(toList());
```

명령형 코드의 break, continue, return 등의 제어 흐름문을 모두 분석해 같은 기능을 수행하는 스트림 연산으로 유추해야 하므로 명령형 코드를 스트림 API로 리팩토링 하는 것은 어렵다

## 코드 유연성 개선

람다를 이용하면 동작 파라미터화(behaviour parameterization)를 쉽게 구현할 수 있다. 즉, 다양한 람다를 전달해 다양한 동작을 표현할 수 있다. 예를 들어 Predicate로 다양한 필터링 기능을 구현하거나, 비교자로 다양한 비교 기능을 구현할 수 있다.

### 함수형 인터페이스 적용

람다 표현식을 사용하려면 함수형 인터페이스가 필요하다. 자주 사용하는 두 가지 패턴으로 람다 표현식의 리팩토링을 살펴본다

- 조건부 연기 실행(conditional deferred execution)
- 실행 어라운드(execute around)

### 조건부 연기 실행(conditional deferred execution)

실제 작업을 처리하는 코드 내부에 제어 흐름문이 복잡하게 얽힌 코드가 있다고 가정한다. 보안 검사나 로깅 코드가 주로 이처럼 사용된다.

```java
if (logger.isLoggable(Log.FINER)) {
	logger.finer("Problem: " + generateDiagnostic());
}
```

위 코드의 문제점은 객체의 상태가 클라이언트 코드로 노출되고, 매번 객체의 상태를 확인해야 한다는 점이다.

다음처럼 내부적으로 확인하는 메서드를 사용하는 것이 바람직할 것이다.

```java
logger.log(Level.FINER, "Problem: " + generateDiagnostic());
```

덕분에 불필요한 if 문을 제거하고 객체의 상태를 노출할 필요도 없어졌지만, 인수로 전달된 메시지 수준에서 logger가 활성화되어 있지 않아도 항상 메시지를 평가하게 된다.

람다를 이용하면 위 문제들을 모두 깔끔하게 해결할 수 있다. 즉, 특정 조건에서만 메시지가 생성될 수 있도록 메시지 생성 과정을 연기(defer)할 수 있어야 하는데, Java 8은 `Supplier`를 인수로 갖는 오버로드된 log메서드를 제공한다.

```java
public void log(Level level, Supplier<String> msgSupploer)

// log 메서드 호출
logger.log(Level.FINER, () -> "Problem: " + generateDiagnostic());
```

조건부 연기 실행으로 해결할 수 있는 문제는 다음과 같다

- 클라이언트 코드에서 객체 상태를 자주 확인하는 코드
- 객체의 일부 메서드를 호출하는 상황에서 람다나 메서드 참조를 통해 객체의 상태를 확인한 다음 메서드를 호출하도록 변경

### 실행 어라운드(execute around)

만약 반복적으로 수행하는 코드가 존재하면 이를 람다로 변환할 수 있다.

파일을 열고 닫을 때 같은 로직을 사용하는 코드를 람다를 이용해서 다양한 방식으로 파일을 처리할 수 있도록 파라미터화하는 예제이다.

```java
interface BufferedReaderProcessor {
	String process(BufferedReader b) throws IOException;
}

public static String processFile(BufferedReaderProcessor p) throws IOException {
	try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))
	) {
		return p.process(br); // BufferedReader 객체 처리
	}
}

// 람다 전달
String oneLine = processFile((BufferedReader br) -> br.readLine());
// 다른 람다 전달
String twoLine = processFile((BufferedReader br) -> br.readLine() + br.readLine());

System.out.println("Read one line: " + oneLine);
System.out.println("Read two line: " + twoLine);
```

람다로 BufferedReader 객체의 동작을 결정할 수 있는 것은 함수형 인터페이스인 BufferedReaderProcessor 덕분이다.

<br>

# 📌 람다로 객체지향 디자인 패턴 리팩토링

람다를 이용하면 이전에 디자인 패턴으로 해결하던 문제를 더 쉽고 간단하게 해결할 수 있다. 또 기존의 많은 객체지향 디자인 패턴을 제거하거나 간결하게 재구현할 수 있다

## 전략(Strategy) 패턴

**전략 패턴**은 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법이다.

<center><img src="/assets/images/modernjavainaction/ch09/1.png"></center>

- 알고리즘을 나타내는 인터페이스: `Strategy` 인터페이스
- 다양한 알고리즘을 한 개 이상의 인터페이스 구현(ConcreteStrategyA 같은 구체 클래스)
- 전략 객체를 사용하는 한 개 이상의 클라이언트

```java
/**
 * 알고리즘을 나타내는 전략 인터페이스
 */
interface ValidationStrategy {

	boolean execute(String s);

}

// 구현체 1: 모두 소문자인지 검사
class IsAllLowerCase implements ValidationStrategy {

	@Override
	public boolean execute(String s) {
		return s.matches("[a-z]+");
	}

}

// 구현체 2: 숫자인지 검사
class IsNumeric implements ValidationStrategy {

	@Override
	public boolean execute(String s) {
		return s.matches("\\d+");
	}

}

class Validator {

	private final ValidationStrategy strategy;

	Validator(ValidationStrategy strategy) {
		this.strategy = strategy;
	}

	public boolean validate(String s) {
		return strategy.execute(s);
	}

}
```

위 전략 패턴을 일반적으로 사용하면 다음과 같이 사용할 수 있다

```java
String s = "aaaa";

Validator numericValidator = new Validator(new IsNumeric());
if (numericValidator.validate(s)) {
	System.out.println("숫자 입니다.");
	} else {
	System.out.println("숫자가 아닙니다.");
}

Validator lowerCaseValidator = new Validator(new IsAllLowerCase());
if (lowerCaseValidator.validate(s)) {
	System.out.println("모두 소문자 입니다.");
} else {
	System.out.println("모두 소문자가 아닙니다.");
}
```

여기서 람다 표현식을 사용할 수 있다. `ValidationStrategy`인터페이스는 함수형 인터페이스로 `Predicate` 같은 함수 디스크립터를 갖고 있음을 파악할 수 있다. 따라서 다양한 전략을 구현하는 새로운 클래스를 구현할 필요 없이 람다 표현식을 직접 전달하면 코드가 간결해진다

```java
Validator numericValidator = 
	new Validator((String s) -> s.matches("\\d+"));

Validator lowerCaseValidator = 
	new Validator((String s) -> s.matches("[a-z]+"));
```

람다를 이용하면 자잘한 코드를 제거할 수 있다. 람다 표현식은 코드 조각(전략)을 캡슐화한다. 즉, 람다 표현식으로 디자인 패턴을 대신할 수 있다는 의미다.

## 템플릿 메서드(Template Method) 패턴

**템플릿 메서드 패턴**은 알고리즘의 개요를 제시한 다음 알고리즘의 일부를 고칠 수 있는 유연함을 제공하는 기법이다. 즉, 어떤 알고리즘을 사용하고 싶은데 그대로는 안 되고 조금 고쳐야 하는 상황에 적합하다.

온라인 뱅킹 애플리케이션을 통해 템플릿 메서드 패턴을 알아본다

```java
abstract class OnlineBanking {

	private static final Map<Integer, Customer> storage = new HashMap<>();

	public void processCustomer(int id) {
		Customer c = storage.get(id);
		makeCustomerHappy(c);
	}

	abstract void makeCustomerHappy(Customer c);

}
```

각각의 은행 지점은 OnelineBanking 추상 클래스를 상속 받아 추상 메서드를 원하는 동작을 수행하도록 구현할 수 있다.

여기서 람다를 사용하면 문제를 쉽게 해결할 수 있다. 람다나 메서드 참조로 알고리즘에 추가할 다양한 컴포넌트를 구현할 수 있다.

위에서 정의한 `makeCustomerHappy()`메서드 시그니처와 일치하도록 `Consumer<Customer>` 형식을 갖는 두 번째 인수를 `processCustomer()` 메서드에 추가한다. 이렇게 하면 추상 클래스를 상속 받지 않고 직접 람다 표현식을 전달해 다양한 동작을 수행할 수 있다

```java
public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
	Customer c = storage.get(id);
	makeCustomerHappy.accept(c);
}

new OnlineBanking().processCustomer(1, (Customer c) -> System.out.println(c));
```

## 옵저버(Observer) 패턴

**옵저버 패턴**은 어떤 이벤트가 발생했을 때 한 객체(주제라 불리는 subject)가 다른 객체 리스트(observer)에 자동으로 알림을 보내야 하는 상황에 사용하는 기법이다.

GUI 애플리케이션에서 옵저버 패턴이 자주 등장한다. 버튼 같은 GUI 컴포넌트에 옵저버를 설정하면 사용자가 버튼을 클릭하면 옵저버에 알림이 전달되고 정해진 동작이 수행된다.

<center><img src="/assets/images/modernjavainaction/ch09/2.png"></center>

옵저버 패턴을 이용해 커스터마이즈된 알림 시스템을 설계하고 구현할 수 있다. 다양한 매체가 뉴스 트윗을 구도하고 있으며 특정 키워드를 포함하는 트윗이 등록되면 알림을 보내는 기능을 구현해본다.

먼저 다양한 옵저버를 그룹화할 Observer 인터페이스가 필요하다. 이후 다양한 키워드에 따라 다른 동작을 수행하는 여러 옵저버를 정의할 수 있다

```java
interface Observer {

	void notify(String tweet);
}

class NYTimes implements Observer {

	@Override
	public void notify(String tweet) {
		if (tweet != null && tweet.contains("money")) {
			System.out.println("NewYork Times 속보!! " + tweet);
		}
	}

}

class Guardian implements Observer {

	@Override
	public void notify(String tweet) {
		if (tweet != null && tweet.contains("queen")) {
			System.out.println("London News... " + tweet);
		}
	}

}

class LeMonde implements Observer {

	@Override
	public void notify(String tweet) {
		if (tweet != null && tweet.contains("wine")) {
			System.out.println("World's best wine! " + tweet);
		}
	}

}
```

다음은 Subject를 구현해야 한다. Feed는 트윗을 받으면 알림을 보낼 옵저버 리스트를 유지하고 있다.

```java
interface Subject {

	void registerObserver(Observer observer);

	void notifyObservers(String tweet);
}

class Feed implements Subject {

	private final List<Observer> observers = new ArrayList<>();

	@Override
	public void registerObserver(Observer observer) {
		this.observers.add(observer);
	}

	@Override
	public void notifyObservers(String tweet) {
		observers.forEach(o -> o.notify(tweet));
	}

}
```

Feed 객체를 생성해 알림을 받고 싶은 옵저버들을 추가하고 트윗을 보내면 해당 트윗에 맞는 동작을 수행하게 된다.

```java
public class ObserverPattern {

	public static void main(String[] args) {
		Feed feed = new Feed();
		feed.registerObserver(new NYTimes());
		feed.registerObserver(new Guardian());
		feed.registerObserver(new LeMonde());

		String tweet = "The queen said her favorite book is Moder Java in Action!!!";
		feed.notifyObservers(tweet); // Guardian의 notify 메서드가 수행된다
	}

}
```

여기서 람다를 사용하면 옵저버를 명시적으로 인스턴스화하지 않고 람다 표현식을 사용해 직접 전달해 실행할 동작을 지정할 수 있다. 만약 할리우드 뉴스 알림을 추가하고 싶다면 따로 옵저버 클래스를 만들지 않고 바로 람다로 직접 전달할 수 있다.

```java
feed.registerObserver((String tweet) -> {
	if (tweet != null && tweet.contains("Hollywood")) {
		System.out.println("Breaking News in Hollywood!!!" + tweet);
	}
});
```

실행해야할 동작이 비교적 간단하면 람다 표현식을 이용해 불필요한 코드를 제거하는 것이 좋다.

## 의무 체인(Chain of Responsibility) 패턴

**의무 체인 패턴**은 작업 처리 객체의 체인(동작 체인 등)을 만들 때 사용하는 기법이다. 한 객체가 어떤 작업을 처리한 후 다른 객체로 결과를 전달하고, 다른 객체도 해야할 작업을 처리한 다음에 또 다른 객체로 전달하는 방식이다.

일반적으로 다으으로 처리할 객체 정보를 유지하는 필드를 포함하는 작업 처리 추상 클래스로 의무 체인 패턴을 구성한다.

```java
abstract class ProcessingObject<T> {

	protected ProcessingObject<T> successor;

	public void setSuccessor(ProcessingObject<T> successor) {
		this.successor = successor;
	}

	/**
	 * 일부 작업을 어떻게 처리해야 할지 전체적으로 기술
	 */
	public T handle(T input) {
		T t = handleWork(input);
		if (successor != null) {
			return successor.handle(t);
		}
		return t;
	}

	abstract protected T handleWork(T input);

}
```

위 추상 클래스를 상속받아 handleWork 메서드를 구현하면 다양한 종류의 작업 처리 객체를 만들 수 있다

```java
class HeaderTextProcessing extends ProcessingObject<String> {

	@Override
	protected String handleWork(String input) {
		return "From Raoul, Mario and Alan: " + input;)
	}

}

class SpellCheckerProcessing extends ProcessingObject<String> {

	@Override
	protected String handleWork(String input) {
		return input.replaceAll("labda", "lambda");
	}

}

ProcessingObject<String> p1 = new HeaderTextProcessing();
SpellCheckerProcessing p2 = new SpellCheckerProcessing();
p1.setSuccessor(p2); // 두 작업 처리 객체를 연결

String input = "Aren't labdas really cool?";
String result = p1.handle(input);
System.out.println("result = " + result);
// result = From Raoul, Mario and Alan: Aren't lambdas really cool?
```

위 패턴은 함수 체인(함수 조합)과 비슷하다. 람다 표현식을 조합하면 불필요한 코드를 제거할 수 있다.

`Function<String, String>`은 `UnaryOperator<String>` 형식의 인스턴스로 표현할 수 있고, andThen 메서드로 이들 함수를 조합해 체인을 만들 수 있다

```java
// 첫 번째 작업 처리 객체
UnaryOperator<String> headerProcessing = (String text) -> "From Raoul, Mario and Alan: " + text;
// 두 번째 작업 처리 객체
UnaryOperator<String> spellCheckerProcessing = (String text) -> text.replaceAll("labda", "lambda");

Function<String, String> pipeline = headerProcessing.andThen(spellCheckerProcessing);
result = pipeline.apply(input);
System.out.println("result = " + result);
// result = From Raoul, Mario and Alan: Aren't lambdas really cool?
```

## 팩토리(Factory) 패턴

**팩토리 패턴**은 인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 사용하는 기법이다.

```java
class ProductFactory {

	public static Product createProduct(String name) {
		switch (name) {
			case "loan":
				return new Loan();
			case "stock":
				return new Stock();
			default:
				throw new RuntimeException(name + " 상품이 없습니다");
		}
	}

}

Product p = ProductFactory.createProduct("loan");
```

위 코드의 장점은 생성자와 설정을 외부로 노출하지 않음으로써 클라이언트가 단순하게 상품을 생산할 수 있다는 것이다.

여기서 람다를 이용하면 다음과 같다

```java
class ProductFactory {

	static final Map<String, Supplier<Product>> map = new HashMap<>();

	static {
		map.put("loan", Loan::new);
		map.put("stock", Stock::new);
	}

	public static Product createProduct(String name) {
		Supplier<Product> supplier = map.get(name);

		if (supplier != null) {
			return supplier.get();
		}
		throw new IllegalArgumentException(name + " 상품이 없습니다");
	}

}
```

<br>

# 📌 람다 테스팅

좋은 SW 구현을 위해서는 프로그램이 의도대로 동작하는지 확인할 수 있는 **단위 테스트(Unit Test)**가 필요하다.

## 보이는 람다 표현식의 동작 테스팅

```java
import static java.util.Comparator.comparing;

import java.util.Comparator;

public class Point {

	...

	public final static Comparator<Point> compareByXAndThenY =
		comparing(Point::getX).thenComparing(Point::getY);

}

@Test
@DisplayName("람다를 이용한 정적 필드 테스트")
void comparingTwoPoints() {
	Point p1 = new Point(10, 15);
	Point p2 = new Point(10, 20);
	int result = Point.compareByXAndThenY.compare(p1, p2);

	assertThat(result < 0).isTrue();
}
```

## 람다를 사용하는 메서드의 동작에 집중하기

람다의 목표는 정해진 동작을 다른 메서드에서 사용할 수 있도록 하나의 조각으로 캡슐화하는 것이다.

그러러면 세부 구현을 포함하는 표현식을 공개하면 안되는데, 람다 표현식을 사용하는 메서드의 동작을 테스트해 람다를 공개하지 않으면서도 람다 표현식을 검증할 수 있다.

```java
public static List<Point> moveAllPointsRightBy(List<Point> points, int x) {
	return points.stream()
		.map(p -> new Point(p.getX() + x, p.getY()))
		.collect(Collectors.toList());
}

@Test
@DisplayName("람다를 사용하는 메서드의 동작에 집중: 람다를 공개하지 않고 테스트로 검증")
void moveAllPointsRightBy() {
	List<Point> points = Arrays.asList(new Point(5, 5), new Point(10, 5));
	List<Point> expected = Arrays.asList(new Point(15, 5), new Point(20, 5));

	List<Point> actual = Point.moveAllPointsRightBy(points, 10);
	assertThat(actual).containsAll(expected);
}
```

> equals(), hashCode()를 재정의 해 x, y가 둘다 같으면 동일한 객체로 판단

## 복잡한 람다를 개별 메서드로 분할하기

많은 로직을 포함하는 람다 표현식이 있을 때, 테스트 코드에서는 람다 표현식을 참조할 수 없는데 어떻게 람다 표현식을 테스트 할 수 있나? 람다 표현식을 **메서드 참조**로 바꾸면 일반 메서드를 테스트하듯 람다 표현식을 테스트할 수 있다.

## 고차원 함수 테스팅

함수를 인수로 받거나 다른 함수를 반환하는 고차원 함수는 좀 더 사용하기 어렵다. 메서드가 람다를 인수로 받는다면 다른 람다의 메서드의 동작을 테스트할 수 있다.

```java
@Test
@DisplayName("고차원 함수 테스팅")
void higherOrderFunction() {
	List<Integer> numbers = Arrays.asList(1, 2, 3, 4);
	List<Integer> even = Filter.filter(numbers, i -> i % 2 == 0);
	List<Integer> smallerThanThree = Filter.filter(numbers, i -> i < 3);

	assertThat(Arrays.asList(2, 4)).containsAll(even);
	assertThat(Arrays.asList(1, 2)).containsAll(smallerThanThree);
}
```

<br>

# 📌 디버깅(Debugging)

문제가 발생한 코드를 디버깅할 때 두 가지를 가장 먼저 확인해야 한다

- 스택 트레이스
- 로깅

하지만 람다 표현식과 스트림은 기존의 디버깅 기법을 무력화한다

## 람다와 스택 트레이스

예외가 발생해 프로그램이 중단되면 스택 프레임(stack frame)에서 어디에서 멈췄고 어떻게 멈췄는지 정보를 얻을 수 있다. 프로그램의 메서드 호출 위치, 인수 값, 호출된 메서드의 정보등을 포함한 호출 정보가 생성되어 이들 정보는 스택 프레임에 저장된다.

따라서 프로그램이 멈추면 스택 프레임별로 보여주는 스택 트레이스(stack trace)를 얻을 수 있다.

하지만 **람다 표현식은 이름이 없어 조금 복잡한 스택 트레이스가 생성**된다.

<center><img src="/assets/images/modernjavainaction/ch09/3.png"></center>

이와 같은 정보는 람다 표현식 내부에서 에러가 발생했음을 가리킨다. 람다는 이름이 없어 컴파일러가 람다를 참조하는 이름을 만들어내 `lambda$main$0` 같은 이름이 출력된다.

람다와 관련된 스택 트레이스는 이해하기 어렵다는 점을 기억해야 한다

## 정보 로깅

스트림은 최종 연산을 호출하면 전체 스트림이 소비된다. 따라서 스트림 파이프라인에 적용된 각각의 중간 연산을 확인하기 위해서는 peek 연산을 사용해야 한다.

peek 연산은 스트림의 각 요소를 소비한 것 처럼 동작하지만 실제로 스트림의 요소를 소비하지는 않는다.

peek 연산을 중간 연산과 함께 사용하면 스트림의 계산 과정을 로깅할 수 있다.

```java
list.stream()
	.peek(x -> System.out.println(x) // 중간 결과 로깅
	.map(x -> x + 10)
	.peek(x -> System.out.println(x) // 중간 결과 로깅
	.filter(x -> x % 2 == 0)
	.peek(x -> System.out.println(x) // 중간 결과 로깅
	...
	.collect(toList());
```

<br>

# 📌 Summary

- 람다 표현식은 가독성이 좋고 더 유연한 코드를 만들어준다
- 익명 클래스는 람다 표현식으로 바꾸는 것이 좋다. 하지만 이때 this, shadow variable 등 미묘하게 의미상 다른 내용을 주의해야 한다
- 메서드 참조를 이용하면 람다 표현식보다 더 가독성이 좋은 코드를 구현할 수 있다
- 반복적으로 컬렉션을 처리하는 루틴은 스트림 API로 대체할 수 있을지 고려해야 한다
- 람다 표현식으로 디자인 패턴의 불필요한 코드를 제거할 수 있다
- 람다 표현식도 단위 테스트를 수행할 수 있다. 하지만 람다 표현식 자체를 테스트하는 것보다 람다 표현식이 사용되는 메서드의 동작을 테스트하는 것이 바람직하다
- 복잡한 람다 표현식은 일반 메서드로 재구현할 수 있다
- 람다 표현식을 사용하면 스택 트레이스를 이해하기 어려워진다
- 스트림 파이프라인에서 요소를 처리할 때 peek 메서드로 중간 과정을 로깅할 수 있다