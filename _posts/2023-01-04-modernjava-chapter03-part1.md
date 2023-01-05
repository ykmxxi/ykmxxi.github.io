---
title: "[모던 자바 인 액션] 03. 람다 표현식 - part1"
categories: [Modern Java in Action]
tag: ["Java"]
toc: true
toc_sticky: true
author_profile: false
sidebar:
    nav: "docs"
---

[[모던 자바 인 액션] 03. 람다 표현식 - part2](https://ykmxxi.github.io/modern%20java%20in%20action/modernjava-chapter03-part2/)

<br>

람다 표현식은 **이름이 없는 함수면서 메서드를 인수로 전달**할 수 있으므로 익명 클래스와 비슷하다. 람다 표현식은 더 깔끔한 코드로 동작을 구현하고 전달한다. 메서드 참조와 함께 사용되면 큰 위력을 발휘할 수 있다.

<br>

# 3.1 람다란 무엇인가?

- **람다 표현식**: 메서드로 전달할 수 있는 익명 함수를 단순화한 것. 이름은 없지만(익명) 파라미터 리스트, 바디, 반환 형식, 발생할 수 있는 예외 리스트는 가질 수 있다.
  - **익명**: 보통의 메서드와 달리 이름이 없다.
  - **함수**: 메서드와 달리 **특정 클래스에 종속되지 않으므로 함수**라 부른다. 하지만 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함한다.
  - **전달**: 메서드 인수로 전달하거나 변수로 저장할 수 있다.
  - **간결성**: 익명 클래스처럼 많은 자질구레한 코드를 구현할 필요가 없다.

코드를 전달하는 과정에서 자질구레한 코드가 많이 생긴다. 람다는 **간결한 방식으로 코드를 전달**해 이 문제를 해결할 수 있다. 람다가 기술적으로 자바 8이전의 자바로 할 수 없었던 일을 제공하는 것은 아니고, 동작 파라미터를 이용할 때 **익명 클래스 등 판에 박힌 코드(지저분한 코드)를 구현할 필요가 없게하는 것**이다.

```java
// 자바 8이전의 코드
Comparator<Apple> byWeight = new Comparator<Apple>() {
    public int compare(Apple o1, Apple o2) {
        return o1.getWeight().compareTo(o2.getWeight());
    }
};

// 람다를 이용한 코드
Comparator<Apple> byWeight = 
    (Apple o1, Apple o2) -> o1.getWeight().compareTo(o2.getWeight());
```

위 예제를 보면 람다를 이용하면 코드가 훨씬 간결해진 것을 확인할 수 있다. 사과 두 개의 무게를 비교하는데 필요한 코드(compare의 바디)를 직접 전달하는 것처럼 코드를 전달할 수 있다.

람다 표현식은 세 부분(파라미터, 화살표, 바디)으로 이루어진다.

- 파라미터 리스트
- 화살표: 람다의 파라미터 리스트와 바디를 구분
- 람다 바디

```java
(Apple o1, Apple o2) // 람다 파라미터
-> // 화살표
o1.getWeight().compareTo(o2.getWeight()); // 바디
```

자바 8에서 지원하는 다섯 가지 람다 표현식 예제는 아래와 같다.

```java
(String s) -> s.length() // 1번

(Apple a) -> a.getWeight() > 150 // 2번

(int x, int y) -> { // 3번
    System.out.println("Result: ");
    System.out.println(x + y);
}

() -> 42 // 4번

(Apple o1, Apple o2) -> o1.getWeight().compareTo(o2.getWeight()); // 5번
```

1. String 형식의 파라미터 하나를 가지며 int를 반환. 람다 표현식에는 **return이 함축되어 있으므로 return 문을 명시하지 않아도 된다**.
2. Apple 형식의 파라미터 하나를 가지며 boolean을 반환. **return이 함축되어 있음.**
3. int 형식의 파라미터 두 개를 가지며 return값이 없다. **람다 표현식은 void 리턴이 가능하며, 여러 행의 문장을 포함할 수 있다.**
4. 파라미터가 없으면 int 42를 반환한다. **파라미터가 없을 수 있다.**
5. Apple 형식의 파라미터 두개를 가지며 int(두 파라미터 비교)를 반환.

```java
// 표현식 스타일(expression style): (파라미터 리스트) -> 표현식
(parameters) -> expression

// 블록 스타일(block style): (파라미터 리스트) -> 구문
(parameters) -> { statements; }
```

간단한 람다 표현식 사용 예제이다.

```java
(List<String> list) -> list.isEmpty() // 불리언 표현식

() -> new Apple(10) // 객체 생성

(Apple a) -> { // 객체에서 소비
    System.out.println(a.getWeight());
}

(String s) -> s.length() // 객체에서 선택/추출

(int a, int b) -> a * b // 두 값을 조합
```

<br>

# 3.2 어디에, 어떻게 람다를 사용할까?

- **함수형 인터페이스라는 문맥에서 람다 표현식을 사용할 수 있다.**

## 3.2.1 함수형 인터페이스

- 함수형 인터페이스: 정확히 하나의 추상 메서드를 지정하는 인터페이스.
  - 자바 API의 함수형 인터페이스로 Comparator, Runnable 등이 있다.
  - 디폴트 메서드를 포함할 수 있다. 많은 디폴트 메서드가 있더라도 추상 메서드가 오직 하나면 함수형 인터페이스다.
    - 디폴트 메서드: 인터페이스의 메서드를 구현하지 않은 클래스를 고려해서 기본 구현을 제공하는 바디를 포함하는 메서드

```java
public interface Comparator<T> {
    int compare(T o1, T o2);
}

public interface Runnable {
    void run();
}
```

람다 표현식으로 **함수형 인터페이스의 추상 메서드 구현을 직접 전달**할 수 있으므로 **전체 표현식을 함수형 인터페이스의 인스턴스로 취급**(함수형 인터페이스를 구현한 클래스의 인스턴스)할 수 있다.

```java
public static void process(Runnable r) {
    r.run();
}

Runnable r1 = () -> System.out.println("Hello World 1"); // 람다 사용

process(r1)
process(() -> System.out.println("Hello World 2")); // 람다 표현식으로 직접 전달
```

## 3.2.2 함수 디스크립터

- **함수 디스크립터(function descriptor)**: 람다 표현식(함수형 인터페이스의 추상 메서드)의 시그니처를 서술하는 메서드
  - 함수형 인터페이스의 추상 메서드 시그니처는 람다 표현식의 시그니처를 가리킨다.
  - Ex: Runnable 인터페이스의 추상 메서드 run()은 인수와 반환값이 없으므로 Runnable 인터페이스는 인수와 반환값이 없는 시그니처로 생각할 수 있다.
  - `() -> void`: 파라미터 리스트가 없으며 void를 반환하는 함수
  - `(Apple, Apple) -> int`: 두 개의 Apple을 인수로 받아 int를 반환하는 함수

람다 표현식의 유형성은 컴파일러가 확인한다. 일단 람다 표현식은 **변수에 할당**하거나 **함수형 인터페이스를 인수로 받는 메서드로 전달**할 수 있으며, **함수형 인터페이스의 추상 메서드와 같은 시그니처**를 갖는다는 사실을 기억!!

- 자바 언어 명세에는 void를 반환하는 메서드 호출과 관련한 특별한 규칙을 정하고 있기 때문에, 한 개의 void 메서드를 호출은 중괄호{ }로 감쌀 필요가 없다.

```java
process(() -> System.out.println("Hello World 1"));

process(() -> { System.out.println("Hello World 2"); });
```

> - Q: 왜 함수형 인터페이스를 인수로 받는 메서드에만 람다 표현식을 사용?
> - A: 자바에 함수 형식을 추가하는 방법도 대안으로 고려했지만 언어를 더 복잡하게 만들지 않는 방법을 선택함. 대부분의 자바 프로그래머가 하나의 추상 메서드를 갖는 인터페이스에 이미 익숙함.

```java
// 1번
execute(() -> {});
public void execute(Runnable r) {
    r.run();
}

// 2번
public Callable<String> fetch() {
    return () -> "Tricky Example :)";
}

// 3번
Predicate<Apple> p = (Apple a) -> a.getWeight();
```

- 1번: 유효한 람다식, () → {}의 시그니처는 () → void이며 람다의 바디가 비어있어 아무런 일도 일어나지 않음
- 2번: 유효한 람다식, 2번 메서드의 시그니처는 () → String이며 유효한 람다식이다.
- 3번: 유효하지 않은 람다식, (Apple) → Integer이므로 test 메서드의 시그니처인 (Apple) → boolean과 일치하지 않는다.

> `@FunctionalInterface`: 함수형 인터페이스임을 가리키는 어노테이션, 함수형 인터페이스가 아닌 인터페이스에 있으면 컴파일러가 에러를 발생시킴.

<br>

# 3.3 람다 활용: 실행 어라운드 패턴

- 실행 어라운드 패턴(execute arround pattern): 자원을 처리하는 코드가 **설정(setup)**과 **정리(cleanup)** 두 과정이 둘러싸는 형태를 갖는 패턴
  - (초기화/준비 코드) (작업) (정리/마무리 코드): 작업을 설정(초기화/준비)과 정리(정리/마무리)가 둘러쌈

```java
// try-with-resource구문 사용, 자원을 명시적으로 닫을 필요가 없어 간결한 코드를 작성하는데 도움
public String processFile() throws IOException {
    try (BufferedReader br = 
            new BufferedReader(new FileReader("data.txt"))) {
        return br.readLine(); // 실제 필요한 작업을 하는 행
    }
}
```

## 3.3.1 1단계: 동작 파라미터화를 기억하라

위 processFile()은 파일에서 한 번에 한 줄만 읽을 수 있다. 한 번에 두 줄을 읽거나 사용빈출이 높은 단어를 반환하려면 **동작 파라미터화**가 필요하다. 기존의 설정, 정리 과정은 재사용하고 processFile()만 **다른 동작을 수행하도록 명령할 수 있게 동작 파라미터화**해야 한다.

`BufferedReader`를 이용해 다른 동작을 수행할 수 있도록 메서드로 동작을 전달해야 한다. 두 줄을 읽게 하려면`BufferedReader`를 인수로 받아 String을 반환하는 람다가 필요하다.

```java
String twoLine = processFile((BufferedReader br) ->
                        br.readLine() + br.readLine());
```

## 3.3.2 2단계: 함수형 인터페이스를 이용해서 동작 전달

람다 표현식은 함수형 인터페이스를 인수로 받는 메서드에서 사용된다.

`BufferedReader -> String`과 `IOException`을 던질 수 있는 시그니처와 일치하는 함수형 인터페이스가 필요하다.

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}

