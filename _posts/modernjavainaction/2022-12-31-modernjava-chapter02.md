---
title: "[모던 자바 인 액션] 02. 동작 파라미터화 코드 전달하기"
categories: [Modern Java in Action]
tag: ["Java"]
author_profile: false
---

- **동작 파라미터화(behavior parameterization)**: 어떻게 실행할 것인지 결정하지 않은 코드 블록

동작 파라미터화를 이용하면 **자주 바뀌는 요구사항에 효과적으로 대응**할 수 있다. 이 코드 블록은 나중에 프로그램에서 호출한다. 즉, 코드 블록의 실행이 나중으로 미뤄진다는 의미이다.

예를 들어 주기적으로 마켓에 가는 친구에게 식료품을 사다 달라는 부탁을 하면 필요한 식료품의 목록만 친구에게 넘겨주면 된다. 이미 주기적으로 마켓을 왔다 갔다하니 길을 알려줄 필요는 없다. 이 동작을 `goAndBuy(List<String> grocery)`라 비유할 수 있다. 어느날 급한일이 생겨 우체국에 소포를 가져다달라는 부탁을 하면 처음 해보는 일이니 상세히 설명을 해줘야 한다. 우체국 까지 가는 길, 송장 번호, 소포를 찾는 법 등을 상세히 적은 리스트를 전달하면 부탁을 들어줄 것이다. 이를 좀 더 포괄적인 작업을 수행할 수 있는 `go()`에 비유할 수 있다. 원하는 동작(기능)을 `go()`의 인수로 전달해주면 된다.

동작 파라미터를 추가하려면 쓸데없는 코드가 늘어나지만, 람다 표현식으로 이 문제를 해결할 수 있다.

<br>

# 2.1 변화하는 요구사항에 대응하기

## 2.1.1 첫 번째 시도: 녹색 사과 필터링

```java
enum Color {RED, GREEN}

public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();

    for (Apple apple : inventory) {
        if (GREEN.equals(apple.getColor())) { // 사과를 선택하는 조건식
            result.add(apple);
        }
    }
    return result;
}
```

농부의 첫 요구사항이 녹색 사과를 필터링하는 기능이라 가정하여 메서드를 정의했다. 만약 농부가 녹색 이외에 다른색 사과를 필터링하는 기능을 요구하면, 조건식만 다르게 설정하면 되기 때문에 `if()`의 조건식만 다른 메서드를 정의하면 된다. 다양한 색이 추가될 수록 조건식만 다른 중복된 코드가 쌓여나가게 된다.

- **비슷한 코드가 반복 존재하면 그 코드를 추상화한다 !!!**

## 2.1.2 두 번째 시도: 색을 파라미터화

filterGreenApples()의 코드를 반복 사용하지 않고 filterRedApples()를 구현하려면 어떻게 추상화 시켜야 할까? **색을 파라미터화**할 수 있도록 **메서드에 파라미터를 추가**하면 **변화하는 요구사항에 좀더 유연하게 대응**하는 코드를 만들 수 있다.

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();

    for (Apple apple : inventory) {
        if (apple.getColor().equals(color)) {
            result.add(apple);
        }
    }
    return result;
}
```

메서드의 파라미터에 색을 추가해 원하는 색을 입력받으면 쉽게 구분할 수 있다. 또 무게에 따라서도 사과를 구분하고 싶어하면 파라미터에 색 대신 무게를 추가하면 될 것이다.

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
    List<Apple> result = new ArrayList<>();

    for (Apple apple : inventory) {
        if (apple.getWeight() > weight) {
            result.add(apple);
        }
    }
    return result;
}
```

하지만 색에 따른 구분과 무게에 따른 구분 메서드의 **코드가 대부분 중복**된다. 이는 소프트웨어 공학의 **DRY(Don’t repeat yourself, 같은 것을 반복하지 말 것) 원칙**을 어기는 것이다. 코드의 중복을 줄이기위해 **구분점이 되는 값들을 모두 파라미터에 추가시키는 메서드**를 만드는 것도 하나의 방법이다. (하지만 실전에서 이 방법을 사용하지 말아야한다.)

## 2.1.3 세 번째 시도: 가능한 모든 속성으로 필터링

