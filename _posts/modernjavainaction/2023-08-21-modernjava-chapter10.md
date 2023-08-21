---
title: "[모던 자바 인 액션] 10. 람다를 이용한 도메인 전용 언어(DSL)"
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

프로그래밍 언어도 결국 언어이다. 언어의 주요 목표는 메시지를 명확하고, 안정적인 방식으로 전달하는 것이다.

즉, 프로그래밍 언어도 의도가 명확하게 전달되어야 한다.

애플리케이션의 핵심 비즈니스를 모델링하는 소프트웨어 영역에서 읽기 쉽고, 이해하기 쉬운 코드는 특히 중요하다.

**도메인 전용 언어(Domain-Specific Languages, DSL)**로 애플리케이션의 비즈니스 로직을 표현해 소프트웨어의 버그와 오해를 잡고 비즈니스 관점에서 제대로 되었는지 확인할 수 있다.

DSL은 특정 도메인을 대상으로 만들어진 특수 프로그래밍 언어로 Maven, Ant 등이 있다. 이들은 빌드 과정을 표현하는 DSL로 간주할 수 있다.

# 📌 도메인 전용 언어(Domain-Specific Languages, DSL)

- **DSL**: 특정 비즈니스 도메인의 문제를 해결하려고 만든 언어

예를 들어 회계 전용 SW 애플리케이션을 개발할 때 비즈니스 도메인에는 통장 입출금 내역서, 계좌 통합 같은 개념이 포함된다. 이런 문제를 표현할 수 있는 DSL을 만들 수 있다. Java에서는 도메인을 표현할 수 있는 클래스와 메서드 집합이 필요하고, DSL은 특정 비즈니스 도메인을 인터페이스로 만든 API라 생각할 수 있다.

DSL은 특정 도메인에 국한되므로 자신의 앞에 놓인 비즈니스 문제를 어떻게 해결할지에만 집중할 수 있다. 따라서 DSL을 이용하면 사용자가 특정 도메인의 복잡성을 더 잘 다룰 수 있다.

DSL을 개발할 때는 두 가지 필요성을 생각하면서 개발해야 한다

- **의사 소통**: 코드의 의도가 명확히 전달되어야 하며 프로그래머가 아니더라도 이해할 수 있어야 한다.
- **한 번 코드를 구현하지만 여러 번 읽는다**: 가독성은 유지보수의 핵심, 쉽게 이애할 수 있는 코드를 구현해야 한다.

## DSL의 장점과 단점

**DSL 장점**

- 간결함: API는 반복을 피할 수 있는 코드를 간결하게 만들 수 있다
- 가독성: 다양한 조직 구성원 간 코드와 도메인 영역이 공유될 수 있도록 쉽게 이애할 수 있게 만든다
- 유지보수
- 높은 수준의 추상화
- 집중: 프로그래머가 특정 코드에 집중할 수 있어 생산성이 좋아진다
- 관심사 분리(Separation of Concerns): 인프라구조와 관련된 문제와 독립적으로 비즈니스 관련 코드에 집중하기가 용이하다

**DSL 단점**

- 설계의 어려움
- 높은 개발 비용
- 추가 우회 계층
- 새로 배워야 하는 언어
- 호스팅 언어의 한계

## JVM에서 이용할 수 있는 다른 DSL 해결책

DSL의 카테고리를 구분하는 가장 흔한 방법은 Martin Fowler가 소개한 방법으로 내부 DSL과 외부 DSL을 나누는 것이다. 내부 DSL(임베디드 DSL)은 순수 Java 코드 같은 기존 호스팅 언어를 기반으로 구현하고, 외부 DSL은 호스팅 언어와는 독립적으로 자체의 문법을 가진다

JVM으로 인해 내부 DSL과 외부 DSL의 중간 카테고리에 해당하는 DSL이 만들어질 가능성이 생겼다. 그루비(Groovy)처럼 JVM에서 실행되며 더 유연하고 표현력이 강력한 언어가 있는데 이들을 다중 DSL이라 칭한다.

### 내부 DSL

Java 8의 람다가 등장하면서 어느정도 읽기 쉽고, 간단하고, 표현력 있는 DSL을 만들 수 있게 됐다.

