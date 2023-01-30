---
title: "[모던 자바 인 액션] 05. 스트림 활용"
categories: [Modern Java in Action]
tag: ["Java"]
author_profile: false
sidebar:
    nav: "counts"
---

<br>

```java
// 외부 반복: 명시적 반복
List<Dish> vegetarianDishesByExternal = new ArrayList<>();
for (Dish dish : menu) {
    if (dish.isVegetarian()) {
        vegetarianDishesByExternal.add(dish);
    }
}

// 내부 반복: filter와 collect 연산을 이용해 데이터 컬렉션 반복을 내부적으로 처리
List<Dish> vegetarianDishesByInternal = menu.stream()
                .filter(Dish::isVegetarian)
                .collect(Collectors.toList());
```

**내부 반복**은 데이터를 어떻게 처리할지는 스트림 API가 관리하므로 편리하게 데이터 관련 작업을 할 수 있다. 따라서 스트림 내에서 다양한 최적화가 이루어지고, 병렬 실행 여부도 결정할 수 있어 외부 반복으로 달성할 수 없는 구현을 할 수 있다.

# 5.1 필터링

## 5.1.1 프레디케이트로 필터링

- `filter(*Predicate*<? *super* T> predicate)`: 프레디케이트를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환
  - Predicate: 불리언을 반환하는 함수

![스트림 필터링](/assets/images/modernjavainaction/ch05/StreamFilter.png "스트림 필터링")

## 5.1.2 고유 요소 필터링

- `distinct()`: 고유 요소로 이루어진 스트림을 반환(중복 제거)
  - 고유 여부는 스트림에서 만든 객체의 `hasCode`, `equals`로 결정

```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
        .filter(i -> i % 2 == 0) // 짝수 필터링
        .distinct() // 고유 요소만 반환(중복 제거)
        .forEach(System.out::println); // 2, 4
```

<br>

# 5.2 스트림 슬라이싱(자바 9)

**스트림 슬라이싱**은 스트림의 **요소를 선택하거나 스킵**하는 다양한 방법을 제공한다. 프레디케이트를 이용하거나, 스트림의 처음 몇 개의 요소를 무시하고, 특정 크기로 스트림을 줄이는 등 다양한 방법을 제공한다.

## 5.2.1 프레디케이트를 이용한 슬라이싱

자바 9은 스트림의 요소를 효과적으로 선택할 수 있도록 `takeWhile`, `dropWhile` 두 가지 새로운 메서드를 지원한다.

**데이터 소스로 사용되는 컬렉션 데이터가 이미 정렬되어 있는 상태**라면, 프레디케이트와 부합하지 않는 요소를 발견하면 반복 작업을 중단해야 효율적이다. 반복 작업 중단을 takeWhile을 통해 지정할 수 있다.

- `takeWhile()`: 무한 스트림을 포함한 모든 스트림에 프레디케이트를 적용해 true인 스트림 요소를 반환

```java
// 정렬된 데이터 소스
public static final List<Dish> specialMenu = Arrays.asList(
    new Dish("season fruit", true, 120, Type.OTHER),
    new Dish("prawns", false, 300, Type.FISH),
    new Dish("rice", true, 350, Type.OTHER),
    new Dish("chicken", false, 400, Type.MEAT),
    new Dish("french fries", true, 540, Type.OTHER)
);

// 320 칼로리 미만 메뉴
List<Dish> sliceMenu = specialMenu.stream()
                .takeWhile(dish -> dish.getCalories() < 320)
                .collect(Collectors.toList());
```

- `dropWhile()`: 프레디케이트가 false가 되는 지점까지 발견된 요소를 버리고 작업을 중단해 나머지 요소를 반환(나머지를 반환)
  - takeWhile과 정반대의 작업을 수행

```java
// 320 칼로리 이상 메뉴
List<Dish> highCaloricDishes = specialMenu.stream()
                .dropWhile(dish -> dish.getCalories() < 320)
                .collect(Collectors.toList());
```

## 5.2.2 스트림 축소

