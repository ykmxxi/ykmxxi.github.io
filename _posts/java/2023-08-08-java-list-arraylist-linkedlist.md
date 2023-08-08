---
title: "[자바] List 인터페이스와 ArrayList, LinkedList"
categories: [자바]
tag: ["자바", "Java"]
toc: true
toc_sticky: true
author_profile: false
sidebar:
    nav: "counts"
---

> [List Interface in Java with Examples: GeeksforGeeks](https://www.geeksforgeeks.org/list-interface-java-examples/)
>
> [[Java] Collection 을 복사하는 여러 방법: hudi.blog](https://hudi.blog/ways-to-copy-collection/)

<br>

# 📌 List 인터페이스

- `List`는 어떤 순서가 있는 데이터의 집합이다.

`List` 인터페이스는 sequence라고도 하며 **중복을 허용**하면서 **저장순서를 유지**하는 컬렉션을 구현하는데 사용된다. `List` 인터페이스를 구현한 클래스들은 데이터가 삽입되는 위치를 정확하게 제어할 수 있으며, 사용자는 인덱스(index)로 데이터에 접근하고 검색할 수 있다.

<center><img src="/assets/images/java/listinterface/1.png"></center>

## 리스트(List) vs 배열(Array)

`List` 인터페이스와 비슷한 자료구조를 떠올리면 배열(`int[]`, `double[]`)이 존재한다. 두 자료구조는 비슷하면서도 조금씩 다른점을 갖고 있다.

배열은 연속적인 메모리에서 같은 종류의 요소들을 저장할 수 있는 **자료구조**이고, 리스트는 순서를 가지며 추가, 삭제, 탐색이 가능한 ADT(Abstract Data Type)다.

> ADT는 데이터의 추상화를 통해 특정 연산을 수행하는 기능과 그 연산의 규칙들을 정의한다. ADT는 데이터의 내부 구현을 감추고, 사용자에게는 데이터 타입의 인터페이스만을 제공하여 데이터를 조작할 수 있는 방법을 제공한다.

- **리스트와 배열의 공통점**
  - 동일한 특성의 데이터를 묶어 저장한다
  - 반복문내에 변수를 이용해 하나의 묶음 데이터들을 모두 접근할 수 있다
- **리스트와 배열의 차이점**
  - 배열은 자료구조이고, 리스트는 ADT이다
  - 배열은 크기(길이)를 변경할 수 없고 리스트의 크기(길이)는 가변적이다. 즉 배열은 길이를 정적 할당(static allocation)하고, 리스트는 길이를 동적 할당(dynamic allocation)한다.
  - 배열은 데이터들이 메모리에 연속적으로 나열되어 할당되고, 리스트는 데이터들의 주소를 참조해 나열한다(데이터들의 물리적 주소가 순차적이지 않다).
  - 배열은 데이터 삭제시 해당 공간(index)이 빈공간으로 남고, 리스트는 데이터 삭제시 빈 공간을 허용하지 않고 기존 데이터들의 위치를 변경한다

배열은 고정된 크기로 선언해야하기 때문에 크기 설정이 어렵다. 경우에 따라 메모리 낭비가 심해질 수 있고, 너무 작게 선언하면 주어진 데이터를 다 담지 못할 수 있다. 하지만 인덱스를 통한 접근 속도가 빠르다.

리스트는 크기를 가변 할당하기 때문에 메모리 관리가 편하고 데이터 삽입, 삭제에 용이하지만 적은양의 데이터만 쓸 경우 메모리가 커진다. 예를 들어 기본형 타입의 정수 배열은 4 Byte를 차지하지만 Integer를 사용한 리스트는 최소 16 Byte(헤더 + 원시 필드 + 패딩 + 객체 주소) 이상 차지한다.

## List가 제공하는 기능

`List`는 `Collection` 인터페이스에 명시된 기능 외에도 여러가지 기능을 제공한다.

- ```java
  boolean add(E e)
  ```

  - 리스트의 끝에 해당 객체를 추가

- ```java
  void add(int index, Object element)
  ```

  - 지정된 index에 객체를 추가

- ```java
  boolean addAll(int index, Collection c)
  ```

  - 지정된 index에 컬렉션의 모든 객체들을 추가

- ```java
  Object get(int index)
  ```

  - 지정된 index에 위치한 객체를 반환

- ```java
  int size()
  ```

  - 리스트의 크기(길이)를 반환

- ```java
  boolean contains(Object o)
  ```

  - 지정된 객체가 리스트에 존재하면 true를 반환

- ```java
  boolean isEmpty()
  ```

  - 리스트가 비어있으면 true를 반환

- ```java
  int indexOf(Object o)
  ```

  - 지정한 객체의 위치를 반환(첫 번째 요소 0번 부터 순방향으로 탐색)

- ```java
  int lastIndexOf(Object o)
  ```

  - 지정한 객체의 위치를 반환(마지막 요소 list.size() - 1 부터 역방향으로 탐색)

- ```java
  ListIterator.listIterator()
  ```

  - 리스트 객체에 접근할 수 있는 반복자를 반환(index = 0 부터 시작)

- ```java
  ListIterator.listIterator(int index)
  ```

  - 지정된 index 부터 시작해 리스트의 객체에 접근할 수 있는 반복자를 반환

- ```java
  Object remove(int index)
  ```

  - 지정된 index의 요소를 삭제하고 삭제된 객체를 반환

- ```java
  Object set(int index, Object element)
  ```

  - 지정된 Index에 지정한 객체를 저장

- ```java
  void sort(Comparator c)
  ```

  - 지정된 비교자 comparator를 이용해 리스트를 정렬

- ```java
  List subList(int fromIndex, int toIndex)
  ```

  - 지정된 범위 fromIndex 부터 toIndex 이전의 객체를 리스트 형태로 반환
  - fromIndex <= 범위 < toIndex

- ```java
  Object[] toArray()
  ```

  - 해당 리스트를 배열로 변환해 반환

## Unmodifiable List 편리하게 만들기

Unmodifiable List는 수정할 수 없는 읽기 전용 리스트다.

특징으로는 데이터 추가, 삭제, 변경이 불가능하고 `null`을 요소로 허용하지 않는다.

데이터 추가, 삭제, 변경시 `UnsupportedOperationException`이 발생하고, `null`이 포함되어 있으면 `NullPointerException`이 발생한다.

- ```java
  static <E> List<E> of(E... elements)
  ```

- ```java
  static <E> List<E> copyOf(Collection<? extends E> coll)
  ```

  - `copyOf()`와 `Collections.unmodifiableList()`의 차이는 원본 객체와의 관계에 있다. `copyOf()`는 원본 객체를 복사한 객체로 원본과 복사본은 아무런 관계가 없다. 하지만 `Collections.unmodifiableList()`는 내부적으로 원본 객체를 그대로 가지고 있어 원본이 수정되면 영향을 받는다

위 두개의 메서드로 생성한 List 인스턴스는 몇 가지 특징을 갖는다

- 수정할 수 없다(Unmodifiable)
  - 추가, 삭제, 변경시 `UnsupportedOperationException`발생
- null 요소를 허용하지 않는다
- 모든 요소가 직렬화 가능하면 직렬화 가능
- 리스트 객체들의 순서는 제공된 인수 또는 제공된 배열의 요소 순서와 동일

<br>

# 📌 ArrayList

- **ArrayList**: 크기가 가변적으로 변하는 선형리스트
  - **연속**적인 공간에 **순차적**으로 데이터를 저장

`List` 인터페이스를 구현한 java.util 패키지의 `ArrayList`는 **데이터의 저장순서가 유지**되고 **중복을 허용**한다는 특징을 갖는다. 일반적인 배열과 같은 순차 리스트이며 인덱스로 내부의 객체를 관리한다. 인덱스로 내부의 객체를 관리하기 때문에 인덱싱(indexing)이 가능하다.

즉, O(1) 상수 시간으로 데이터에 매우 빠르게 접근할 수 있다.

하지만 연속적인 공간에 순차적으로 데이터가 존재해 중간에 데이터를 추가하거나 삭제할 때 오래 걸린다.

> `ArrayList`는 기존 `Vector`를 개선한 자료구조로 구현원리와 기능적인 측면은 거의 비슷하다. `Vector`와 거의 유사하게 작동하는데 멀티 스레드 환경을 위한 동기화에서 차이점이 존재한다. `ArrayList`는 동기화 처리가 되어 있지 않은 대신 그만큼 처리 속도가 빠르다.

Object 배열을 이용해 데이터를 순차적으로 저장하기 때문에 모든 종류의 객체를 담을 수 있고, 저장 용량(capacity) 초과시 크기가 더 큰 새로운 배열을 생성해 기존 배열의 저장된 내용을 복사한 다음 객체를 저장하므로 크기에 관한 고민이 필요가 없다.

```java
import java.util.ArrayList;

ArrayList<String> list1 = new ArrayList<>(); // default capacity = 10
ArrayList<String> list2 = new ArrayList<>(20); // capacity = 20
ArrayList<String> list3 = new ArrayList<>(list1); // 다른 컬렉션의 데이터를 초기값으로 설정
```

<br>

# 📌 LinkedList

- **LinkedList**: 데이터 요소가 노드로 구성되어 연결된 리스트 형태의 자료 구조
  - **비 연속**적인 공간에 **순차적**으로 데이터를 저장
  - 각 노드는 데이터와 다음 노드를 가리키는 포인터(참조)를 포함
  - **노드(객체)** 끼리의 주소 포인터를 서로 가리키며 **링크(참조)**
  - 객체를 만들면 객체의 주소가 생기게 되는데, 노드마다 각기 객체의 주소를 서로 참조함으로서 연결 형태를 구성하는 것

`LinkedList`는 비 연속적인 공간에 데이터를 저장하기 때문에 ‘데이터의 추가/삭제’가 유리하다.

하지만 `ArrayList`에 비해 데이터 탐색 시간이 오래 걸린다. `LinkedList`는 데이터를 탐색할 때 앞 or 뒤에서 차례대로 탐색해 해당 값을 찾는다.

`LinkedList`는 주로 아래와 같은 경우 많이 사용한다

- 데이터의 삽입과 삭제가 빈번한 경우
  - `ArrayList`와 달리 데이터가 중간에 추가/삭제 되어도 데이터가 저장된 공간을 이동시키지 않고 노드의 참조를 변경하기만 하면된다
- 리스트의 크기가 상대적으로 작을 경우
- 순차적인 데이터 접근이 주로 필요한 경우

```java
import java.util.LinkedList;

LinkedList<String> list1 = new LinkedList<>();
LinkedList<String> list2 = new LinkedList<>(list1);
```

## 단방향 연결 리스트(Singly Linked-List)

다음 노드를 가리키는 포인터인 `next`만 가지고 있는 Linked-List. 하지만 단일 연결 리스트는 현재 요소에서 이전 요소로 접근해야 할때 매우 부적합한 특징이 존재.

```java
class Node<E> {

	Node<E> next; // 다음 노드를 가리키는 포인터
	E data; // 데이터 저장 필드
}
```

## 양방향 연결 리스트(Doubly Linked-List)

단방향 연결 리스트에서 이전 노드를 가리키는 포인터인 `prev`가 추가된 형태의 Linked-List. 이동 방향이 양방향으로 가능해 역순으로도 검색이 가능하다.

```java
class Node<E> {

	Node<E> next; // 다음 노드를 가리키는 포인터
	Node<E> prev; // 이전 노드를 가리키는 포인터
	E data; // 데이터 저장 필드
}
```

> Java 컬렉션 프레임워크에 구현된 `LinkedList` 클래스는 doubly linked list로 구현 되어 있다.