문자열 목록을 출력하는 상황을 Java 8의 새 forEach 메서드를 이용하는 예제로 살펴본다

```java
// Java 7 문법: 익명 내부 클래스 사용
List<String> numbers = Arrays.asList("one", "two", "three");
numbers.forEach(new Consumer<String>() { // 코드의 잡음
	@Override
	public void accept(String s) { // 코드의 잡음
		System.out.println(s); // 코드의 잡음
	}
});
```

Java 8에서는 위 예제에서 발생하는 코드의 잡음이 많이 줄어든다

```java
// 람다 표현식
numbers.forEach(s -> System.out.println(s));

// 메서드 참조
numbers.forEach(System.out::println);
```

이렇게 Java로 DSL을 구현하면 다음과 같은 장점을 얻을 수 있다.

- 새로운 패턴과 기술을 배우는 비용을 줄인다
- 나머지 코드와 함꼐 DSL을 컴파일할 수 있어 다른 컴파일러를 이용하거나 외부 도구를 사용할 추가 비용이 들지 않는다
- 추가로 개발된 DSL을 쉽게 합칠 수 있다

### 다중 DSL

같은 Java Byte Code를 사용하는 JVM 기반 언어를 이용하면 DSL 병합 문제를 해결할 수 있다. 요즘 JVM에서 실행되는 언어는 100개가 넘고, 스칼라와 그루비, 코틀린 같은 언어도 등장해 호환성을 유지하며 쉽게 배울 수 있는 강점을 가진다. 이들은 Java보다 제약이 적고, 간편한 문법을 지향하도록 설계되었다.

위 언어들을 사용하면 결과적으로 문법적 잡음이 없으며 개발자가 아닌 사람도 코드를 쉽게 이해할 수 있다. 더 친화적인 DSL을 만들 수 있지만 다음과 같은 불편함도 초래한다.

- 새로운 언어를 배우는데 드는 비용, 결국 고수준 구현을 위해서는 깊은 지식이 필요
- 두 개 이상의 언어가 혼재해 여러 컴파일러로 소스를 빌드하도록 빌드 과정 개선이 필요
- JVM에서 실행되지만 Java와 100% 완벽하게 호환되지 않을 때가 많음

### 외부 DSL

외부 DSL은 무한한 유연성을 제공하지만 자신만의 문법과 구문으로 새 언어를 설계한다는 어려움이 있다. 새 언어를 파싱하고, 파서의 결과를 분석해 외부 DSL을 실행할 코드를 만들어야 하는데, 이는 아주 큰 작업이다.

Java 기반 파서 생성기를 이용하면 도움이 되지만 논리 정연한 프로그래밍 언어를 새로 개발한다는 것은 매우 어려운 일이다. 또 외부 DSL은 쉽게 제어 범위를 벗어날 수 있어 처음 설계한 목적과 크게 벗어나는 경우도 많다.

<br>

# 📌 최신 Java API의 작은 DSL

Java의 새로운 기능의 장점을 적용한 첫 API는 Native Java API이다. Java 8 이전의 Native Java API는 한 개의 추상 메서드를 가진 인터페이스를 갖고 있었다. 하지만 내부 클래스를 구현하려면 불필요한 코드가 추가되어야 했는데, 람다와 메서드 참조가 등장하면서 새로운 판도가 열렸다.

Java 8의 Comparator 인터페이스에 새 메서드가 추가되었는데 이를 통해 어떻게 람다가 Native Java API의 재사용성과 메서드 결합도를 높였는지 확인해본다

```java
// 무명 내부 클래스 이용
Collections.sort(persons, new Comparator<Person>() {
	@Override
	public int compare(Person o1, Person o2) {
		return o1.getAge() - o2.getAge();
	}
});
```

위 무명 내부 클래스를 간단한 람다 표현식으로 바꾸면 다음과 같다. 여기서 Java가 제공하는 정적 유틸리티 메서드와 메서드 참조를 이용하면 더 개선할 수 있다.