- `limit(n)`: 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환
  - 스트림이 정렬되어 있으면 최대 요소 n개를 반환
  - 스트림이 정렬되어 있지 않으면 정렬되지 않은 상태로 반환

```java
List<Dish> sliceMenu = specialMenu.stream()
                .takeWhile(dish -> dish.getCalories() > 300)
                .limit(3)
                .collect(Collectors.toList());
```

![스트림 축소](/assets/images/modernjavainaction/ch05/StreamLimit.png "스트림 축소")

## 5.2.3 요소 건너뛰기

- `skip(n)`: 처음 n개 요소를 제외한 스트림을 반환
  - n개 이하의 요소를 포함하는 스트림에 호출하면 빈 스트림이 반환
  - limit과 상호 보완적인 연산을 수행

```java
List<Dish> sliceMenu = specialMenu.stream()
                .takeWhile(dish -> dish.getCalories() > 300)
                .skip(2)
                .collect(Collectors.toList());
```

![요소 건너뛰기](/assets/images/modernjavainaction/ch05/StreamSkip.png "요소 건너뛰기")

<br>

# 5.3 매핑

특정 객체에서 특정 데이터 선택 작업은 데이터 처리 과정에서 자주 수행되는 연산이다. 스트림 API의 `map`과 `flatMap` 메서드는 특정 데이터를 선택하는 기능을 제공한다.

## 5.3.1 스트림의 각 요소에 함수 적용

- `map()`: 함수를 인수로 받는 메서드, 인수로 제공된 함수를 각 요소에 적용해 새로운 요소로 매핑된 스트림을 반환
- 기존의 값을 고치는 것 보다 **새로운 버전을 만든다**라는 개념에 가까워 **변환(transforming)**에 가까운 **매핑(mapping)**이라는 용어를 사용

```java
List<String> dishNames = menu.stream()
                .map(Dish::getName) // Stream<String> 반환
                .collect(Collectors.toList());
```

## 5.3.2 스트림 평면화

map을 이용해 고유 문자로 이루어진 리스트를 반환하기 위해서는 어떻게 해야하는가?

예시로`["Hello", "World"]` 리스트가 있다면 결과로 `["H", "e", "l", "o", "W", "r", "d"]`를 포함하는 리스트가 반환되어야 한다.

문자로 매핑 후 distinct로 고유 문자를 필터링한다고 가정하면 `Stream<String[]>`이 반환되어 원하는 타입인 `Stream<String>`이 아니게 된다.

```java
List<String> words = Arrays.asList("Hello", "World");
words.stream()
    .map(word -> word.split(""))
    .distinct() // Stream<String[]>
    .forEach(System.out::print);
```

**flatMap 메서드를 이용하면 위 문제를 해결할 수 있다.**

### map과 Arrays.stream 활용

우선 스트림 대신 **문자열 스트림이 필요**하다. 문자열을 받아 스트림을 만드는 `Arrays.stream()` 메서드를 호출한다.

```java
words.stream()
    .map(word -> word.split("")) // 각 단어를 개별 문자열 배열로 변환
    .map(Arrays::stream) // 각 배열을 별도의 스트림으로 생성
    .distinct() // Stream<Stream<String>>
    .forEach(System.out::print);
```

### flatMap 사용

- `flatMap`: 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑. 즉, 하나의 평면화된 스트림을 반환
  - 스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행

```java
words.stream()
    .map(word -> word.split("")) // 각 단어를 개별 문자열 배열로 변환
    .flatMap(Arrays::stream) // 생성된 스트림을 하나의 스트림으로 평면화, Stream<String>
    .distinct() // Stream<String>
    .forEach(System.out::print);
```

![스트림 flatMap](/assets/images/modernjavainaction/ch05/StreamFlatMap.png "스트림 flatMap")

<br>

# 5.4 검색과 매칭

스트림 API는 **특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리**를 위해 다양한 유틸리티 메서드를 제공한다.

## 5.4.1 프레디케이트가 적어도 한 요소와 일치하는지 확인

