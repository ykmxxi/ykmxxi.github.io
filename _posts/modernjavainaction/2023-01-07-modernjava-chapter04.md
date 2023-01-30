---
title: "[모던 자바 인 액션] 04. 스트림 소개"
categories: [Modern Java in Action]
tag: ["Java"]
author_profile: false
sidebar:
    nav: "counts"
---

<br>

거의 모든 자바 애플리케이션은 **컬렉션(Collections)**을 만들고 처리하는 과정을 포함한다. 컬렉션으로 데이터를 그룹화하고 처리할 수 있다. 하지만 완벽한 컬렉션 관련 연산을 지원하려면 많이 부족하다.

예를 들어 요리와 관련된 비즈니스 로직을 처리할 때 카테고리로 그룹화한다든가 가장 비싼 요리, 칼로리가 낮은 요리를 찾는 등의 연산이 포함된다. `SELECT name FROM dishes WHERE calorie < 400`이라는 문장은 칼로리가 낮은 음식을 선택하는 SQL 질의다. SQL 질의는 요리의 속성을 이용해 어떻게 필터링할 것인지는 구현할 필요가 없다. 단지 **기대하는 것이 무엇인지 직접 표현**할 수 있다. 즉, SQL에서는 **질의를 어떻게 구현해야 할지 명시할 필요가 없으며 구현은 자동으로 제공**된다. 컬렉션으로 이와 같은 기능을 만들 수 있을까?

또 요소가 많은 컬렉션을 처리할 때는 **멀티코어 아키텍처를 활용해 병렬로 컬렉션의 요소를 처리**해야 성능을 높일 수 있다. 하지만 병렬 처리 코드를 만드는 것은 복잡하고 어렵다. 병렬 처리를 쉽게 제공할 수 있을까?

위 두 질문의 답은 **스트림**이다.

# 4.1 스트림이란?

**스트림(Stream)**은 자바 8 API에 새로 추가된 기능으로 선언형(데이터를 처리하는 임시 구현 코드 대신 질의로 표현)으로 컬렉션 데이터를 처리할 수 있다. 또 멀티스레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리할 수 있다.

자바 8이전의 코드로 저칼로리 요리명을 반환하고, 칼로리를 기준으로 요리를 정렬하는 코드는 아래와 같다.

```java
    // 자바 7
    public static List<String> lowCaloricDishesJava7(List<Dish> menu) {
        // 가비지 변수를 사용(컨테이너 역할만 하는 중간 변수)
        List<Dish> lowCaloricDishes = new ArrayList<>();

        for (Dish dish : menu) {
            if (dish.getCalories() < 400) {
                lowCaloricDishes.add(dish);
            }
        }

        // 칼로리 기준 정렬
        Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
            @Override
            public int compare(Dish o1, Dish o2) {
                return Integer.compare(o1.getCalories(), o2.getCalories());
            }
        });

        List<String> lowCaloricDishesName = new ArrayList<>();
        for (Dish dish : lowCaloricDishes) {
            lowCaloricDishesName.add(dish.getName());
        }

        return lowCaloricDishesName;
    }
```

다음은 자바 8의 스트림을 이용한 코드이다. 자바 8에서는 가비지 변수(lowCaloricDishes)를 사용하지 않고 세부 구현은 라이브러리 내에서 모두 처리한다.

```java
    public static List<String> lowCaloricDishesJava8(List<Dish> menu) {
        return menu.parallelStream()
                .filter(d -> d.getCalories() < 400) // 400칼로리 이하 요리 선택
                .sorted(Comparator.comparing(Dish::getCalories)) // 칼로리로 요리 정렬
                .map(Dish::getName) // 요리명 추출
                .collect(Collectors.toList()); // 요리명을 리스트로 저장
    }
```

스트림의 새로운 기능이 소프트웨어공학적으로 다음의 다양한 이득을 준다.

