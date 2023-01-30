---
title: "[모던 자바 인 액션] 03. 람다 표현식 - part2"
categories: [Modern Java in Action]
tag: ["Java"]
author_profile: false
sidebar:
    nav: "counts"
---

[[모던 자바 인 액션] 03. 람다 표현식 - part1](https://ykmxxi.github.io/modern%20java%20in%20action/modernjava-chapter03-part1/)

<br>

# 3.6 메서드 참조

- 메소드 참조를 이용하면 **기존의 메서드 정의를 재활용해서 람다처럼 전달**할 수 있다.
- 하나의 메서드를 참조하는 람다를 편리하게 표현할 수 있는 문법으로 생각

```java
// 기존의 코드
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

// 메서드 참조
inventory.sort(Apple::getWeight);
```

## 3.6.1 요약

메서드 참조는 **특정 메서드만을 호출하는 람다의 축약형**이라고 생각할 수 있다. 람다가 '어떤 메소드를 직접 호출해' 라고 명령하면 메서드명을 직접 참조하는 것이 편리하다.

메서드 참조를 이용하면 기존 메서드 구현으로 람다 표현식을 만들 수 있다.

- **명시적으로 메서드명을 참조해 가독성을 높인다.**
- 실제로 메서드를 호출하는 것은 아니므로 괄호는 필요 없다.

```java
// 기존의 코드
() -> Thread.currentThread().dumpStack()
(str, i) -> str.substring(i)
(String s) -> System.out.println(s)
(String s) -> this.isValidName(s)

// 메서드 참조
Thread.currentThread()::dumStack
String::substring
System.out::println
this::isValidName
```

### 메서드 참조를 만드는 방법

메서드 참조는 세 가지 유형으로 구분할 수 있다.

- **정적 메서드 참조**

```java
Integer::parseInt
Double::parseDouble
```

- **다양한 인스턴스 메서드 참조**

```java
String::length
String::toUpperCase
String::toLowerCase
```

- **기존 객체의 인스턴스 메서드 참조**
  - 람다 표현식에서 현존하는 외부 객체의 메서드를 호출할 때 사용

생성자, 배열 생성자, super 호출 등에 사용할 수 있는 특별한 형식의 메서드 참조도 있다. 만약 List에 포함된 문자열을 대소문자 구분없이 정렬하는 프로그램을 구현하려 하면, List의 sort메서드는 인수로 Comparator를 기대한다. 이 때 String클래스에 정의되어 있는 compareToIgnoreCase 메서드로 람다 표현식을 정의할 수 있다.

```java
List<String> listOfStrings = Arrays.asList("d", "A", "C", "b", "a");
// 기존 람다 표현식
listOfStrings.sort((s1, s2) -> s1.compareToIgnoreCase(s2));

// 메서드 참조 이용
listOfStrings.sort(String::compareToIgnoreCase);
```

- 컴파일러는 메서드 참조가 주어진 함수형 인터페이스와 호환하는지 확인한다. 즉, **메서드 참조는 콘텍스트의 형식과 일치해야 한다.**

## 3.6.2 생성자 참조

- 클래스명과 new 키워드를 이용해서 기존 생성자의 참조를 만들 수 있다. (정적 메서드 참조와 비슷)

```java
Supplier<Apple> c1 = () -> new Apple();
Apple a1 = c1.get();

// 생성자 참조
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get();

Function<Integer, Apple> c2 = (weight) -> new Apple(weight);
Apple a2 = c2.apply(110);

// 생성자 참조
Function<Integer, Apple> c2 = Apple::new;
Apple a2 = c2.apply(110);
```

각 리스트의 요소를 포함하는 사과 리스트를 생성자 참조로 만들면 아래와 같다.

```java
public class GetApples {
    public static void main(String[] args) {
        List<Integer> weights = Arrays.asList(100, 150, 200, 300, 160);
        List<String> colors = Arrays.asList("Green", "Red", "Green", "Red", "Green");

        List<Apple> apples = map(weights, colors, Apple::new);
        for (Apple apple : apples) {
            System.out.println(apple);
        }
    }

    public static List<Apple> map(List<Integer> weights, List<String> colors, BiFunction<Integer, String, Apple> f) {
        List<Apple> result = new ArrayList<>();

        for (int i = 0; i < weights.size(); i++) {
            result.add(f.apply(weights.get(i), colors.get(i)));
        }
        return result;
    }
}
```

<br>

# 3.7 람다, 메서드 참조 활용

사과 리스트를 람다 표현식과 메서드 참조를 활용해 간결하게 해결하는 방법을 살펴본다.

## 3.7.1 1단계: 코드 전달

자바 8의 List API에서 sort 메서드를 이용한다. sort 메서드는 Comparator 객체를 인수로 받아 비교하므로, 필요한 Comparator를 정의해 **sort의 동작을 파라미터화** 시킨다.

```java
static class AppleComparator implements Comparator<Apple> {
    @Override
    public int compare(Apple o1, Apple o2) {
        return o1.getWeight() - o2.getWeight();
    }
}

inventory.sort(new AppleComparator());
```

## 3.7.2 2단계: 익명 클래스 사용

만약 한 번만 사용하는 Comparator 이면, 위 코드처럼 구현하기 보단 **익명 클래스**를 이용하는 것이 좋다.

```java
inventory.sort(new Comparator<Apple>() {
    public int compare(Apple o1, Apple o2) {
        return o1.getWeight() - o2.getWeight();
    }
});
```

## 3.7.3 3단계: 람다 표현식 사용

Comparator의 함수 디스크립터는 `(T, T) -> int` 이다. 사과 정렬에서는 더 정확히 `(Apple, Apple) -> int`로 표현할 수 있다.

```java
inventory.sort((a1, a2) -> // 파라미터 형식(Apple)은 추론 가능하니 생략한다.
        (a1.getWeight() - a2.getWeight())
);
```

가독성을 더 향상시키기 위해 Comparator의 정적 메서드인 `comparing()`을 이용한다.

- `comparing()`: Comparator의 정적 메서드로 Comparable 키를 추출해서 Comparator 객체로 만드는 Function함수를 인수로 받는 메서드이다.

```java
import static java.util.Comparator.comparing;

inventory.sort(comparing(apple -> apple.getWeight()));
```

## 3.7.4 4단계: 메서드 참조 사용

위 코드를 더 깔끔하게 전달하기 위해 메서드 참조를 사용한다.

```java
import static java.util.Comparator.comparing;

inventory.sort(comparing(Apple::getWeight));
```

- 단지 코드만 짧아진것이 아니라 의미도 명확해졌다. 즉, **코드 자체의 의미를 생각하면 "Apple을 weight별로 비교해서 inventory를 sort하라"는 의미를 전달할 수 있다.**

<br>

# 3.8 람다 표현식을 조합할 수 있는 메서드

자바 8의 몇몇 함수형 인터페이스는 람다 표현식을 조합할 수 있도록 다양한 **유틸리티 메서드**를 포함한다. 즉, 간단한 여러 개의 **람다 표현식을 조합해서 복잡한 람다 표현식**을 만들 수 있다는 것이다.

예를 들어 두 Predicate를 조합해서 두 조건의 or 연산을 수행하는 Predicate를 만들 수 있다. 또 한 함수의 결과가 다른 함수의 입력이 되도록 두 함수를 조합할 수도 있다.

함수형 인터페이스에서는 **디폴트 메서드(default method)**를 제공하기 때문에 람다 표현식을 조합할 수 있다. 디폴트 메서드는 추상 메서드가 아니기 때문에 함수형 인터페이스의 정의에서도 벗어나지 않는다.

## 3.8.1 Comparator 조합

```java
import static java.util.Comparator.comparing;

inventory.sort(comparing(Apple::getWeight));
```

만약 사과의 무게를 오름차순이 아닌 내림차순으로 정렬하고 싶다면 인터페이스 자체에서 주어진 비교자의 순서를 뒤바꾸는 reversed라는 디폴트 메서드를 이용하면 된다.

```java
import static java.util.Comparator.comparing;

inventory.sort(comparing(Apple::getWeight).reversed());
```

만약 무게가 같은 사과가 존재한다면 어떤 사과를 먼저 정렬해야 하는지 고민에 빠질 수 있다. 이럴 땐 비교 결과를 더 다듬을 수 있는 **두 번째 Comparator를 만들 수 있다. 이를 Comparator 연결이라 한다.**

```java
import static java.util.Comparator.comparing;

inventory.sort(comparing(Apple::getWeight)
        .reversed() // 내림차순 정렬
        .thenComparing(Apple::getCountry)); // 두 사과 무게가 같으면, 국가별로 정렬
```

- `thenComparing()`: 두 번째 비교자를 만드는 메서드
  - comparing 메서드처럼 함수를 인수로 받아 첫 번째 비교자를 이용해서 두 객체가 같다고 판단되면 두 번째 비교자에게 객체를 전달

## 3.8.2 Predicate 조합

복잡한 Predicate를 만들 수 있도록 negate, and, or 세 가지 메서드를 제공한다.

- `negate()`: 기존 Predicate를 반전 시킬 때 사용

```java
Predicate<Apple> notRedApple = redApple.negate();
```

- `and()`: `&&` 와 같이 두 Predicate를 연결 시키는 메서드

```java
// 빨간색이면서 무거운 사과
Predicate<Apple> redAndHeavyApple = redApple.and(apple -> apple.getWeight() > 150);
```

- `or()`: `||`와 같이 두 Predicate를 연결 시키는 메서드
  - `(redApple || heavyApple) && greenApple`

```java
// 빨간색이면서 무거운 사과 또는 초록색 사과
Predicate<Apple> redAndHeavyAppleOrGreen =
                redApple.and(apple -> apple.getWeight() > 150)
                        .or(apple -> "Green".equals(apple.getColor()));
```

## 3.8.3 Function 조합

Function 인터페이스는 Function 인스턴스를 반환하는 andThen, compose 두 가지 디폴트 메서드를 제공한다.

- `andThen()`: 주어진 함수를 먼저 적용한 결과를 다른 함수의 입력으로 전달하는 함수를 반환

```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h = f.andThen(g); // g(f(x))

int result = h.apply(1); // 4를 반환, (1 + 1) * 2
```

- `compose()`: 인수로 주어진 함수를 먼저 실행한 다음에 그 결과를 외부 함수의 인수로 제공

```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h = f.compose(g); // f(g(x))

int result = h.apply(1); // 3을 반환, (1 * 2) + 1
```

<br>

# 3.9 Summary

- 람다 표현식은 익명 함수의 일종이다. 익명이지만 파라미터 리스트, 바디, 반환 형식을 가지며 예외를 던질 수 있다.
- 함수형 인터페이스는 하나의 추상 메서드만을 정의하는 인터페이스다.
  - `@FunctionalInterface` 애노테이션을 붙인다.
- 함수형 인터페이스를 기대하는 곳에서만 람다 표현식을 사용할 수 있다.
- 람다 표현식을 이용해 함수형 인터페이스의 추상 메서드를 바로 제공할 수 있으며 **람다 표현식 전체가 함수형 인터페이스의 인스턴스로 취급된다.**
- 람다 형식의 기대 형식(type expected)을 **대상 형식**(target type)이라고 한다.
- 메서드 참조를 이용하면 기존의 메서드 구현을 재사용하고 직접 전달할 수 있다.
- 함수형 인터페이스는 람다 표현식을 조합할 수 있도록 다양한 **디폴트 메서드**를 제공한다.