- `anyMatch`: 프레디케이트가 주어진 스트림에서 적어도 한 요소와 일치하는지 확인할 때 사용
  - 불리언을 반환하므로 최종 연산

```java
if (menu.stream()
        .anyMatch(Dish::isVegetarian)) {
    System.out.println("The menu includes dishes for vegetarians.");
} else {
    System.out.println("Nothing for vegetarians.");
}
```

## 5.4.2 프레디케이트가 모든 요소와 일치하는지 검사

- `allMatch`: 프레디케이트가 주어진 스트림에서 모든 요소가 주어진 프레디케이트와 일치하는지 검사
  - 불리언을 반환하므로 최종 연산

```java
if (menu.stream()
        .allMatch(dish -> dish.getCalories() < 1000)) {
    System.out.println("All dishes are healthy.");
}
```

- `noneMatch`: allMatch와 반대 연산을 수행. 즉, 주어진 프레디케이트와 일치하는 요소가 없는지 확인.
  - 불리언을 반환하므로 최종 연산

```java
if (menu.stream()
        .noneMatch(dish -> dish.getCalories() >= 1000)) {
    System.out.println("All dishes are healthy.");
}
```

anyMatch, allMatch, noneMatch는 스트림 **쇼트서킷** 기법을 활용해 최적의 계산을 지원한다.

> **쇼트서킷**: `&&`, `||` 연산의 특성을 활용해 모든 연산을 처리하지 않고 참, 거짓을 반별하는 기법. 만약 and 연산에서 하나라도 거짓이 존재하면 나머지 결과와 상관없이 전체 결과가 거짓이 되고 or 연산에서는 하나라도 참이 존재하면 나머지 결과와 상관없이 전체 결과가 참이 된다.

## 5.4.3 요소 검색

- `findAny`: 현재 스트림에서 임의의 요소를 반환
  - 다른 스트림연산과 연결해서 사용 가능

```java
Optional<Dish> dish = menu.stream()
                .filter(Dish::isVegetarian)
                .findAny();
```

> 스트림 파이프라인은 내부적으로 단일 과정으로 실행할 수 있도록 최적화된다. 즉, 쇼트서킷을 이용해 결과를 찾는 즉시 실행을 종료한다.

- `Optional<T>`: 값의 존재나 부재 여부를 표현하는 컨테이너 클래스
  - null은 쉽게 에러를 일으킬 수 있어 자바 8부터 Optional클래스를 제공
  - null 확인 관련 버그를 피할 수 있음
  - 값이 존재하는지 확인하고 값이 없을 때 어떻게 처리할지 강제하는 기능을 제공
  - `isPresent()`: 값을 포함하면 true, 포함하지 않으면 false를 반환
  - `ifPresent(Consumer<T> block)`: 값이 있으면 block을 실행, 없으면 아무 일도 일어나지 않음

```java
menu.stream()
    .filter(Dish::isVegetarian)
    .findAny() // Optional<Dish>
    .ifPresent(dish -> System.out.println(dish.getName())); 
```

## 5.4.4 첫 번째 요소 찾기

리스트 또는 정렬된 연속 데이터로부터 생성된 스트림처럼 일부 스트림에는 **논리적인 아이템 순서**가 정해져 있을 수 있어 첫 번째 요소를 찾기 위한 기능이 필요하다.

- `findFirst`: 스트림에서 첫 번째 요소를 반환

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> firstSquareDivisibleByThree = numbers.stream()
                                                    .map(i -> i * i)
                                                    .filter(i -> i % 3 == 0)
                                                    .findFirst();
