---
title: "[모던 자바 인 액션] 06. 스트림으로 데이터 수집"
categories: [Modern Java in Action]
tag: ["Java"]
toc: true
toc_sticky: true
author_profile: false
sidebar:
    nav: "docs"
---

<br>

**스트림은 데이터 집합을 멋지게 처리하는 게으른 반복자**라 생각하면 된다. 중간 연산과 최종 연산이 존재하며, 중간 연산은 여러 연산을 연결해 스트림 파이프라인을 구성하며 스트림을 소비하지 않는다. 반면 최종 연산은 스트림을 소비해 최종 결과를 도출한다.

이전 까지는 최종 연산 collect의 toList를 이용해 리스트로만 결과를 도출 했지만, collect도 reduce와 같이 다양한 요소 누적 방식을 인수로 받아서 최종 결과를 도출할 수 있다. 다양한 **요소 누적 방식은 Collector 인터페이스에 정의**되어 있다.

> 컬렉션(Collection), 컬렉터(Collector), collect를 헷갈리지 않도록 주의한다.

# 6.1 컬렉터란?

명령형 프로그래밍으로 작성한 코드를 함수형 프로그래밍으로 변경하면 눈에 띄게 코드양이 줄어드는 것을 확인할 수 있다. 함수형 프로그래밍은 **무엇을 도출 해야하는지 직접 명시**하고 있어 쉽게 파악이 가능하다.(가독성 증가)

**Collector 인터페이스 구현은 스트림의 요소를 어떤 식으로 도출할지 지정한다.**

- `toList()`: 각 요소를 리스트로 반환
- `groupingBy()`: 각 key bucket과 key bucket에 대응하는 요소를 value로 포함하는 Map을 반환

명령형 코드는 다수준 그룹화에서 다중 루프와 조건문이 추가되어 가독성과 유지보수성이 크게 떨어지지만 함수형 프로그래밍은 간단하게 작성할 수 있다.

## 6.1.1 고급 리듀싱 기능을 수행하는 컬렉터

함수형 API의 장점은 높은 수준의 **조합성과 재사용성**이다. 컬렉터(Collector)는 collect로 결과를 수집하는 과정을 간단하면서 유연하게 정의가 가능하다. collect가 호출되면 스트림의 요소에 **컬렉터로 파라미터화된 리듀싱 연산이 수행**된다. 이 연산을 이용해 스트림의 각 요소를 방문하면서 **컬렉터가 작업을 처리**한다.

연산과정은 아래와 같다.

1. 스트림의 **각 요소를 탐색**한다.
2. 변환 함수에 따라 스트림의 **요소를 추출**한다.
3. collect에서 구현한 메서드에 따라 **리듀싱 연산이 수행**된다. (지정한 자료구조에 값이 누적된다,)

함수를 요소로 변환할 때는 컬렉터(Collector)를 적용하며 최종 결과를 저장하는 자료구조 값에 값을 누적한다. **Collector 인터페이스의 메서드를 어떻게 구현하느냐에 따라 어떤 리듀싱 연산이 수행될지 결정**된다.

따라서 **Collectors 유틸리티 클래스는 자주 사용하는 컬렉터 인스턴스를 정적 팩토리 메서드로 제공**한다.

```java
List<Transaction> transanctions = transactionStream.collect(Collectors.toList());
// Collectors 유틸리티 클래스가 요소를 List로 반환하는 정적 팩토리 메서드 toList()를 제공한다.
```

## 6.1.2 미리 정의된 컬렉터

Collectors 유틸리티 클래스의 정적 팩토리 메서드는 크게 세 가지로 구분할 수 있다.

1. **스트림 요소를 하나의 값으로 리듀스(reduce)하고 요약(summarize)**

   총합, 평균 등의 다양한 계산을 수행할 때 사용

2. **요소 그룹화(grouping)**

   다수준으로 그룹화 하거나 각각의 서브그룹에 리듀싱 연산(컬렉터를 조합)을 적용할 때 사용

3. **요소 분할(partitioning)**

   프레디케이트(Predicate)를 그룹화 함수로 사용