색, 무게를 모두 파라미터로 추가한 메서드이다.

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {
    List<Apple> result = new ArrayList<>();

    for (Apple apple : inventory) {
        if ((flag && apple.getColor().equals(color)) ||
            (!flag && apple.getWeight() > weight)) {
            result.add(apple);
        }
    }
    return result;
}
```

모든 구분점이 되는 값을 파라미터에 추가시키는 위 코드는 크게 세 가지 문제점이 존재한다.

1. `flag`가 메서드를 작성한 사람 이외에는 무엇을 의미하는지 알기 어렵다. 코드는 다른 사람도 읽기 편해야 한다고 했다.
2. 무겁고 녹색인 사과를 원한다면 이 메서드로는 구분할 수 없다.
3. 구분점이 되는 값이 추가되면(출하지, 모양 등) 다시 메서드를 뜯어 고쳐야한다. 즉, 엔지니어링적으로 비싼 대가를 치러야 한다.

지금까지 문자열, 정수, 불리언 등의 값으로 메서드를 파라미터화했다. 이는 문제가 잘 정의되어 있는 상황에서는 잘 동작하나, **현실의 문제는 잘 정의되어 있지 않다**. 어떤 기준으로 어떻게 필터링할 것인지 효과적으로 전달할 수 있다면 더 좋을 것이다. 즉, **동작 파라미터화를 이용해 유연성을 얻어야 한다**.

<br>

# 2.2 동작 파라미터화

단순히 구분점이 되는 값들을 파라미터에 추가하는 방법이 아닌 **변화하는 요구사항에 좀 더 유연하게 대응**해야 한다. 사과의 어떤 구분점이 되는 값들에 기초해서 `boolean`값을 반환하는 방법이 있다. **참 또는 거짓을 반환**하는 함수 **Predicate**를 사용하는 방법이다. 이 방법은 **선택 조건을 결정하는 인터페이스를 정의**한다.

```java
public interface ApplePredicate {
    boolean test(Apple apple);
}
public class AppleHeavyWeightPredicate implements ApplePredicate {
    @Override
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}
public class AppleGreenColorPredicate implements ApplePredicate {
    @Override
    public boolean test(Apple apple) {
        return GREEN.equals(apple.getColor());
    }
}
```

- **전략 디자인 패턴**: 각 알고리즘(전략)을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음 런타임에 알고리즘을 선택하는 기법

여기서 ApplePredicate가 알고리즘 패밀리, 나머지 두 클래스가 전략이다. 알고리즘 패밀리 객체를 받아 조건을 검사하도록 필터링 메서드를 고치면 **다양한 동작을 받아서 실행**할 수 있다. 이를 **동작을 파라미터화**시킨다고 한다.

## 2.2.1 네 번째 시도: 추상적 조건으로 필터링

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();

    for (Apple apple : inventory) {
        if (p.test(apple)) {
            result.add(apple);
        }
    }
    return result;
}
```

이제 필요한 대로 다양한 ApplePredicate를 구현해 메서드로 전달할 수 있다. ApplePredicate를 적절히 구현한 클래스만 만들면 **다양한 요구사항에 대응**할 수 있다. 이제 전달한 ApplePredicate객체에 의해 메서드의 동작이 결정되니 엄청난 유연성을 즐길 수 있다. `filterApples()`의 동작을 **파라미터화**한 것이다.

메서드의 동작을 정의하는 것이 test 메서드 이기 때문에 가장 중요한 구현이다. 메서드는 객체만 인수로 받으므로 ApplePredicate 객체로 감싸 전달해야 한다. test 메서드를 구현한 객체를 이용해 **코드를 전달**한 것이다.

- **동작 파라미터화의 강점**: 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다. 따라서 한 메서드가 다른 동작을 수행하도록 재활용할 수 있어, 유연한 API를 만들 때 동작 파라미터화가 중요한 역할을 한다.

여러 클래스를 구현해서 인스턴스화하는 과정이 조금 번거로울 수 있다. 이 부분을 어떻게 개선할 수 있을까?

<br>

# 2.3 복잡한 과정 간소화

인터페이스를 구현하는 여러 클래스를 정의한 다음 인스턴스화하는 과정은 상당히 번거로운 작업이며 시간 낭비일 수 있다. 또 로직과 관련 없는 코드가 많이 추가될 수 있다.

자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 **익명 클래스(anonymous class)**라는 기법을 제공한다. 익명 클래스를 이용하면 코드의 양을 줄일 수 있다. 하지만 좋은 해결방법이 아니다.

## 2.3.1 익명 클래스

- 익명 클래스: 자바의 지역 클래스와 비슷한 개념으로, 이름이 없는 클래스다.
  - 클래스의 선언과 인스턴스화를 동시에 수행
  - 즉석에서 필요한 구현을 만들어서 사용가능

## 2.3.2 다섯 번째 시도: 익명 클래스 사용