```

> `findFirst` vs `findAny`: 두 메서드가 따로 분리된 이유는 **병렬성** 때문이다. 병렬 실행에서는 첫 번째 요소를 찾기 어렵다. 요소의 반환 순서가 필요 없다면 병렬 스트림에서는 제약이 적은 findAny를 사용한다.

<br>

# 5.5 리듀싱

리듀스(reduce)연산을 이용하면 스트림 요소를 조합해서 더 복잡한 질의를 표현할 수 있다.

- 리듀싱 연산: 모든 스트림 요소를 처리해서 Integer 같은 값으로 도출
  - 함수형 프로그래밍에서는 마치 종이를 작은 조각이 될 때까지 반복해서 접는 것과 비슷하다는 의미로 **폴드(fold)**라고 부른다.

## 5.5.1 요소의 합

for-each 루프를 이용해 리스트의 숫자 요소를 더하는 코드는 리스트의 각 요소를 결과에 반복적으로 더하고, 하나의 요소가 남을 때까지 reduce과정을 반복한다.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int result = 0;
for (int number : numbers) {
    result += number;
}
```

이 코드는 두 개의 파라미터를 사용한다.

- result 변수의 초기값 0
- 리스트의 모든 요소를 조합하는 연산(+)

위 코드를 복사&붙여넣기하지 않고 다른 사칙연산을 구현하려면 어떻게 해야할까?

- **reduce를 이용하면 반복된 패턴을 추상화**할 수 있다.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int sum = numbers.stream().reduce(0, Integer::sum);
int product = numbers.stream().reduce(1, (a, b) -> a * b);
```

reduce는 두 개의 인수를 갖는다.

- 초기값
- 두 요소를 조합해서 새로운 값을 만드는 `BinaryOperator<T>`

sum의 연산 과정은 람다의 첫 번째 파라미터에 0이 사용되었고, 스트림에서 1을 소비해 두 번째 파라미터로 사용한다. 0 + 1 결과인 1이 새로운 누적값이 되었고, 람다를 다시 호출하면 다음 요소인 2를 소비한다. 결과는 3이 되고 이런 식으로 다음 요소를 계속해서 소비하면서 누적값을 갱신한다.

- **초기값이 없는 경우**: 초기값을 받지 않도록 오버로드된 reduce도 존재한다. 이 reduce는 Optional 객체를 반환한다. 만약 스트림에 아무 요소도 없는 상황에서는 값이 없음을 가리킬 수 있도록 Optional 객체로 감싼 결과를 반환해야 한다.

## 5.5.2 최대값과 최소값

두 요소에서 최대값과 최소값을 비교할 수 있는 람다만 존재하면 reduce 연산을 이용해 구할 수 있다. 새로운 값을 이용해서 스트림의 모든 요소를 소비할 때까지 람다를 반복 수행하면서 최대값과 최소값을 생산한다.

```java
Optional<Integer> max = numbers.stream().reduce(Integer::max);
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```

### reduce 메서드의 장점과 병렬화

reduce를 이용하면 내부 반복이 추상화되면서 내부 구현에서 병렬로 reduce를 실행할 수 있게 된다. 하지만 반복적인 합계에서는 변수를 공유해야 하므로 쉽게 병렬화하기 어렵다. 강제적으로 동기화시키더라도 스레드 간의 소모적인 경쟁 때문에 이득이 없다. **가변 누적자 패턴(nutable accumulator pattern)은 병렬화와 거리가 너무 먼 기법**이다. 병렬화를 위해 **포크/조인 프레임워크(fork/join framework)**를 이용해 입력을 분할하고, 분할된 입력을 더한 다음 각 결과 값을 합치는 방법이 존재한다.

### 스트림 연산: 상태 없음과 상태 있음

- map, filter 등은 입력 스트림에서 각 요소를 받아 0 또는 결과를 출력 스트림으로 보낸다. 따라서 사용자가 제공한 람다나 메서드 참조가 내부적인 가변 상태를 갖지 않는다면 상태가 없는, 즉 **내부 상태를 갖지 않는 연산(stateless operation)**이다.
- reduce, max, min 같은 연산은 결과를 누적할 내부 상태가 필요하다. 스트림에서 처리하는 요소 수와 관계없이 **내부 상태의 크기는 한정(bounded)**되어 있다. sorted, distinct 같은 연산은 과거의 이력을 알고 있어야 정렬, 중복 제거가 가능하다. 만약 어떤 요소를 출력 스트림으로 추가하려면 모든 요소가 버퍼에 추가되어 있어야 한다. **연산을 수행하는 데 필요한 크기가 정해져있지 않아 데이터 스트림의 크기가 너무 크거나 무한이라면 문제**가 생길 수 있다. 이러한 연산을 **내부 상태를 갖는 연산(stateful operation)**이라 한다.

<br>

# 5.6 실전 연습

- Transaction.java

```java
public class Transaction {
    private final Trader trader;
    private final int year;
    private final int value;