// 정의한 인터페이스를 processFile()의 인수로 전달할 수 있음
public String processFile(BufferedReaderProcessor p) throws IOException {
    ...
}
```

## 3.3.3 3단계: 동작 실행

BufferedReaderProcessor에 정의된 process()의 시그니처 `BufferedReader -> String`와 같은 람다를 전달할 수 있다. 람다 표현식으로 **추상 메서드 구현을 직접 전달**할 수 있으며, **함수형 인터페이스의 인스턴스로 전달된 코드와 같은 방식으로 처리**되어 해당 인터페이스 객체의 메서드를 호출할 수 있다.

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = 
            new BufferedReader(new FileReader("data.txt"))) {
        return p.process(br); // BufferedReader 객체 처리
    }
}
```

## 3.3.4 4단계: 람다 전달

람다를 이용해 다양한 동작을 processFile()에 전달할 수 있다.

```java
// 한 줄을 읽어오는 코드
String oneLine = processFile((BufferedReader br) -> br.readLine());

// 두 줄을 읽어오는 코드
String twoLine = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

<br>

# 3.4 함수형 인터페이스 사용

함수형 인터페이스는 **오직 하나의 추상 메서드**를 지정한다. 추상 메서드는 람다 표현식의 시그니처를 묘사하고, 이 시그니처를 **함수 디스크립터**라고 한다. 다양한 람다식을 사용하려면 공통의 함수 디스크립터를 기술한 함수형 인터페이스의 집합이 필요하다. 자바 API는 Comparable, Runnable, Callable등의 다양한 인터페이스를 제공하고 있다.

자바 8은 `java.util.function` 패키지로 여러 새로운 함수형 인터페이스를 제공한다. 대표적으로 Predicate, Consumer, Function 인터페이스가 있다.

## 3.4.1 Predicate

- `java.util.function.Predicate<T>`: test() 추상 메서드를 정의하고 있는 인터페이스
  - test(): 제네릭 형식 T의 객체를 인수로 받아 boolean을 반환
  - T 형식의 객체를 사용하는 boolean 표현식이 필요한 상황에서 사용

```java
@FunctionalInterface
interface Predicate<T> {
    boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> results = new ArrayList<>();