```java
// 람다 표현식 이용
Collections.sort(persons, (p1, p2) -> p1.getAge() - p2.getAge());
		
// 정적 유틸리티 메서드와 메서드 참조를 이용해 가독성 높이기
Collections.sort(persons, comparing(Person::getAge));
```

여기서 그치지 않고 Java 8은 reverse 메서드를 사용해 역순(내림차순)으로 정렬할 수 있다

> Java 11에서 메서드 이름의 일관성을 위해 reverse → reversed로 변경되었다.

```java
// reversed 메서드를 이용해 역순 정렬하기
Collections.sort(persons, comparing(Person::getAge).reversed());
```

추가로 같은 나이의 사람들은 이름을 비교해 알파벳 순으로 정렬할 수도 있다

```java
// 추가로 정렬이 가능하다: 나이를 먼저 정렬하고 같은 나이의 사람들을 알파벳 순으로 정렬
Collections.sort(persons, comparing(Person::getAge).thenComparing(Person::getName));
```

마지막으로 List 인터페이스에 추가된 새 sort 메서드를 이용하면 코드가 깔끔하게 정리된다.

```java
// List 인터페이스의 sort 메서드
persons.sort(comparing(Person::getAge).thenComparing(Person::getName));
```

위 변화는 Collections 정렬 도메인의 최소 DSL을 보여준다. 작은 영역에 국한된 예제이지만 람다와 메서드 참조를 이용해 가독성, 재사용성, 결합성을 높일 수 있는지 보여준다

## 스트림 API = 컬렉션을 조작하는 DSL

Stream 인터페이스는 Native Java API에 작은 내부 DSL을 적용한 좋은 예시이다. Stream은 컬렉션의 항목을 필터, 정렬, 변환, 그룹화, 조작하는 작지만 강력한 DSL이다. 즉, Stream 인터페이스는 데이터 리스트를 조작하는 DSL로 간주할 수 있다.

```java
// 반복 형식으로 예제 로그 파일에서 여러 행을 읽는 코드
List<String> errors = new ArrayList<>();
int errorCount = 0;
BufferedReader br = new BufferedReader(new FileReader(FILE_NAME));
String line = br.readLine();

while (errorCount < 40 && line != null) {
	if (line.startsWith("ERROR")) {
		errors.add(line);
		errorCount++;
	}
	line = br.readLine();
}
```

위 코드는 장황해 의도를 한 눈에 파악하기 어렵다. 문제가 분리되지 않아 가독성과 유지보수성 모두 저하되었고 같은 의무를 지닌 코드가 여러 행에 분산되어 있다.

Stream 인터페이스를 이용해 함수형으로 코드를 구현하면 쉽고 간결하게 코드를 구현할 수 있다

```java
// 스트림 이용하기
errors = Files.lines(Paths.get(FILE_NAME)) // 파일을 열어 문자열 스트림 생성
		.filter(l -> l.startsWith("ERROR")) // ERROR 필터링
		.limit(40) // 결과를 첫 40행으로 제한
		.collect(Collectors.toList()); // 결과 문자열을 리스트로 수집
```

스트림 API의 Fluent Style은 잘 설계된 DSL의 또 다른 특징이다. 모든 중간 연산은 게으르며(lazy) 다른 연산으로 파이프라인될 수 있는 스트림으로 반환된다. 최종 연산은 적극적이며 전체 파이프라인이 계산을 일으킨다.

## Collector 인터페이스 = 데이터를 수집하는 DSL

Collector 인터페이스는 데이터 수집을 수행하는 DSL로 간주할 수 있다. Collector 인터페이스를 이용하면 스트림의 항목을 수집, 그룹화, 파이션할 수 있다. 또 Collectors 클래스에서 제공하는 정적 팩토리 메서드를 이용해 필요한 Collector 객체를 만들고 합치는 기능도 사용할 수 있다.

DSL 관점에서 보면 특히 Comparator 인터페이스는 다중 필드 정렬을 지원하도록 합쳐질 수 있고, Collectors는 다중 수준 그룹화를 달성할 수 있도록 합쳐질 수 있다.