    public Transaction(Trader trader, int year, int value) {
        this.trader = trader;
        this.year = year;
        this.value = value;
    }

    public Trader getTrader() {
        return trader;
    }

    public int getYear() {
        return year;
    }

    public int getValue() {
        return value;
    }

    @Override
    public String toString() {
        return "{" + this.trader + ", " + "year: " + this.year + ", " + "value: " + this.value + "}";
    }
}
```

- Trader.java

```java
public class Trader {
    private final String name;
    private final String city;

    public Trader(String name, String city) {
        this.name = name;
        this.city = city;
    }

    public String getName() {
        return name;
    }

    public String getCity() {
        return city;
    }

    @Override
    public String toString() {
        return "Trader: " + this.name + " in " + this.city;
    }
}
```

- StreamPractice.java

```java
import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

public class StreamPractice {
    public static void main(String[] args) {
        Trader raoul = new Trader("Raoul", "Cambridge");
        Trader mario = new Trader("Mario", "Milan");
        Trader alan = new Trader("Alan", "Cambridge");
        Trader brian = new Trader("Brian", "Cambridge");

        List<Transaction> transactions = Arrays.asList(
                new Transaction(brian, 2011, 300),
                new Transaction(raoul, 2012, 1000),
                new Transaction(raoul, 2011, 400),
                new Transaction(mario, 2012, 710),
                new Transaction(mario, 2012, 700),
                new Transaction(alan, 2012, 950)
        );

        // 1. 2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정리
        List<Transaction> transaction2011 = transactions.stream()
                .filter(tr -> tr.getYear() == 2011)
                .sorted(Comparator.comparing(Transaction::getValue))
                .collect(Collectors.toList());

        System.out.println("2011년 거래(오름차순)");
        for (Transaction transaction : transaction2011) {
            System.out.println(transaction);
        }

        // 2. 거래자가 근무하는 모든 도시를 중복 없이 나열하시오.
        List<String> cities = transactions.stream()
                .map(Transaction::getTrader)
                .map(Trader::getCity)
                .distinct()
                .collect(Collectors.toList());

        /*
        transactions.stream()
                .map(transaction -> transaction.getTrader().getCity())
                .distinct()
                .collect(Collectors.toList());
         */

        System.out.println("거래자가 근무하는 모든 도시들: " + cities);

        // 3. Cambridge 에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬
        List<Trader> tradersInCambridge = transactions.stream()
                .map(Transaction::getTrader)
                .filter(trader -> trader.getCity().equals("Cambridge"))
                .distinct()
                .sorted(Comparator.comparing(Trader::getName))
                .collect(Collectors.toList());

        System.out.println("Cambridge에서 근무하는 트레이더: " + tradersInCambridge);

        // 4. 모든 거래자의 이름을 알파벳순으로 정렬해서 반환
        List<String> traders = transactions.stream()
                .map(transaction -> transaction.getTrader().getName())
                .distinct()
                .sorted()
                .collect(Collectors.toList());
        System.out.println("트레이더: " + traders);

        // String 형태로 반환하는 법
        String traderStr = transactions.stream()
                .map(transaction -> transaction.getTrader().getName())
                .distinct()
                .sorted()
                .collect(Collectors.joining());
        System.out.println(traderStr);

        // 5. 밀라노에 거래자가 있는가?
        if (transactions.stream().anyMatch(transaction -> transaction.getTrader().getCity().equals("Milan"))) {
            System.out.println("밀라노에 거래자가 있습니다.");
        } else {
            System.out.println("밀라노에 거래자가 없습니다.");
        }

        // 6. 케임브리지에 거주하는 거래자의 모든 트랜잭션값을 출력하시오.
        System.out.println("케임브리지 거래자의 트랜잭션값들");
        transactions.stream()
                .filter(transaction -> transaction.getTrader().getCity().equals("Cambridge"))
                .map(Transaction::getValue)
                .forEach(System.out::println);

        // 7. 전채 트랜잭션 중 최댓값은 얼마인가?
        Optional<Integer> max = transactions.stream()
                .map(Transaction::getValue)
                .reduce(Integer::max);

        Optional<Transaction> maxTransaction = transactions.stream()
                .max(Comparator.comparing(Transaction::getValue));

        // 8. 전체 트랜잭션 중 최솟값은 얼마인가?
        Optional<Integer> min = transactions.stream()
                .map(Transaction::getValue)
                .reduce(Integer::min);

        Optional<Transaction> minTransaction = transactions.stream()
                        .min(Comparator.comparing(Transaction::getValue));

        System.out.println(max + "," + min);
        System.out.println(maxTransaction + ", " + minTransaction);
      
        // 거래가 없을 때 기본 문자열을 사용할 수 있도록발견된 거래가 있으면 문자열로 바꾸는 꼼수를 사용함(예, the Stream is empty)
        System.out.println(minTransaction.map(String::valueOf).orElse("No transactions found"));
    }
}
```

<br>

# 5.7 숫자형 스트림

reduce를 이용해 기본형(int, long 등)의 값을 얻는데 박싱 비용이 존대한다. 만약 메뉴의 칼로리 합을 구하는 스트림이면 내부적으로 합계를 계산하기 전에 Integer를 기본형으로 언박싱해야 한다.

```java
int calories = menu.stream().map(Dish::getCalories).sum(); // ERROR
```

위 코드처럼 sum 메서드를 직접 호출할 수 없다. map 메서드가 `Stream<T>`를 생성하기 때문이다. 다행히도 스트림은 숫자 스트림을 효율적으로 처리할 수 있도록 **기본형 특화 스트림(primitive stream specialization)**을 제공한다.

## 5.7.1 기본형 특화 스트림

스트림 API는 **박싱 비용을 피할 수 있도록 세 가지 기본형 특화 스트림**을 제공한다.

- `IntStream`: int 요소에 특화된 기본형 스트림
- `DoubleStream`: double 요소에 특화된 기본형 스트림
- `LongStream`: long 요소에 특화된 기본형 스트림

각각의 인터페이스는 숫자 스트림의 합계를 계산하는 sum, 최대값 검색 max, 최소값 검색 min 같이 자주 사용되는 숫자 관련 리듀싱 연산 수행 메서드를 제공한다. 또 필요할 때 다시 객체 스트림으로 복원하는 기능도 제공한다.

특화 스트림은 **오직 박싱 과정 효율성과 관련** 있으며 스트림에 추가 기능을 제공하지는 않는다.

### 숫자 스트림으로 매핑

스트림을 기본형 특화 스트림으로 변환하기 위해 `mapToInt`, `mapToDouble`, `mapToLong` 세 메서드를 가장 많이 사용한다. map과 같은 기능을 수행하지만 특화된 스트림을 반환한다.

```java
int calories = menu.stream()
                .mapToInt(Dish::getCalories) // IntStream
                .sum(); // max, min, average 등 다양한 유틸리티 메서드 지원