    for (T t : list) {
        if (p.test(t)) {
            results.add(t);
        }
    }
    return results;
}

Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> listOfStrings = new ArrayList<>(List.of("Hi", "Hello", "World"));

List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate); // [Hi, Hello, World]
```

## 3.4.2 Consumer

- `java.util.function.Consumer<T>`: accept() 추상 메서드를 정의하고 있는 인터페이스
  - accept(): 제네릭 형식 T를 인수로 받아서 void를 반환
  - T 형식의 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때 사용

```java
@FunctionalInterface
interface Consumer<T> {
    void accept(T t);
}

public class ConsumerTest {
    public static <T> void forEach(List<T> list, Consumer<T> c) {
        for (T t : list) {
            c.accept(t);
        }
    }

    public static void main(String[] args) {
        forEach(
                Arrays.asList(1, 2, 3, 4, 5), (Integer i) -> System.out.println(i)
        );
    }
}
```

## 3.4.3 Function

- `java.util.function.Function<T, R>`: apply() 추상 메서드를 정의하고 있는 인터페이스
  - apply(): 제네릭 형식 T를 인수로 받아서 제네릭 형식 R 객체를 반환
  - 입력을 출력으로 매핑하는 람다를 정의할 때 활용 가능

```java
// String 리스트를 인수로 받앙 각 문자열의 길이를 반환하는 예제
@FunctionalInterface
interface Function<T, R> {
    R apply(T t);
}