```java
// 자동차를 브랜드별 그리고 색상별로 그룹화하기
Map<String, Map<Color, List<Car>>> carsByBrandAndColor = cars.stream()
		.collect(groupingBy(Car::getBrand, groupingBy(Car::getColor)));

// fluent style로 다중 필드 comparator 정의
Comparator<Car> comparator = comparing(Person::getAge).thenComparing(Car::getColor);

// Collectors를 중첩해 다중 수준 Collector 만들기
Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>> carGroupingCollector = 
		groupingBy(Car::getBrand, groupingBy(Car::getColor));
```

셋 이상의 컴포넌트를 조합할 때는 fluent style이 중첩 형식에 비해 가독성이 좋다. 가장 안쪽의 Collector가 첫 번째로 평가되어야 하지만 논리적으로는 최종 그룹화에 해당하므로 서로 다른 형식은 이를 어떻게 처리하는지를 상반적으로 보여준다.

Natvie Java API를 좀 더 자세히 살펴보고 내부 설계 결정 이유를 생각하면 가독성 있는 DSL 구현 패턴과 기법을 배울 수 있다.

<br>

# 📌 Java로 DSL을 만드는 패턴과 기법

DSL은 특정 도메인 모델에 적용할 친화적이고 가독성 높은 API를 제공한다.

DSL을 만들기 위해 간단한 도메인 모델 세 가지를 먼저 구현한다

- `Stock`: 주어진 시장에 주식 가격을 모델링하는 순수 자바 빈즈
  - symbol과 market을 필드로 갖는다
  - 각 필드의 getter/setter 존재
- `Trade`: 주어진 가격에서 주어진 양의 주식을 사거나 파는 거래
  - type, stock, quantity, price를 필드로 갖는다
  - 각 필드의 getter/setter 존재
  - 가격과 수량을 곱한값을 돌려주는 `getValue()`
- `Order`: 고객이 요청한 한 개 이상의 거래 주문

```java
public class Order {

	private String customer;
	private List<Trade> trades = new ArrayList<>();

	public void addTrade(Trade trade) {
		trades.add(trade);
	}

	public String getCustomer() {
		return customer;
	}

	public void setCustomer(String customer) {
		this.customer = customer;
	}

	public double getValue() {
		return trades.stream()
			.mapToDouble(Trade::getValue)
			.sum();
	}

}
```

먼저 도메인 객체의 API를 직접 이용해 주식 거래 주문을 만들어 본다. 이 코드는 상당히 장황하고 비개발자인 도메인 전문가가 이해하기 어렵고 검증하기 어렵다. 조금 더 직접적이고 직관적인 도메인 모델을 반영할 수 있는 DSL이 필요하다.

```java
/**
 * 도메인 객체의 API를 직접 이용해 주식 거래 주문 생성
 */
public class Version1 {

	public static void main(String[] args) {
		// 주문 고객을 설정
		Order order = new Order();
		order.setCustomer("BigBank");

		// 거래 생성
		Trade trade1 = new Trade();
		trade1.setType(Trade.Type.BUY);

		// 거래할 주식 생성
		Stock stock1 = new Stock();
		stock1.setSymbol("APPLE");
		stock1.setMarket("NYSE");

		// 거래 정보 설정
		trade1.setStock(stock1);
		trade1.setPrice(125.00);
		trade1.setQuantity(100);

		// 주문 설정
		order.addTrade(trade1);
	}

}
```

## 메서드 체인(Method Chain)

DSL에서 가장 흔한 방식 중 하나인 **메서드 체인**이다.

이 방법을 이용하면 한 개의 메서드 호출 체인으로 거래 주문을 정의할 수 있다

```java
// 고객 지정(forCustomer) -> buy or sell -> stock -> on -> at -> ... -> on -> at -> end
Order order = MethodChainingOrderBuilder.forCustomer("BigBank")
			.buy(100) // 거래 1: 매수
			.stock("APPLE")
			.on("NYSE")
			.at(125.00)
			.sell(50) // 거래 2: 매도
			.stock("GOOGLE")
			.on("NASDAQ")
			.at(375.00)
			.end();
```