다음은 익명 클래스를 이용해 새로운 객체를 만드는 예제이다.

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
    public boolean test(Apple apple) {
        return RED.equals(apple.getColor());
    }
});
```

익명클래스가 번거로운 작업을 줄이는 좋은 방법은 아니다. 여전히 문제점을 포함하고 있다.

1. 익명 클래스는 인스턴스를 구현한 클래스를 작성하는 것처럼 많은 공간을 차지한다. (반복되는 코드 발생)
2. 많은 프로그래머가 익명 클래스의 사용에 익숙하지 않다.

**코드의 장황함은 나쁜 특성이다**. 유지보수하는 데 시간이 오래 걸리고 읽기 어려운 코드가 되어 외면받기 쉽다. 익명 클래스를 사용하면 인터페이스를 구현하는 여러 클래스를 선언하는 과정을 조금 줄일 수 있지만 좋은 해결방법은 아니다. 이런 문제를 해결하기 위해 **람다 표현식**이 도입된다.

**동작 파라미터화는 변화하는 요구사항에 유연하게 대응할 수 있으므로 권장되는 기법**이다. 람다 표현식이라는 더 간단한 코드 전달 기법이 도입되었으니 간결하고 유연한 코드의 작성이 가능하다.

## 2.3.3 여섯 번째 시도: 람다 표현식 사용

```java
List<Apple> redApples =
    filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

간결해지면서 문제를 더 잘 설명하는 코드가 되었다. 람다 표현식을 사용하면 복잡성 문제를 해결하고 유연성과 간결함이라는 두 마리 토끼를 모두 잡을 수 있다.

## 2.3.4 일곱 번째 시도: 리스트 형식으로 추상화

```java
public interface Predicate<T> {
    boolean test(T t);
}

public class Filter {
    public static <T> List<T> filter(List<T> list, Predicate<T> p) { // 형식 파라미터 T 사용
        List<T> result = new ArrayList<>();

        for (T e : list) {
            if (p.test(e)) {
                result.add(e);
            }
        }
        return result;
    }
}
```

**형식 파라미터 T**를 이용하면 사과뿐 아니라 오렌지, 바나나, 배, 정수열, 문자열 등의 **모든 리스트에 필터 메서드를 사용**할 수 있다.

<br>

# 2.4 동작 파라미터화 실전 예제

동작 파라미터화는 변화하는 요구사항에 쉽게 적응하는 유용한 패턴임을 확인했다. 이 패턴은 동작을 캡슐화한 다음 메서드로 전달해서 메서드의 동작을 파라미터화한다. 코드 전달 개념을 확실히 익힐 수 있도록 실전 예제를 살펴보도록 한다. 자바 API의 많은 메서드는 정렬, 스레드 등을 포함한 다양한 동작으로 파라미터화할 수 있다.

## 2.4.1 Comparator로 정렬하기

자바 8의 List에는 `sort()`가 포함되어 있다. `java.util.Comparator` 객체를 이용해서 `sort()`의 동작을 파라미터화할 수 있다.

```java
public interface Comparator<T> {
    int compare(T o1, T o2);
}

inventory.sort(new Comparator<Apple>() {
    @Override
    public int compare(Apple o1, Apple o2) {
        return o1.getWeight().compareTo(o2.getWeight());
    }
});
```

람다 표현식을 이용한다면 더 간단하게 구현할 수 있다.

```java
inventory.sort((Comparator<Apple>) (o1, o2) -> o1.getWeight().compareTo(o2.getWeight()));
```

## 2.4.2 Runnable로 코드 블록 실행

자바에서는 Runnable 인터페이스를 이용해서 실행할 코드 블록을 지정할 수 있다.

```java
public interface Runnable {
    void run();
}

Thread t = new Thread(new Runnable() {
    public void run() {
        System.out.println("Hello World");
    }
});
```

람다 표현식을 이용한다면 더 간단하게 구현할 수 있다.

```java
Thread t = new Thread(() -> System.out.println("Hello World"));
```

## 2.4.3 Callable을 결과로 반환하기

- ExecutorService 추상화: 자바 5부터 지원
  - 태스크 제출과 실행 과정의 연관성을 끊어주는 인터페이스
  - 태스크를 스레드 풀로 보내고 결과를 Future로 저장
  - Runnable의 업그레이드 버전이라 생각하면 됨

```java
ExecutorService executorService = Executors.newCachedThreadPool();
Future<String> threadName = executorService.submit(new Callable<String>() {
    @Override
    public String call() throws Exception {
        return Thread.currentThread().getName();
    }
});
```

람다 표현식을 이용한다면 더 간단하게 구현할 수 있다.

```java
Future<String> threadName = executorService.submit(
        () -> Thread.currentThread().getName());
```

<br>

# 2.5 Summary

- 동작 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다.
- 동작 파라미터화를 이용하면 변화하는 요구사항에 더 잘 대응할 수 있는 코드를 구현할 수 있다.
- 코드 전달 기법을 이용하면 동작을 메서드의 인수로 전달할 수 있다. 익명 클래스, 인터페이스 구현을 이용해 코드를 전달할 수 있고, 자바 8부터는 람다 표현식을 이용해 간결하게 표현이 가능해졌다.