- **선언형으로 코드를 구현**할 수 있다. 즉, 루프와 if 조건문 등의 제어 블록을 사용하지 않고 **동작의 수행을 지정**할 수 있다.
- 선언형 코드(스트림)와 동작 파라미터를 활용하면 **변하는 요구사항에 쉽게 대응**할 수 있다. 즉, 기존 코드를 복사 붙여넣기 방식을 사용하지 않고 **필터링을 통해 쉽게 구현**할 수 있다.
- 여러 빌딩 블록 연산(filter, sorted, map, collect)을 연결해서 복잡한 **데이터 처리 파이프라인**을 만들 수 있다.

filter, sorted, map, collect 같은 연산들은 **고수준 빌딩 블록**으로 이루어져 있어 특정 스레딩 모델에 제한되지 않고 자유롭게(단일, 병렬 모두) 어떤 상황에서든 사용할 수 있다. **스트림 API를 이용하면 데이터 처리 과정을 병렬화하면서 스레드와 락을 걱정할 필요가 없다.**

**스트림 API의 특징**을 세 가지로 요약하면 다음과 같다.

- **선언형**: 더 간결하고 가독성이 좋아진다.
- **조립할 수 있음**: 유연성이 좋아진다.
- **병렬화**: 성능이 좋아진다.

<br>

# 4.2 스트림 시작하기

**스트림(Stream)**: **데이터 처리 연산**을 지원하도록 **소스**에서 추출된 **연속된 요소**(Sequence of elements)

- **연속된 요소**: 스트림은 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공한다. filter, sorted, map처럼 표현 계산식이 주를 이루기 때문에 **스트림의 주제는 계산**이다.

> **컬렉션**은 자료구조이므로 시간과 공간의 복잡성과 관련된 요소 저장 및 접근 연산이 주를 이룬다. 즉, **컬렉션의 주제는 데이터**이다.

- **소스**: 스트림은 컬렉션, 배열, I/O 자원 등의 데이터 제공 **소스로부터 데이터를 소비**한다.
- **데이터 처리 연산**: 함수형 프로그래밍 언어의 연산과 데이터베이스와 비슷한 연산을(filter, map, reduce, find, match, sort 등) 통해 **데이터를 조작**한다. 또 스트림 연산은 **순차적 or 병렬로 실행**할 수 있다.

스트림의 두 가지 중요 특징.

- **파이프라이닝(Pipelining)**: 스트림 연산끼리 연결해 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환한다. 그 덕분에 **게으름(laziness)**, **쇼트서킷(short-circuiting)**같은 최적화가 가능하다. 연산 파이프라인은 데이터 소스에 적용하는 데이터베이스 질의와 같다.
- **내부 반복**: 반복자를 이용하는 컬렉션과 달리 스트림은 내부 반복을 지원한다.

```java
// 스트림 파이프라인 연산 예제
List<String> threeHighCaloricDishNames = menu.stream() // 메뉴의 스트림을 얻는다
            .filter(dish -> dish.getCalories() > 300) // 300 칼로리 초과 요리를 필터링
            .map(Dish::getName) // 요리명을 추출
            .limit(3) // 그 중 3개 선착순 제한
            .collect(Collectors.toList()); // 결과를 리스트로 저장
```

위 파이프라인 연산 예제의 흐름을 다음과 같다.

1. menu에 stream 메서드를 호출해 스트림을 얻는다. 여기서 **데이터 소스는 menu**이고 연속된 요소를 스트림에 제공한다.
2. filter, map, limit, collect로 이어지는 **데이터 처리 연산**을 적용한다. **collect를 제외한 모든 연산은 서로 파이프라인을 형성할 수 있도록 스트림을 반환**한다.
3. collect 연산으로 파이프라인을 처리해서 결과를 반환한다. collect를 호출하기 전까지는 데이터 소스에서 무엇도 선택되지 않으며 출력 결과도 없다. 즉, collect가 호출되기 전까지 메서드 호출이 저장되는 효과가 있다.