```java
/**
 * 메서드 체인 DSL을 제공하는 주문 빌더
 */
public class MethodChainingOrderBuilder {

	// 빌더로 감싼 주문
	public final Order order = new Order();

	public static MethodChainingOrderBuilder forCustomer(String customer) {
		return new MethodChainingOrderBuilder(customer);
	}

	// 고객의 주문을 만드는 정적 팩토리 메서드
	private MethodChainingOrderBuilder(String customer) {
		order.setCustomer(customer);
	}

	// 주식을 사는 TradeBuilder 생성
	public TradeBuilder buy(int quantity) {
		return new TradeBuilder(this, Trade.Type.BUY, quantity);
	}

	// 주식을 파는 TradeBuilder 생성
	public TradeBuilder sell(int quantity) {
		return new TradeBuilder(this, Trade.Type.SELL, quantity);
	}

	// 주문에 주식을 추가
	public MethodChainingOrderBuilder addTrade(Trade trade) {
		order.addTrade(trade);
		return this; // 추가 주문을 만들어 추가할 수 있도록 빌더 자체를 반환
	}

	// 주문 생성을 종료하고 반환
	public Order end() {
		return order;
	}

}
```

```java
/**
 * Stock 클래스의 인스턴스를 만드는 거래 빌더
 */
public class TradeBuilder {

	private final MethodChainingOrderBuilder builder;
	public final Trade trade = new Trade();

	public TradeBuilder(MethodChainingOrderBuilder builder, Trade.Type type, int quantity) {
		this.builder = builder;
		trade.setType(type);
		trade.setQuantity(quantity);
	}

	public StockBuilder stock(String symbol) {
		return new StockBuilder(builder, trade, symbol);
	}

}
```

```java
/**
 * 주식을 생성하는 빌더
 */
public class StockBuilder {

	private final MethodChainingOrderBuilder builder;
	private final Trade trade;
	private final Stock stock = new Stock();

	public StockBuilder(MethodChainingOrderBuilder builder, Trade trade, String symbol) {
		this.builder = builder;
		this.trade = trade;
		stock.setSymbol(symbol);
	}

	// 주식의 시장을 지정, 거래에 주식을 추가하고, 최종 빌더를 반환
	public TradeBuilderWithStock on(String market) {
		stock.setMarket(market);
		trade.setStock(stock);
		return new TradeBuilderWithStock(builder, trade);
	}

}
```

```java
public class TradeBuilderWithStock {

	private final MethodChainingOrderBuilder builder;
	private final Trade trade;

	public TradeBuilderWithStock(MethodChainingOrderBuilder builder, Trade trade) {
		this.builder = builder;
		this.trade = trade;
	}

	// 거래되는 주식의 단위 가격을 설정한 다음 원래 주문 빌더를 반환
	public MethodChainingOrderBuilder at(double price) {
		trade.setPrice(price);
		return builder.addTrade(trade);
	}

}
```

이렇게 `MethodChainingOrderBuilder`가 끝날 때까지 다른 거래를 플루언트 방식으로 추가할 수 있다.

빌더 덕분에 사용자는 미리 지정된 절차에 따라 플루언트 API의 메서드를 호출하도록 강제되고, 다음 거래를 설정하기 전 기존 거래를 올바르게 설정하게 된다.

주문에 사용한 파라미터가 빌더 내부에 국한되고, 가독성을 개선하는 효과를 가진다.

하지만 빌더를 구현해야 한다는 것이 메서드 체인의 단점이다. 상위 수준의 빌더를 하위 수준의 빌더와 연결할 많은 접착 코드도 필요하다.

## 중첩된 함수(Nested Function)

**중첩된 함수** DSL 패턴은 다른 함수 안에 함수를 이용해 도메인 모델을 만든다

```java
// 중첩된 함수로 주식 거래 생성
Order order = order("BigBank",
			buy(100, stock("IBM", on("NYSE")), at(125.00)),
			sell(50, stock("GOOGLE", on("NASDAQ")), at(375.00))
		);
```

중첩된 함수 DSL을 제공하는 주문 빌더

