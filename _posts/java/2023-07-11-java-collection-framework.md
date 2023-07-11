---
title: "[자바] Collection Framework"
categories: [자바]
tag: ["자바", "Java"]
toc: true
toc_sticky: true
author_profile: false
sidebar:
    nav: "counts"
---

> [자바 [JAVA] - 자바 컬렉션 프레임워크 (Java Collections Framework)](https://st-lab.tistory.com/142)
>
> [Collections in Java](https://www.javatpoint.com/collections-in-java)

<br>

# 📌 자료구조 이해

자료구조는 대표적으로 선형 자료구조(Linear Data Structure)와 비선형 자료구조(Nonlinear Data Structure)로 구분할 수 있다.

- 선형 자료구조: 데이터가 일렬로 연결된 형태의 자료구조
  - 대표적인 선형 자료구조: List, Queue, Deque
- 비선형 자료구조: 선형 자료구조와 반대로 각 데이터가 여러 개의 요소와 연결 된 형태의 자료구조
  - 대표적인 비선형 자료구조: Graph, Tree
- 기타 자료구조: 선형, 비선형에 포함되지 않는 자료구조
  - 대표적인 기타 자료구조: Set

Java Collection Framework는 위 자료구조들을 표준화, 정형화시켜 쉽게 자료구조를 다루도록 도와주는 프레임워크다.

<br>

# 📌 Collection Framework

컬렉션 프레임워크는 다수의 데이터(Data Group)를 효과적으로 처리할 수 있는 표준화된 방법을 제공하는 인터페이스다.

즉, 데이터를 저장하는 자료 구조와 데이터를 처리하는 알고리즘을 구조화해 클래스로 구현해 놓은 프레임워크다.

다수의 데이터를 다루는 데 필요한 다양하고 풍부한 클래스들을 제공하고, 인터페이스와 다형성을 이용한 객체지향적 설계를 통해 표준화되어 있기 때문에 익히기 편리하고, 재사용성이 높은 코드를 작성할 수 있다는 장점이 있다.

`Collection`인터페이스는 `Iterable`인터페이스를 상속 받아 정의되어 있다.

- `Iterable`인터페이스를 상속 받아 Enhanced For-Loop를 사용할 수 있다

<center><img src="/assets/images/java/collection/1.png"></center>

> Map 인터페이스는 다른 형태로 컬렉션을 다루기 때문에 같은 상속계층도에 포함되지 않는다.

`Collection`인터페이스의 대표적인 기능을 먼저 살펴본다. 아래의 메서드 외에도 다양한 기능을 지원한다.

```java
public interface Collection<E> extends Iterable<E> {

	int size(); // 컬렉션의 데이터 수 반환

	boolean isEmpty(); // 컬렉션에 요소가 없는 경우 true 

	boolean contains(Object o); // 지정된 요소가 있는 경우 true

	Object[] toArray(); // 컬렉션의 요소 타입에 맞는 배열을 반환

	boolean add(E e); // 컬렉션에 요소 추가

	boolean remove(Object o); // 컬렉션의 요소 삭제

	boolean equals(Object o); // 지정된 객체와 컬렉션의 동등성 판단

	int hashCode(); // 컬렉션의 해시 값 반환

}
```

<br>

# 📌 Methods of Collection interface

- ```java
  boolean add(Object o)
  ```

  - 지정된 객체를 Collection에 추가

- ```java
  boolean addAll(Collection c)
  ```

  -  지정된 Collection를 Collection에 추가

- ```java
  void clear()
  ```

  - Collection의 모든 객체를 삭제

- ```java
  boolean contains(Object o)
  ```

  - 지정된 객체가 Collection에 포함되어 있는지 확인, 포함되어 있으면 `true`를 반환

- ```java
  boolean containsAll(Collection c)
  ```

  - 지정된 Collection이 Collection에 포함되어 있는지 확인, 포함되어 있으면 `true`를 반환

- ```java
  boolean equals(Object o)
  ```

  - 지정된 객체가 Collection가 동일한치 비교, 동일하면 `true`를 반환

- ```java
  int hasCode()
  ```

  - Collection의 hash code 값을 반환

- ```java
  boolean isEmpty()
  ```

  - Collection이 비어있는지 확인, 비어있는 경우 `true`를 반환

- ```java
  Iterator iterator()
  ```

  - Collection의 `Iterator`(반복자)를 반환

- ```java
  boolean remove(Object o)
  ```

  - 지정된 객체를 Collection에서 삭제

- ```java
  boolean removeAll(Collection c)
  ```

  - 지정된 Collection의 모든 값을 Collection에서 삭제

- ```java
  boolean retainAll(Collection c)
  ```

  - 지정된 Collection의 값들만 남기고 다른 값들을 Collection에서 삭제
  - 메서드 호출 후 Collection에 변화가 있으면 `true`를 반환

- ```java
  int size()
  ```

  - Collection에 저장된 객체의 개수를 반환

- ```java
  Object[] toArray()
  ```

  - Collection에 저장된 객체를 객체배열로 반환

- ```java
  Object[] toArray(Object[] a)
  ```

  - 지정된 배열에 Collection의 값들을 저장해서 반환


## Methods of Collection: Java 8 이후

Java 8이후 default method와 Stream이 제공되면서 몇 가지 메서드가 추가되었다.

- ```java
  default boolean removeIf(Predicate<? super E> filter)
  ```

  - 지정한 Predicate를 만족하는 모든 데이터를 삭제

- ```java
  default Stream<E> parallelStream()
  ```

  - Collection을 소스로 사용하는 병렬 스트림을 반환

- ```java
  default Stream<E> stream()
  ```

  - Collection을 소스로 사용하는 스트림을 반환

- ```java
  default Spliterator<E> spliterator()
  ```

  - Collection의 지정된 요소에 대한 Spliterator를 생성

- ```java
  default <T> T[] toArray(IntFunction<T[]> generator)
  ```

  - 제공된 generator 함수를 사용하여 반환된 배열을 할당하여 Collection의 모든 요소를 포함하는 배열을 반환