<br>

# 6.2 리듀싱과 요약

컬렉터(`Stream.collect` 메서드의 인수)로 **스트림의 모든 항목을 하나의 결과로** 합칠 수 있다.

## 6.2.1 스트림값에서 최대값과 최소값 검색

- `Collectors.maxBy`: 스트림의 최대값 계산 메서드
  - 요소를 비교하는 데 사용할 Comparator를 인수로 받는다.
- `Collectors.minBy`: 스트림의 최소값 계산 메서드
  - 요소를 비교하는 데 사용할 Comparator를 인수로 받는다.

```java
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);

Optional<Dish> mostCalorieDish = menu.stream()
                .collect(Collectors.maxBy(dishCaloriesComparator));

Optional<Dish> leastCalorieDish = menu.stream()
                .collect(Collectors.minBy(dishCaloriesComparator));
```

## 6.2.2 요약 연산

스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 연산에 리듀싱 기능이 자주 사용된다. 이러한 연산을 **요약(summarization) 연산**이라 한다.

- `Collectors.summingInt`: 객체를 int로 매핑하는 함수를 인수로 받아 int로 매핑한 컬렉터를 반환
  - collect 메서드로 전달되어 요약 작업을 수행
  - 누적자에 값이 더해짐
- `Collectors.summingLong`, `Collectors.summingDouble`: summingInt와 같은 방식으로 동작하며 long, double 형식의 데이터로 요약
- `Collectors.averagingInt`, `Collectors.averagingLong`, `Collectors.averagingDouble`: 평균값 계산 요약 메서드
- `Collectors.summarizingInt`: 요소 수, 합계, 평균, 최대값, 최소값 등을 계산하는 팩토리 메서드
  - IntSummaryStatistics 클래스로 모든 정보가 수집됨.
  - summarizingLong, summarizingDouble 이 존재하며 각각 LongSummaryStatistics, DoubleSummaryStatistics로 정보가 수집된다.

```java
int totalCalories = menu.stream()
                .collect(Collectors.summingInt(Dish::getCalories));

double averageCalories = menu.stream()
                .collect(Collectors.averagingDouble(Dish::getCalories));

IntSummaryStatistics menuStatistics = menu.stream()
                .collect(Collectors.summarizingInt(Dish::getCalories));
```

## 6.2.3 문자열 연결

- `Collectors.joining()`: 스트림 각 객체에 `toString()`을 호출해 얻은 문자열을 하나의 문자열로 반환
  - 내부적으로 StringBuilder를 이용해 처리
  - 요소 사이에 구분 문자열(delimiter)을 넣을 수 있도록 오버로드됨

```java
String menus = menu.stream()
                .map(Dish::getName)
                .collect(Collectors.joining());
// porkbeefchickenfrench friesriceseason fruitpizzaprawnssalmon

String menusWithDelimiter = menu.stream()
                .map(Dish::getName)
                .collect(Collectors.joining(", "));
// pork, beef, chicken, french fries, rice, season fruit, pizza, prawns, salmon
```

## 6.2.4 범용 리듀싱 요약 연산

최대값, 최소값 연산과 요약 연산등 대부분의 컬렉터는 `Collectors.reducing` 팩토리 메서드로 정의가 가능하다. 특화된 컬렉터를 사용하는 이유는 단지 가독성을 높이기 위해서다.

```java
// 특화된 컬렉터 사용
int totalCalories = menu.stream()
                .collect(Collectors.summingInt(Dish::getCalories));

// reducing 사용
totalCalories = menu.stream()
                    .collect(Collectors.reducing(
                                0, Dish::getCalories, (i, j) -> i + j));
```

reducing 메서드는 세 개의 인수를 받는다.

- 첫 번째 인수: 리듀싱 연산의 시작값이거나 인수가 없을 때 반환값(보통 0을 사용)
- 두 번째 인수: 변환 함수
- 세 번째 인수: BinaryOperator

```java
Optional<Dish> mostCalorieDish = menu.stream()
                .collect(Collectors.reducing(
                        (d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2
                ));
```