```java
/**
 * 중첩된 함수 DSL을 제공하는 주문 빌더
 */
public class NestedFunctionOrderBuilder {

	// 고객과 거래들을 받아 주문 생성
	public static Order order(String customer, Trade... trades) {
		Order order = new Order();
		order.setCustomer(customer);
		Stream.of(trades)
			.forEach(order::addTrade);
		return order;
	}

	// 매수 거래 생성
	public static Trade buy(int quantity, Stock stock, double price) {
		return buildTrade(quantity, stock, price, Trade.Type.BUY);
	}

	// 매도 거래 생성
	public static Trade sell(int quantity, Stock stock, double price) {
		return buildTrade(quantity, stock, price, Trade.Type.SELL);
	}

	// 거래 생성 빌더
	private static Trade buildTrade(int quantity, Stock stock, double price, Trade.Type type) {
		Trade trade = new Trade();
		trade.setQuantity(quantity);
		trade.setStock(stock);
		trade.setPrice(price);
		trade.setType(type);
		return trade;
	}

	// 거래된 주식의 단갈ㄹ 정의하는 더미 메서드
	public static double at(double price) {
		return price;
	}

	// 거래된 주식을 생성
	public static Stock stock(String symbol, String market) {
		Stock stock = new Stock();
		stock.setSymbol(symbol);
		stock.setMarket(market);
		return stock;
	}

	// 주식이 거래된 시장을 정의하는 더미 메서드
	public static String on(String market) {
		return market;
	}

}
```

중첩된 함수 DSL은 메서드 체인에 비해 **도메인 객체 계층 구조**에 그대로 반영된다는 것이 장점이다

하지만 결과 DSL에 더 많은 괄호를 사용해야 하고, 인수 목록을 정적 메서드에 넘겨줘야 한다는 제약이 있다. 또 객체에 선택 사항 필드가 있으면(`final` 키워드가 없는 필드), 해당 인수를 생략할 수 있어 이 가능성을 처리할 수 있도록 여러 메서드 오버로딩을 구현해야 한다.

## 람다 표현식을 이용한 함수 시퀀싱

람다 표현식으로 정의한 함수 시퀀스를 DSL로 사용할 수 있다

```java
// 함수 시퀀싱으로 주식 거래 생성
Order order = order(o -> {
			o.forCustomer("BigBank");
			o.buy(t -> {
				t.quantity(80);
				t.price(125.00);
				t.stock(s -> {
					s.symbol("IBM");
					s.market("NYSE");
				});
			});
			o.sell(t -> {
				t.quantity(100);
				t.price(375.00);
				t.stock(s -> {
					s.symbol("GOOGLE");
					s.market("NASDAQ");
				});
			});
		});
```

람다 표현식을 받아 실행해 도메인 모델을 만들어 내는 여러 빌더를 구현해야 하므로 메서드 체인 방식과 비슷하다.

하지만 `Consumer` 객체를 빌더가 인수로 받아 DSL 사용자가 람다 표현식으로 인수를 구현할 수 있다

```java
/**
 * 함수 시퀀싱 DSL을 제공하는 주문 빌더
 * Consumer 객체를 빌더가 인수로 받음
 */
public class LambdaOrderBuilder {

	private Order order = new Order();

	public static Order order(Consumer<LambdaOrderBuilder> consumer) {
		LambdaOrderBuilder builder = new LambdaOrderBuilder();
		consumer.accept(builder); // 주문 빌더로 전달된 람다 표현식을 실행
		return builder.order; // Consumer를 실행해 만들어진 주문을 반환
	}

	public void forCustomer(String customer) {
		order.setCustomer(customer);
	}

	public void buy(Consumer<TradeBuilder> consumer) {
		trade(consumer, Trade.Type.BUY);
	}

	public void sell(Consumer<TradeBuilder> consumer) {
		trade(consumer, Trade.Type.SELL);
	}

	private void trade(Consumer<TradeBuilder> consumer, Trade.Type type) {
		TradeBuilder tradeBuilder = new TradeBuilder();
		tradeBuilder.trade.setType(type);
		consumer.accept(tradeBuilder); // TradeBuilder로 전달할 람다 표현식 실행
		order.addTrade(tradeBuilder.trade);
	}

}
```