각 연산은 다음 작업을 수행한다.

- `filter`: 람다를 인수로 받아 스트림에서 특정 요소를 제외시킨다.
- `map`: 람다를 이용해서 한 요소를 다른 요소로 변환하거나 정보를 추출한다.
- `limit`: 정해진 개수 이상의 요소가 스트림에 저장되지 못하게 스트림 크기를 축소(truncate)한다.
- `collect`: 스트림을 다른 형식으로 변환한다.

스트림 라이브러리에서는 필터링, 추출, 축소 기능을 제공하므로 직접 기능을 구현할 필요가 없다. 결과적으로 스트림 API는 **파이프라인을 더 최적화할 수 있는 유연성을 제공**한다.

<br>

# 4.3 스트림과 컬렉션

컬렉션(Collections)과 스트림(Stream) 모두 **연속된** 요소 형식의 값을 저장하는 자료구조의 인터페이스를 제공한다. 여기서 **연속된(Sequenced)**은 순차적으로 값에 접근한다는 것을 의미한다.

데이터를 **언제 계산**하느냐가 컬렉션과 스트림의 가장 큰 차이다.

- 컬렉션: 현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 자료구조. 즉, 컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 한다.
  - 컬렉션은 적극적으로 생성된다(생산자 중심)
- 스트림: 요청할 때만 요소를 계산하는 고정된 자료구조. 즉, 요소를 추가하거나 제거할 수 없다. 스트림은 사용자가 요청하는 값만 스트림에서 추출한다는 것이 핵심이다.
  - 스트림은 생산자(producer)와 소비자(consumer) 관계를 형성한다.
  - 스트림은 게으르게 만들어진 컬렉션과 같다. 요청할 때만 값을 계산하기 때문이다. (소비자 중심)

## 4.3.1 딱 한 번만 탐색할 수 있다.

스트림도 반복자처럼 한 번만 탐색할 수 있다. 즉, **탐색된 스트림의 요소는 소비**된다. 한 번 탐색한 요소를 다시 탐색하려면 초기 데이터 소스에서 새로운 스트림을 만들어야 한다. 데이터 소스가 컬렉션이라면 문제가 없지만 입출력 채널이라면 소스를 반복 사용할 수 없어 새로운 스트림을 만들 수 없다.

```java
List<String> title = Arrays.asList("Modern", "Java", "In", "Action");
Stream<String> s = title.stream();
s.forEach(System.out::println); // 각 문자열 출력
s.forEach(System.out::println); // IllegalStateException 발생
```

- **스트림은 단 한 번만 소비할 수 있다!!!**

## 4.3.2 외부 반복과 내부 반복

- **외부 반복(external iteration)**: 컬렉션 인터페이스를 사용하기 위해 직접 요소를 반복함.
  - 명시적으로 컬렉션 항목을 하나씩 가져와서 처리한다.

```java
// for-each 루프를 이용하는 외부 반복
List<String> names = new ArrayList<>();
for (Dish dish : menu) {
    names.add(dish.getName());
}

// 반복자를 사용하는 외부 반복
List<String> names = new ArrayList<>();
Iterator<String> iterator = menu.iterator();
while (iterator.hasNext()) { // 명시적 반복
    Dish dish = iterator.next();
    names.add(dish.getName());
}
```

- **내부 반복(internal iteration)**: 스트림 라이브러리는 반복을 알아서 처리하고 결과를 어딘가에 저장.
  - 함수에 어떤 작업을 수행할지 지정하면 알아서 처리된다.

```java
List<String> names = menu.stream()
                        .map(Dish::getName)
                        .collect(Collectors.toList());
```

외부 반복보다 내부 반복이 더 좋은 두 가지 이유가 있다.

1. 작업을 투명하게 **병렬로 처리**한다.
2. 더 **최적화된 다양한 순서로 처리**할 수 있다.