```

### 객체 스트림으로 복원하기

기본형 특화 스트림은 기본형의 정수값만 만들 수 있다. 만약 정수가 아닌 객체와 같은 다른 값을 반환하고 싶으면`boxed()`를 이용해기본형 특화 스트림을 원상태인 특화되지 않은 스트림으로 복원할 수 있다.

```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
Stream<Integer> stream = intStream.boxed();
```

### 기본값: OptionalInt

합계를 구하는 예제는 문제가 없지만 최대값을 찾을 때는 0이라는 기본값 때문에 잘못된 결과가 나올 수 있다. 스트림에 요소가 없는 상황과 실제 최대값이 0인 상황을 구별하기 위해 `Optional` 클래스를 사용한다.

Optional을 Integer, String 등의 참조 형식으로 파라미터화할 수 있고, 세 가지 기본형 특화 스트림 버전인 `OptionalInt`, `OptionalDouble`, `OptionalLong`을 제공한다.

```java
OptionalInt maxCalories = menu.stream()
                            .mapToInt(Dish::getCalories)
                            .max();

int max = maxCalories.orElse(1); // 값이 없을 때 기본 최대값을 명시적으로 설정
```

## 5.7.2 숫자 범위

프로그램에서 특정 범위의 숫자를 이용해야 하는 상황이 자주 발생한다. 자바 8은 IntStream과 LongStream에서 두 가지 정적 메서드를 제공한다.

- `range(int startInclusive, int endInclusive)`: 시작값과 종료값이 포함되지 않는 범위의 숫자를 생성
  - `range(1, 100)`: 2 부터 99 까지의 숫자를 생성

- `rangeClosed(int startInclusive, int endInclusive)`: 시작값과 종료값이 포함되는 범위의 숫자를 생성
  - `rangeClosed(1, 100)`: 1 부터 100 까지의 숫자를 생성


```java
IntStream evenNumbers = IntStream.rangeClosed(1, 100)
                .filter(n -> n % 2 == 0); // 1 부터 100 까지의 짝수 스트림