public static <T, R> List<R> map(List<T> list, Function<T, R> f) {
    List<R> results = new ArrayList<>();

    for (T t : list) {
        results.add(f.apply(t));
    }
    return results;
}

List<Integer> length = map(
        Arrays.asList("Modern", "Java", "In", "Action"),
        (String s) -> s.length()
);
```

### 기본형 특화

자바의 모든 형식은 참조형(reference type) 아니면 기본형(primitive type)에 해당한다. 하지만 지네릭 파라미터에는 참조형만을 사용할 수 있기 때문에, 자바는 기본형을 참조형으로 변환하는 기능을 제공한다.

- **박싱(boxing)**: 기본형을 참조형으로 변환하는 기능
- **언박싱(unboxing)**: 참조형을 기본형으로 변환하는 기능
- **오토박싱(autoboxing)**: 박싱과 언박싱이 자동으로 이루어지는 기능

하지만 **박싱과정은 비용이 소모**된다. 박싱한 값은 기본형을 감싸는 Wrapper며 힙 메모리에 저장된다. 따라서 박싱한 값은 메모리를 더 소비하며 기본형을 가져올 때도 메모리를 탐색하는 과정이 필요하다.

- 자바 8에서는 기본형을 입출력으로 사용하는 상황에서 오토박싱을 피할 수 있도록 함수형 인터페이스를 제공한다. 일반적으로 특정 형식을 입력받는 함수형 인터페이스 이름 앞에는 DoublePredicate, IntConsumer, LongFunction처럼 형식명이 붙는다.

```java
@FunctionalInterface
public interface IntPredicate {
    boolean test(int t);
}

IntPredicate evenNumbers = (int i) -> i % 2 == 0;
evenNumbers.test(1000); // true(박싱 없음)

IntPredicate oddNumbers = (int i) -> i % 2 != 0;
evenNumbers.test(1000); // false(박싱)
```

> **함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않는다**. 즉, 예외를 던지는 람다 표현식을 만들려면 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의하거나 람다를 try-catch로 감싸야 한다.

<br>

# 3.5 형식 검사, 형식 추론, 제약

람다로 함수형 인터페이스의 인스턴스를 만들 수 있다. 람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는지의 정보가 포함되어 있지 않아, 람다의 실제 형식을 파악해야 한다.

## 3.5.1 형식 검사

람다가 사용되는 콘텍스트(context)를 이용해 람다의 형식을 추론할 수 있다.

- **대상 형식(target type)**: 콘텍스트(람다가 전달될 파라미터나 람다가 할당되는 변수 등)에서 기대되는 람다 표현식의 형식

```java
List<Apple> heavierThan150g = filter(inventory, (Apple a) -> a.getWeight() > 150);
// Apple을 인수로 받아 boolean을 반환하는 유요한 코드
```

위 코드의 형식 확인 과정은 아래와 같다.

1. filter 메서드의 선언을 확인
2. filter 메서드는 두 번째 파라미터로 Predicate<Apple> 형식(대상 형식)을 기대
3. Predicate<Apple>은 test라는 한 개의 추상 메서드를 정의하는 함수형 인터페이스
4. test 메서드는 Apple을 받아 boolean을 반환하는 함수 디스크립터를 묘사
5. filter 메서드로 전달된 인수는 이와 같은 요구사항을 만족해야 함.

람다 표현식이 예외를 던질 수 있다면 추상 메서드도 같은 예외를 던질 수 있도록 throws로 선언해야 한다.

## 3.5.2 같은 람다, 다른 함수형 인터페이스

**대상 형식(target type)**이라는 특징 때문에 같은 람다 표현식이어도 호환되는 추상 메서드를 가진 **다른 함수형 인터페이스로 사용**될 수 있다.

- 하나의 람다 표현식을 다양한 함수형 인터페이스에 사용할 수 있다.
  - 어떤 메소드의 시그니처가 사용되어야 하는지를 **명시적으로 구분하도록 람다를 캐스트**할 수 있다.

```java
Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;