- reducing 메서드가 한 개의 인수를 갖는 경우
  - 첫 번째 인수: 스트림의 **첫 번째 요소**를 시작값으로 지정
  - 두 번째 인수: 자신을 그대로 반환하는 **항등 함수(identity function)**를 인수로 받음

```java
// counting 컬렉터도 reudcing 메서드를 이용해서 구현할 수 있다.
public static <T> Collector<T, ?, Long> counting() {
    return Collectors.reducing(0L, e -> 1L, Long::sum);
}
```

- Generic 와일드카드 `?`: 컬렉터의 누적자 형식이 알려지지 않았음을, 즉 누적자의 형식이 자유로움을 의미

함수형 프로그래밍에서는 **하나의 연산을 다양한 방식**으로 해결할 수 있다. 스트림 인터페이스에서 직접 제공하는 메서드를 이용하면 간결하게 해결이 가능하지만, **컬렉터를 사용하면 코드가 좀 더 복잡한 대신 재사용성과 높은 수준의 추상화와 일반화**를 얻을 수 있다.

<br>

# 6.3 그룹화

자바 8의 함수형을 이용하면 가독성 있는 코드로 데이터 집합을 하나 이상의 특성으로 분류해 그룹화할 수 있다.

- `Collectors.groupingBy`: 이 함수를 기준으로 스트림이 그룹화됨
  - **분류 함수(classification function)**라고 부름

```java
Map<Dish.Type, List<Dish>> dishesByType = menu.stream()
                .collect(Collectors.groupingBy(Dish::getType));
// 각 키(Dish.Type)에 대응하는 스트림의 모든 항목 리스트를 값으로 갖는 맵이 반환
// {MEAT=[pork, beef, chicken], OTHER=[french fries, rice, season fruit, pizza], FISH=[prawns, salmon]}
```

## 6.3.1 그룹화된 요소 조작

그룹화 한 다음에는 각 결과 그룹의 요소를 조작하는 연산이 필요하다. 예제로 음식 Type별로 그룹화한 결과에 500 칼로리가 넘는 요리만 필터한다고 가정한다.

```java
Map<Dish.Type, List<Dish>> caloricDishesByType = menu.stream()
                .filter(dish -> dish.getCalories() > 500)
                .collect(Collectors.groupingBy(Dish::getType));
```

위 코드의 문제점은 필터의 프레디케이트에 **만족하는 값이 그룹에 없다면 그룹 자체가 사라진다(key 자체가 사라짐)**는 점이다. 이를 위해 groupingBy 메서드가 두 번째 인수를 갖도록 오버로드해 문제를 해결한다.

즉, **Collector 안으로 필터 프레디케이트가 이동**된다.

- `Collectors.filtering`: Collectors 클래스의 또 다른 정적 팩토리 메서드로 프레디케이트를 인수로 받는 메서드, 이 프레디케이트로 각 그룹의 요소와 필터링 된 요소를 재그룹화 시킴.
  - 프레디케이트를 만족하지 못하는 그룹은 빈 리스트를 가짐.

```java
Map<Dish.Type, List<Dish>> caloricDishesByType = menu.stream()
                .collect(Collectors.groupingBy(Dish::getType,
                        Collectors.filtering(dish -> dish.getCalories() > 500, Collectors.toList())));
```

- `Collectors.mapping()`: 매핑 함수와 각 항목에 적용한 함수를 모으는 데 사용하는 또 다른 컬렉터를 인수로 받아 그룹화된 항목을 조작하는 메서드
- `Collectors.flatMapping()`: 매핑 함수와 각 항목에 적용한 함수를 모으는 데 사용하는 또 다른 컬렉터를 인수로 받아 그룹화된 항목을 조작하는 메서드, 한 수준으로 평면화(중복 제거)