System.out.println(evenNumbers.count()); // 50
```

<br>

# 5.8 스트림 만들기

스트림은 **데이터 처리 질의를 표현하는 강력한 도구**이다. 컬렉션에서 스트림을 얻을 수 있고, 범위의 숫자에서 스트림을 만들 수 있으며 일련의 값, 배열, 파일, 함수를 이용한 무한 스트림도 만들 수 있다.

## 5.8.1 값으로 스트림 만들기

- `Stream.of()`: 임의의 수를 인수로 받아 스트림을 생성하는 정적 메서드

```java
Stream<String> stream = Stream.of("Modern ", "Java ", "In ", "Action");
stream.map(String::toUpperCase).forEach(System.out::println);
```

- `empty()`: 스트림을 비우는 정적 메서드

```java
Stream<String> emptyStream = Stream.empty();
```

## 5.8.2 null이 될 수 있는 객체로 스트림 만들기

자바 9에서는 null이 될 수 있는 객체를 스트림으로 만들 수 있는 새로운 메서드가 추가되었다.

- `Stream.ofNullable()`: null이 될 수 있는 객체를 스트림으로 만들 수 있는 정적 메서드

```java
String homeValue = System.getProperty("home"); // 제공된 키에 대응하는 속성이 없으면 null을 반환

Stream<String> homeValueStream = Stream.ofNullable(System.getProperty("home"));

Stream<String> valueStream = Stream.of("config", "home", "user")
                .flatMap(key -> Stream.ofNullable(System.getProperty(key)));
```

## 5.8.3 배열로 스트림 만들기

- `Arrays.stream()`: 배열을 인수로 받아 스트림을 생성하는 정적 메서드

```java
int[] numbers = {2, 3, 5, 7, 11, 13};
int sum = Arrays.stream(numbers).sum();
```

## 5.8.4 파일로 스트림 만들기

파일을 처리하는 등의 입출력 연산에 사용하는 자바의 NIO API(비블록 I/O)도 스트림 API를 활용할 수 있다.

```java
        long uniqueWordCount = 0L; // 파일에 존재하는 고유 단어 개수
        try (Stream<String> lines =
                     Files.lines(Paths.get("data.txt"), Charset.defaultCharset())) {
            uniqueWordCount = lines.flatMap(line -> Arrays.stream(line.split(", ")))
                    .distinct()
                    .count();
        } catch (IOException e) {
            e.printStackTrace();
        }