```java
public class TradeBuilder {

	public Trade trade = new Trade();

	public void quantity(int quantity) {
		trade.setQuantity(quantity);
	}

	public void price(double price) {
		trade.setPrice(price);
	}

	public void stock(Consumer<StockBuilder> consumer) {
		StockBuilder builder = new StockBuilder();
		consumer.accept(builder);
		trade.setStock(builder.stock);
	}

}
```

```java
public class StockBuilder {

	public Stock stock = new Stock();

	public void symbol(String symbol) {
		stock.setSymbol(symbol);
	}

	public void market(String market) {
		stock.setMarket(market);
	}

}
```

이 패턴은 메서드 체인 패턴과 중첩된 함수 패턴의 두 가지 장점을 더한다. 플루언트 방식으로 도메인을 정의할 수 있고, 다양한 람다 표현식의 중첩 수준과 비슷하게 도메인 객체의 계층 구조를 유지한다.

하지만 많은 설정 코드가 필요하고, DSL 자체가 람다 표현식 문법에 의한 잡음의 영향을 받는다.

어떤 DSL 패턴을 사용할지는 기호에 달렸고, 꼭 1가지 패턴만을 사용하는 규칙은 없으니 각 패턴을 조합해서 사용해도 된다.

<br>

# 📌 Java 8 DSL

## 메서드 체인의 장점과 단점

### 장점

- 메서드 이름이 키워드 인수 역할을 한다
- 선택형 파라미터와 잘 동작
- DSL 사용자가 정해진 순서로 메서드를 호출하도록 강제할 수 있음
- 정적 메서드를 최소화하거나 없앨 수 있음
- 문법적 잡음을 최소화

### 단점

- 구현이 장황함
- 빌드를 연결하는 접착 코드가 필요
- 들여쓰기 규칙으로만 도메인 객체 계층을 정의

## 중첩 함수의 장점과 단점

### 장점

- 구현의 장황함을 줄일 수 있음
- 함수 중첩으로 도메인 객체 계층을 반영

### 단점

- 정적 메서드의 사용이 빈번
- 이름이 아닌 위치로 인수를 정의
- 선택형 파라미터를 처리할 메서드 오버로딩이 필요

## 람다를 이용한 함수 시퀀싱

### 장점

- 선택형 파라미터와 잘 동작
- 정적 메서드를 최소화하거나 없앨 수 있음
- 람다 중첩으로 도메인 객체 계층을 반영
- 빌더의 접착 코드가 없음

### 단점

- 구현이 장황함
- 람다 표현식 자체의 문법적 잡음이 DSL에 존재

## 스프링 통합(Spring Integration)

**스프링 통합**은 유명한 엔터프라이즈 통합 패턴을 지원할 수 있도록 의존성 주입에 기반한 스프링 프로그래밍 모델을 확장한다. 스프링 통합의 핵심 목표는 복잡한 엔터프라이즈 통합 솔루션을 구현하는 단순한 모델을 제공하고 비동기, 메시지 주도 아키텍처를 쉽게 적용할 수 있게 돕는 것이다.

스프링 통합은 스프링 기반 애플리케이션 내의 경량의 원격, 메시징, 스케쥴링을 지원한다

스프링 통합 DSL에서 가장 널리 사용하는 패턴은 메서드 체인이다. 하지만 최상위 수준의 객체를 만들 때, 그리고 내부의 복잡한 메서드 인수에는 함수 시퀀싱과 람다 표현식을 사용한다.

<br>

# 📌 Summary

- DSL의 주요 기능은 개발자와 도메인 전문가(비개발자) 사이의 간격을 좁히는 것이다
- DSL은 크게 내부적(DSL이 사용될 애플리케이션을 개발한 언어를 그대로 활용) DSL과 외부적(직접 언어를 설계) DSL로 분류할 수 있다
- JVM 기반 언어를 사용하면 다른 언어로 다중 DSL을 개발할 수 있다
- Java 8의 람다 표현식과 메서드 참조 덕분에 DSL을 개발하는 언어로 사용할 수 있게 개선 되었다. 대표적인 Java DSL은 Stream과 Collectors 클래스가 존재한다
- Java로 DSL을 구현할 때 보통 메서드 체인, 중첩 함수, 함수 시퀀싱 세 가지 패턴이 사용된다