```java
Map<Dish.Type, List<String>> namesByType = menu.stream()
                .collect(Collectors.groupingBy(Dish::getType, Collectors.mapping(Dish::getName, Collectors.toList())));
// {MEAT=[pork, beef, chicken], OTHER=[french fries, rice, season fruit, pizza], FISH=[prawns, salmon]}

public static final Map<String, List<String>> dishTags = new HashMap<>();
static {
    dishTags.put("pork", asList("greasy", "salty"));
    dishTags.put("beef", asList("salty", "roasted"));
    dishTags.put("chicken", asList("fried", "crisp"));
    dishTags.put("french fries", asList("greasy", "fried"));
    dishTags.put("rice", asList("light", "natural"));
    dishTags.put("season fruit", asList("fresh", "natural"));
    dishTags.put("pizza", asList("tasty", "salty"));
    dishTags.put("prawns", asList("tasty", "roasted"));
    dishTags.put("salmon", asList("delicious", "fresh"));
}

Map<Dish.Type, Set<String>> dishNamesByType = menu.stream()
                .collect(Collectors.groupingBy(Dish::getType,
                        Collectors.flatMapping(dish -> dishTags.get(dish.getName()).stream(),
                                Collectors.toSet())
                ));
// {MEAT=[salty, greasy, roasted, fried, crisp], OTHER=[salty, greasy, natural, light, tasty, fresh, fried], FISH=[roasted, tasty, fresh, delicious]}
```

## 6.3.2 다수준 그룹화

- `Collectors.groupingBy`: 항목을 다수준으로 그룹화 가능
  - 일반적인 분류 함수와 컬렉터를 인수로 받음.
  - **바깥쪽 groupingBy 메서드의 스트림 항목을 분류할 내부 groupingBy 메서드를 전달해 두 수준으로 스트림의 항목을 그룹화 할 수 있음.**

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = menu.stream()
                .collect(Collectors.groupingBy(Dish::getType, // 첫 번째 수준의 분류 함수, [FISH, MEAT, OTHER]
                                Collectors.groupingBy(dish -> { // 두 번째 수준의 분류 함수, [DIET, NORMAL, FAT]
                                    if (dish.getCalories() <= 400) {
                                        return CaloricLevel.DIET;
                                    } else if (dish.getCalories() <= 700) {
                                        return CaloricLevel.NORMAL;
                                    } else {
                                        return CaloricLevel.FAT;
                                    }
                                })
                        )
                );
```

다수준 그룹화 연산은 **다양한 수준으로 확장할 수 있다**. 즉, n수준 그룹화의 결과는 n수준 트리 구조로 표현되는 n수준 맵이 된다.

## 6.3.3 서브그룹으로 데이터 수집

다수준 그룹화 연산을 위해 groupingBy로 넘겨주는 컬렉터의 형식은 제한이 없다.

```java
// 두 번째 인수로 counting 컬렉터를 전달해 요리의 수를 종류별로 계산할 수 있다.
Map<Dish.Type, Long> typesCount = menu.stream()
                .collect(Collectors.groupingBy(Dish::getType, Collectors.counting()));
// {MEAT=3, OTHER=4, FISH=2}
```

- `groupingBy(f)`는 `groupingBy(f, Collectors.toList())`의 축약형이다.

### 컬렉터 결과를 다른 형식에 적용

- `Collectors.collectingAndThen`: 적용할 컬렉터와 변환 함수를 인수로 받아 다른 컬렉터를 반환
  - 컬렉터가 반환한 결과를 다른 형식으로 활용
  - 반환되는 컬렉터는 기존 컬렉터의 Wrapper 역할

```java
Map<Type, Dish> mostCaloricByType = menu.stream()
                .collect(Collectors.groupingBy(Dish::getType, // 분류 함수
                        Collectors.collectingAndThen(
                                Collectors.maxBy(Comparator.comparingInt(Dish::getCalories)), // 감싸진 컬렉터
                                Optional::get // 반환 함수
                        )
                ));
// {MEAT=pork, OTHER=pizza, FISH=salmon}
```

- groupingBy와 자주 사용되는 메서드에는 mapping, flatMapping 메서드가 존재한다.

```java
Map<Type, HashSet<CaloricLevel>> caloricLevelsByType = menu.stream()
                .collect(Collectors.groupingBy(Dish::getType, Collectors.mapping(dish -> {
                            if (dish.getCalories() <= 400) {
                                return CaloricLevel.DIET;
                            } else if (dish.getCalories() <= 700) {
                                return CaloricLevel.NORMAL;
                            } else {
                                return CaloricLevel.FAT;
                            }
                        }, Collectors.toCollection(HashSet::new)) // 원하는 형식으로 결과를 제어
                ));