```

## 5.8.5 함수로 무한 스트림 만들기

스트림 API는 함수에서 스트림을 만들 수 있는 두 개의 정적 메서드 `Stream.iterate`, `Stream.generate`를 제공한다.

두 연산을 이용하면 **무한 스트림(infinite stream)**을 만들 수 있다.

- **무한 스트림(infinite stream)**: 고정된 컬렉션에서 만든 스트림과 달리 크기가 고정되지 않은 스트림
  - 무한한 값을 출력하지 않고 limit 메서드와 함께 사용된다.

### iterate 메서드

- `Stream.iterate(초기값, 람다식)`: 초기값과 람다식을 인수로 받아서 새로운 값을 끊임없이 생산
  - 기존 결과에 의존해서 순차적으로 연산을 수행
  - 요청할 때마다 값을 생산할 수 있어 무한 스트림을 만든다. 이러한 스트림을 **Unbounded Stream**이라고 표현한다.
  - 일반적으로 연속된 일련의 값을 만들 때 사용한다.

```java
Stream.iterate(0, n -> n + 2)
    .limit(10)
    .forEach(System.out::println); // 짝수 스트림을 생성해 10개를 출력 0 ~ 18
```

- 자바 9의 iterate 메서드는 2번째 인수를 프레디케이트를 받을 수 있도록 지원한다.

```java
// 0 에서 시작해 100보다 커지면 중단
IntStream.iterate(0, n -> n < 100, n -> n + 4)
        .forEach(System.out::println);

// filter 메서드는 언제 이 작업을 중단해야 하는지 알 수 없어 종료되지 않는다.
IntStream.iterate(0, n -> n + 4)
        .filter(n -> n < 100)
        .forEach(System.out::println)

// takeWhile을 이용하면 쇼트서킷을 통해 중단된다.
IntStream.iterate(0, n -> n + 4)
        .takeWhile(n -> n < 100)
        .forEach(System.out::println)
```

### generate 메서드

- `Stream.generate(Supplier<T>)`: iterate 메서드와 비슷하게 요구할 때 값을 계산하는 무한 스트림을 생산
  - iterate와 달리 생산된 각 값을 연속적으로 계산하지 않는다.

```java
Stream.generate(Math::random)
    .limit(5)
    .forEach(System.out::println);
```

> 스트림을 병렬로 처리하면서 올바른 결과를 얻으려면 **불변 상태 기법**을 고수해야 한다. 만약 가변(mutable) 상태 객체를 사용하면 잘못된 결과를 얻을 수 있다.

<br>

# 5.9 Summary

- 스트림 API를 이용하면 복잡한 데이터 처리 질의를 표현할 수 있다.
- filter, distinct, takeWhile(Java 9), dropWhile(Java 9), skip, limit 메서드로 스트림을 필터링하거나 자른다.
  - 데이터 소스가 정렬되어 있다는 사실을 알면 takeWhile과 dropWhile 메서드를 효과적으로 사용할 수 있다.
- map, flatMap 메서드로 스트림의 요소를 추출하거나 변환한다.
- findFirst, findAny 메서드로 스트림의 요소를 검색한다. allMatch, noneMatch, anyMatch 메서드로 주어진 프레디케이트와 일치하는 요소를 검색할 수 있다.
  - 쇼트서킷 기법을 통해 최적의 계산을 한다.
- reduce 메서드로 스트림의 모든 요소를 반복 조합하며 값을 도출할 수 있다.
- filter, map 등은 상태를 저장하지 않는 **상태 없는 연산**이다. reduce 같은 연산은 값을 계산하는 데 필요한 상태를 저장한다. sorted, distinct 등의 메서드는 과거의 정보를 이용해야 하기 때문에 모든 요소를 버퍼에 저장한다. 이런 메서드를 **상태 있는 연산**이라 한다.
- 박싱 비용을 줄이기 위해 기본형 특화 스트림 3가지를 제공한다.
  - IntStream, DoubleStream, LongStream