스트림 라이브러리의 **내부 반복은 데이터 표현과 하드웨어를 활용한 병렬성 구현을 자동으로 선택**한다. 반면 **외부 반복에서는 병렬성을 스스로 관리**해야 한다(병렬성을 포기 or `synchronized`로 복잡한 처리).

스트림은 내부 반복을 사용하므로 **반복 과정을 프로그래머가 신경 쓰지 않아도 된다**. 하지만 반복을 숨겨주는 연산 리스트가 미리 정의되어 있어야 하고 자바 8은 이 연산(filter, map 등)을 제공해 준다. 이 연산들은 대부분 람다 표현식을 인수로 받으므로 **동작 파라미터화를 활용**할 수 있다.

<br>

# 4.4 스트림 연산

스트림 인터페이스의 연산을 크게 두 가지로 구분할 수 있다.

- **중간 연산(intermediate operation)**: 연결할 수 있는 스트림 연산(서로 연결되어 파이프라인을 형성)
- **최종 연산(terminal operation)**: 스트림을 닫는 연산

## 4.4.1 중간 연산

- **중간 연산(intermediate operation)**: 연결할 수 있는 스트림 연산(서로 연결되어 파이프라인을 형성)

![중간 연산](/assets/images/modernjavainaction/IntermediateOperation.png "중간 연산")

중간 연산은 **다른 스트림을 반환**한다. 따라서 여러 중간 연산을 연결해 질의를 만들 수 있다. 중간 연산의 중요한 특징은 단말 연산을 스트림 파이프라인에 실행하기 전까지 연산을 수행하지 않는다는 것이다. 즉, **게으르다(lazy).**

- 중간 연산을 합친 다음에 중간 연산을 최종 연산으로 한 번에 처리

게으른 특성 덕분에 몇 가지 **최적화 효과를 얻을 수 있다**.

- limit 연산 그리고 **쇼트서킷**으로 연산이 최적화 된다.

  - **쇼트서킷(short-circuit)**: 논리 연산에서 두 피연산자 중 어느 한쪽만 '참'이면은 우측 피연산자의 값은 평가하지 않고 바로 결과를 얻는 기법

- **루프 퓨전(loop fusion)**: 서로 다른 중간 연산이 한 과정으로 병합

## 4.4.2 최종 연산

- **최종 연산(terminal operation)**: 스트림을 닫는 연산, 스트림 파이프라인에서 스트림 이외의(List, Integer, void 등) 결과를 도출

![최종 연산](/assets/images/modernjavainaction/TerminalOperation.png "최종 연산")

## 4.4.3 스트림 이용하기

스트림 이용 과정은 다음과 같이 세 가지로 요약할 수 있다.

- 질의를 수행할 데이터 소스
- 스트림 파이프라인을 구성할 중간 연산 연결
- 스트림 파이프라인을 실행하고 결과를 만들 최종 연산

스트림 파이프라인의 개념은 빌더 패턴과 비슷하다.

- 빌더 패턴(builder pattern): 호출을 연결해서 설정을 만듦(중간 연산 연결), 그리고 준비된 설정에 build 메서드를 호출(최종 연산)

<br>

# 4.5 Summary

- 스트림은 소스에서 추출된 **연속 요소로 데이터 처리 연산을 지원**한다.
- 스트림은 내부 반복을 지원한다. 내부 반복은 filter, map, sorted 등의 연산으로 **반복을 추상화**한다.
- 스트림에는 중간 연산과 최종 연산이 있다. 중간 연산은 스트림을 반환해 파이프라인을 구성할 수 있지만 아무런 결과도 생성할 수 없다. 최종 연산은 스트림 파이프라인을 처리해서 스트림이 아닌 결과를 반환한다.
- 스트림의 요소는 요청할 때 **게으르게(lazy) 계산**된다.