// {MEAT=[DIET, FAT, NORMAL], OTHER=[DIET, NORMAL], FISH=[DIET, NORMAL]}
```

<br>

# 6.4 분할

- **분할(Partition):** **특수한 종류의 그룹화**
- `Collectors.partitioningBy`: **분할 함수(partitioning function)**라 불리는 프레디케이트를 분류 함수로 사용하는 그룹화 기능을 제공하는 메서드
  - 프레디케이트가 Boolean을 반환하므로 key 타입은 Boolean으로 최대 두 개의 그룹(true or false)으로 분류된다.

```java
Map<Boolean, List<Dish>> partitionedMenu = menu.stream()
                .collect(partitioningBy(Dish::isVegetarian));
// {false=[pork, beef, chicken, prawns, salmon], true=[french fries, rice, season fruit, pizza]}

// 프레디케이트로 필터링한 다음 리스트로 반환해도 똑같은 결과를 얻을 수 있다.
List<Dish> forVegetarianDishes = partitionedMenu.get(true);
// [french fries, rice, season fruit, pizza]
```

## 6.4.1 분할의 장점

- 참(true), 거짓(false) 두 가지 요소의 스트림 리스트를 모두 유지한다는 것이 장점이다.
- 컬렉터를 두 번째 인수로 전달할 수 있도록 오버로드된 `partitioningBy`메서드도 존재한다.

```java
Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType = menu.stream()
                .collect(partitioningBy(Dish::isVegetarian, groupingBy(Dish::getType)));
// {false={FISH=[prawns, salmon], MEAT=[pork, beef, chicken]}, true={OTHER=[french fries, rice, season fruit, pizza]}}
```

## 6.4.2 숫자를 소수와 비소수로 분할하기

```java
public class PartitionPrimeNumber {

    public static boolean isPrime(int candidate) {
        return IntStream.rangeClosed(2, (int) Math.sqrt(candidate))
                .noneMatch(i -> candidate % i == 0);
    }

    public static Map<Boolean, List<Integer>> getPartitionPrimes(int n) {
        return IntStream.rangeClosed(2, n)
                .boxed()
                .collect(Collectors.partitioningBy(PartitionPrimeNumber::isPrime));
    }