Comparator<Apple> c1 =
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
ToIntBiFunction<Apple, Apple> c2 = 
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

// 람다 캐스트
execute((Action) () -> {System.out.println("Lambda cast")});
```

- 다이아몬드 연산자: 자바 7에서도 다이아몬드 연산자 <>로 콘텍스트에 따른 제네릭 형식을 추론할 수 있다는 사실을 확인할 수 있다.

```java
List<String> listOfStrings = new ArrayList<>();
List<Integer> listOfIntegers = new ArrayList<>();
```

- 특별한 void 호환 규칙: 람다의 바디에 일반 표현식이 있으면 void를 반환하는 함수 디스크립터와 호환된다(파라미터 리스트도 호환되어야 함). List의 add 메서드는 boolean을 반환하지만 유효한 코드가 된다.

```java
// Predicate는 boolean, Consumer는 void 반환값을 갖는다.
Predicate<String> p = s -> list.add(s);
Consumer<String> c = s -> list.add(s);
```

## 3.5.3 형식 추론

- **대상 형식(target type)을 이용해 컴파일러는 람다의 시그니처도 추론할 수 있다.**

컴파일러가 람다 표현식의 파라미터 형식에 접근할 수 있어 람다 문법에서 이를 생략할 수 있다. 즉, **자바 컴파일러는 람다 파라미터 형식을 추론할 수 있다**. (형식 추론 대상 파라미터가 하나면 괄호를 생략할 수 있다.)

```java
// apple의 타입을 명시적으로 지정하지 않았지만 컴파일러가 추론
List<Apple> greenApples = 
    filter(inventory, apple -> "GREEN".equals(apple.getColor()));

Comparator<Apple> c =
    (a1, a2) -> a1.getWeight().compareTo(a2.getWeight()); // a1, a2의 형식 추론
```

상황에 따라 가독성을 높이기 위해 명시적으로 형식을 포함해도 되고 형식을 생략해도 된다.

## 3.5.4 지역 변수 사용

람다 표현식에서는 익명 함수가 하는 것처럼 **자유 변수**를 활용할 수 있다. 이와 같은 동작을 **람다 캡처링**이라 한다.

- **자유 변수(free variable)**: 파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수
  - 약간의 제약이 존재
- **람다 캡처링(capturing lambda)**: 자유 변수를 활용하는 동작

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
// OK, final로 선언되지 않았지만 변경되지 않아 실질적으로 final처럼 취급
```

람다는 인스턴스 변수와 정적 변수를 자유롭게 캡처할 수 있지만 약간의 제약이 있다.

- 지역 변수는 명시적으로 `final`로 선언되어 있어야 하거나 실질적으로 `final`로 선언된 변수와 똑같이 사용되어야 한다. 즉, 람다 표현식은 한 번만 할당할 수 있는 지역 변수를 캡처할 수 있다.
  - 지역 변수는 스택에 위치해 자신을 정의한 스레드와 생존을 같이 해야 한다.

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber); // ERROR
portNumber = 4331;
```

지역 변수에 이런 제약이 존재하는 이유는 아래와 같다.

- 인스턴스 변수는 힙에 저장, 지역 변수는 스택에 위치
  - 인스턴스 변수는 **스레드가 공유하는 힙에 존재**하기 때문에 특별한 제약이 없다.
- 람다에서 지역 변수에 바로 접근할 수 있다는 가정하에 람다가 스레드에서 실행되면, 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려 할 수 있다.
- 자바에서는 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공하기 때문에 값이 바뀌지 않으려면 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생김.

> 클로저(closure): 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스
>
> - Q: 람다가 클로저의 정의에 부합한가?
> - A: **No**, 람다는 메서드의 인수로 전달되 수 있으며 자신의 외부 영역의 변수에 접근할 수 있다. 하지만 람다가 정의된 메서드의 지역 변수의 값은 바꿀 수 없다. 덕분에 람다는 변수가 아닌 **값에 국한되어 어떤 동작을 수행**한다는 사실을 확인할 수 있다. 만약 **가변 지역 변수를 새로운 스레드에서 캡처할 수 있다면 안전하지 않은 동작을 수행할 가능성이 생긴다.**