    public static void main(String[] args) {
        List<Integer> primeNumbers = getPartitionPrimes(100).get(true);
        List<Integer> nonePrimeNumbers = getPartitionPrimes(100).get(false);
    }

}
```

<br>

# 6.5 Collector 인터페이스

- **Collector 인터페이스**: 리듀싱 연산(컬렉터)을 어떻게 구현할지 제공하는 메서드 집합으로 구성.

인터페이스기 때문에 직접 구현해서 더 효율적으로 문제를 해결하는 컬렉터를 만들 수 있다.

```java
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    Function<A, R> finisher();
    BinaryOperator<A> combiner();
    Set<Characteristics> characteristics();
}
```

- T: 수집될 스트림 항목의 제네릭 형식
- A: 누적자, 수집 과정에서 중간 결과를 누적하는 객체의 형식
- R: 수집 연산 결과 객체의 형식(거의 컬렉션 형식)

## 6.5.1 Collector 인터페이스의 메서드

- characteristics를 제외한 나머지 네 개의 메서드는 collect 메서드에서 실행하는 함수를 반환한다.
- characteristics는 메서드가 어떤 최적화를 이용해서 리듀싱 연산을 수행할 것인지 결정하도록 돕는 힌트 특성 집합을 제공한다.

### Supplier: 새로운 결과 컨테이너 생성

- `Supplier<A> supplier()`: 빈 결과로 이루어진 Supplier를 반환하는 메서드
  - 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수

```java
@Override
public Supplier<List<T>> supplier() {
    return ArrayList::new;
}
```

### accumulator: 결과 컨테이너에 요소 추가

- `BiConsumer<A, T> accumulator()`: 리듀싱 연산을 수행하는 함수를 반환
  - n 번째 요소를 탐색할 때 두 인수(누적자, n 번째 요소)를 함수에 적용
  - 함수의 반환값은 void, 요소를 탐색하면서 적용하는 함수에 의해 누적자 내부상태가 바뀌므로 값을 단정할 수 없음

```java
@Override
public BiConsumer<List<T>, T> accumulator() {
    return List::add;
}
```

### finisher: 최종 변환값을 결과 컨테이너로 적용

- `Function<A, R> finisher()`: 탐색을 끝내고 누적자 객체를 최종 결과로 변환하면서 누적 과정을 끝낼 때 호출할 함수를 반환
  - 누적자 객체 자체가 최종 결과인 상황도 존재, 이럴 때는 **항등 함수를 반환**

```java
@Override
public Function<List<T>, List<T>> finisher() {
    // return i -> i;
    return Function.identity();
}
```

### combiner: 두 결과 컨테이너 병합

- `BinaryOperator<A> combiner()`: 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이 결과를 어떻게 처리할지 정의

```java
@Override
public BinaryOperator<List<T>> combiner() {
    return (list1, list2) -> {
        list1.addAll(list2);
        return list1;
    };
}
```

- combiner를 이용하면 스트림의 리듀싱을 병렬로 수행할 수 있다.
  - 자바 7의 fork/join 프레임워크와 Spliterator를 사용

1. 스트림을 조건에 따라 재귀적으로 분할한다(일반적으로 프로세싱 코어의 개수를 초과하지 않게 나눈다).
2. 나뉜 모든 서브스트림에 리듀싱 연산을 적용한다.
3. combiner 메서드를 통해 분할된 모든 서브스트림의 결과를 합치면서 연산이 완료된다.

### characteristics

- `Set<Characteristics> characteristics()`: 컬렉터의 연산을 정의하는 Characteristics 형식의 불변 집합을 반환, 스트림을 병렬로 리듀스할 것인지 그리고 병렬로 리듀스 하면 어떤 최적화를 선택해야 할지 힌트를 제공
  - **UNORDERED**: 리듀싱 결과가 스트림의 요소 방문 순서나 누적 순서에 영향을 받지 않음.
  - **CONCURRENT**: 다중 스레드에서 accumulator를 동시에 호출하며 병렬 리듀싱을 수행할 수 있음. UNORDERED와 함께 설정하지 않으면 **정렬되어 있지 않은(의미 없는)상황에서만 병렬 리듀싱 수행.**
  - **IDENTITY_FINISH**: 항등함수를 사용하는 finisher 메서드를 생략할 수 있다. 최종 결과로 누적자 객체를 바로 사용. 누적자 A를 결과 R로 안전하게 형변환.

### 컬렉터 구현을 만들지 않고도 커스텀 수집 수행하기

- IDENTITY_FINISH 수집 연산에서는 새로 구현하지 않고도 결과를 얻을 수 있다. 스트림은 세 함수(발행, 누적, 병합)를 인수로 받는 collect 메서드를 오버로드해 구현이 가능하다.

```java
List<Dish> dishes = menu.stream()
                .collect(
                        ArrayList::new, // 발행
                        List::add, // 누적
                        List::addAll // 병합
                );
// [pork, beef, chicken, french fries, rice, season fruit, pizza, prawns, salmon]
```

<br>

# 6.6 Summary

- collect는 스트림의 요소를 요약 결과로 누적하는 다양한 방법을 인수로 갖는 최종 연산
- groupingBy()로 스트림의 요소를 그룹화하거나 partitioningBy()로 스트림의 요소를 분할할 수 있다.
- 컬렉터는 다수준의 그룹화, 분할, 리듀싱 연산에 적합하게 설계되어 있다.
- Collector 인터페이스에 정의된 메서드를 구현해 커스텀 컬렉터를 구현할 수 있다.