---
title: "[자료구조] 선형 자료구조: 연결 리스트(LinkedList)"
categories: [DataStructure]
tag: ["DataStructure"]
toc: true
toc_sticky: true
author_profile: false
use_math: true
sidebar:
    nav: "docs"
---

# 📌 연결 리스트(Linked List)?

배열(Array)의 문제점은 크기에 있다. 배열을 너무 크게 선언해 남는 메모리 공간이 넘쳐날 수 있고, 너무 작게 선언해 원하는 만큼 데이터를 담지 못할수 있다.

연결리스트의 핵심 아이디어는 **노드(Node)**이다. 노드는 포인터인 `next`와 노드에 넣는 데이터를 가리키는 포인터 `data`, 총 두 가지 정보가 들어있다. 각각의 포인터는 메모리의 어떤 위치를 가리킨다. `next` 포인터는 다음 노드의 위치를 가리키기 때문에 `next`라고 부른다. `data`포인터는 데이터를 가리키기 때문에 `data`라고 부른다.

연결 리스트는 `head`라는 이름의 포인터에서 시작한다. `head`는 리스트의 첫 번째 노드를 가리키고, 힙에서는 이 연결 리스트의 `head`만 알고 있기 때문에 `head.next`, `head.data`등으로 노드의 내용을 찾는다.
`head.next.next.next.~`처럼 chaining코드를 사용해 노드를 탐색할 수 있지만, 연결 리스트의 길이가 매우 길 경우 임시 포인터를 사용해 탐색하는 방법을 사용한다.

<br>

# 📌 노드와 크기

```java
public class LinkedList<E> implements List<E> {
    class Node<E> { // inner class로 외부에서 직접적인 접근이 불가능
        E data;
        Node<E> next; // 다음 노드를 가리키기 때문에 타입이 Node

        public Node(E obj) {
            data = obj;
            next = null;
        }
    }

    private Node<E> head;
    private Node<E> tail;
    private int currentSize; // 노드가 하나 생성될 때 마다 +1

    public LinkedList() {
        head = null;
        tail = null;
        currentSize = 0;
    }
		...
}

```

`currentSize`를 선언한 이유는 연결 리스트의 크기를 빠르게 얻기 위해서이다. 노드의 개수를 직접 셀 경우, 요소가 n개이면 n번 세야한다. 따라서 하나씩 세는 것의 시간 복잡도는 $\theta(n)$이다. 하지만 변수를 선언하고 리스트에 노드가 추가될 때 마다 변수의 값을 늘려주면 리스트의 크기를 바로 알 수 있다. 이 때 시간 복잡도는 $O(1)$이다.

<br>

# 📌 경계 조건 (Boundary Conditions)

어떤 자료구조든 신경 써야할 경계 조건이 5가지 존재한다.

1. **자료구조가 비어있는 경우**

   만약 자료구조의 요소를 제거했을 때 비어있는 경우가 되거나, 빈 자료구조에 요소를 추가했을 때는 평소와 어떻게 다른가?
   연결 리스트의 경우 비어있는 연결 리스트의 `head`는 `null`을 가리키고 있어야 한다.

2. **자료구조에 단 하나의 요소가 들어있는 경우**

   요소가 하나만 있을 때 제거하게 되면 빈 자료구조가 된다. 그러면 경계 조건 1번을 고려해야 한다.

3. **자료구조의 첫 번째 요소를 제거하거나 추가할 경우**

   자료구조에 첫 번째 요소를 추가할 경우 `head`를 주의해야 한다. 이 `head`를 어떻게 처리할지 생각해 봐야 한다.

4. **자료구조의 마지막 요소를 제거하거나 추가할 경우**

   자료구조의 마지막 요소를 제거하거나 추가할 경우 `tail`를 주의해야 한다. 이 `tail`를 어떻게 처리할지 생각해 봐야 한다.

5. **자료구조의 중간 부분을 처리하는 경우**

<br>

# 📌 addFirst()

연결 리스트의 생성자를 호출하면 가장 먼저 진행되는 것이 `head = null;` 이다. 새로운 요소를 추가할 때 노드 클래스 객체가 한개 생성되고, 이 노드 클래스는 `next`와 `data`를 가지고 있다.

`head`는 생성된 노드 객체 A의 `next`를 가리키게 되고 연결 리스트에 요소가 하나 추가 된다. 여기서 새로운 노드 객체 B를 생성해 연결 리스트의 첫 번째 요소로 추가하면 어떻게 될까? 단순히 `head`가 새로 생성된 B의 `next`를 가리키게 되면, 아무것도 노드 A를 가리키지 않기 때문에  **Garbage Collector**에 의해  노드가 사라지게 된다.

이러한 문제를 방지하기 위한 방법은 다음과 같다.

1. 새로운 노드 B를 생성한다.
2. 새로 생성된 노드 B의 `next`가 현재 `head`를 가리키도록 한다.
3. `head`가 새로운 노드 B를 가리키도록 한다.

이러면 기존의 노드가 삭제되는 경우를 방지할 수 있다. 이 방법은 새로운 요소를 추가하기 위해 뒷부분을 살펴볼 필요가 없어 $O(1)$의 시간 복잡도를 가지게 된다.

```java
public void addFirst(E obj) {
    Node<E> node = new Node<E>(obj);
    node.next = head; // 새로운 노드가 기존의 첫 번째 노드를 가리킨다
    head = node; // head는 새로운 노드를 가리킨다  
    tail = node; // tail은 새로운 노드를 가리킨다
}
```

위 메서드의 순서를 지키지 않으면 **Garbage Collector**에 의해 노드가 사라질 수 있다.

<br>

# 📌 addLast()

총 3개의 요소가 저장되어 있는 연결 리스트 마지막에 새로운 요소를 추가하려면 어떻게 해야하는가? head의 정보를 이용해 `head.next.next.next = node`로 새로운 요소를 추가하게 되면 코드가 매우 길어지고, 새로운 요소를 추가할 때 확장성이 떨어지게 된다.

임시 포인터 `tmp`를 이용해 `tmp.next = null`인 노드를 찾아야 한다. 조건을 만족하는 노드 다음에 새로운 노드를 추가하면 된다.

```java
public void addLast(E obj) {
    Node<E> node = new Node<E>(obj);
    Node<E> tmp = head;

    while (tmp.next != null) {
        tmp = tmp.next;
    }
    tmp.next = node;
}
```

위 코드는 경계 조건 1인 비어있는 연결 리스트에서 문제가 발생한다. 비어있는 연결 리스트에서 `NullPointerException`이 발생하게 된다. 비어있는 연결 리스트에서 `tmp = head = null`이기 때문이다. 예외 방지를 위해 먼저 연결 리스트가 비어있는지 확인해야 한다.

```java
public void addLast(E obj) {
    Node<E> node = new Node<E>(obj);
    // 비어있는 연결 리스트인지 확인
    if (head == null) {
        head = node;
        currentSize++;
        return;
    }

    Node<E> tmp = head;

    while (tmp.next != null) {
        tmp = tmp.next;
    }
    tmp.next = node;
    currentSize;
}
```

위 while문은 마지막 요소를 찾기 위해 $O(n)$의 시간 복잡도를 가지게 된다. 더 빠른 탐색을 위해 `tail`이라는 포인터를 추가한다. `tail`은 마지막 요소를 가리키게 된다.

```java
public void addLast(E obj) {
    Node<E> node = new Node<E>(obj);
    // 비어있는 연결 리스트인지 확인
    if (head == null) {
        head = node;
        tail = node;
        currentSize++;
        return;
    }

    tail.next = node;
    tail = node;
    currentSize++;
}
```

`tail` 포인터를 사용하게 되면 연결 리스트의 끝에 요소를 추가하는 작업은 $O(1)$의 시간 복잡도를 가지게 된다

<br>

# 📌 removeFirst()

연결 리스트의 첫 번째 요소를 삭제하기 위해서는 어떻게 해야하는가? head가 두 번째 요소를 가리키게 하면, 첫 번째 노드는 아무것도 가리키고 있지 않아 가비지 컬렉션이 일어난다. 즉, **Garbage Collector**에 의해 버려지게 된다.

```java
head = head.next;
```

먼저 **경계 조건1인 비어있는 리스트**를 생각해 보자. 
비어있는 연결 리스트에서 `head = null`이기 때문에`head = head.next;`는 `NullPointerException`이 발생한다. `head`가 `null`이기 때문에 `next`값을 가지고 있지 않기 때문이다. 
**경계 조건1 을 준수**하기 위해 비어있는 연결리스트의 **첫 번째 요소 삭제는 null을 반환**한다.

```java
public E removeFirst() {
    if (head == null) {
        return null;
    }

    // data 객체를 반환하기 위한 임시변수
    E tmp = head.data;
}
```

다음 경계 **조건2인 요소가 한 개만 있을 때**를 생각해 보자. 여기서 `head`와 `tail`은 첫 번째 요소를 가리키고 있다. 요소를 제거하면 `head`와 `tail`모두 `null`을 가리키고 있어야 한다.

그렇다면 요소가 한 개만 있는지 어떻게 알 수 있는가? 총 3가지 방법이 존재한다. 예제에서는 3번 방식을 채택했다.

1. `if (head.next == null)`
2. `if (currentSize = 1)`
3. `if (head == tail)`

```java
public E removeFirst() {
    if (head == null) {
        return null;
    }

    // data 객체를 반환하기 위한 임시변수
    E tmp = head.data;
    if (head == tail) { // 요소가 한 개인 경우
        head = null;
        tail = null;
    } else { // 요소가 두개 이상인 경우
        head = head.next;
    }
    currentSize--;

    return tmp;
}
```

<br>

# 📌 removeLast()

연결 리스트의 마지막 요소를 삭제하기 위해서는 어떻게 해야하는가? 마지막 요소를 가리키는 포인터 `tail`을 뒤에서 두 번째 요소를 가리키도록 만든다. 그럼 마지막 요소의 이전 요소는 어떻게 찾아야 하는가? `head`를 이용해 리스트 맨 앞부터 차례대로 찾아야 한다. 이 방법은 리스트의 크기가 100, 200과 같이 클 때는 매우 비효율적이다.

두 가지 해결책이 존재한다.

1. `currentSize`를 이용하는 방법.

   `currentSize-1`은 뒤에서 두 번째 요소를 가리킨다.

2. 임시 포인터 두개를 이용하는 방법이다.

   첫 포인터는 `current`이고, 이 포인터는 `head`를 가리킨다. 두 번째 포인터는 `previous`이고, `null`을 가리키고 있다. `previous` → `current` → `current.next`를 리스트의 끝까지 반복한다.
   `if (current == tail)` or `current.next == null`이면 current가 마지막 요소에 도달했다는 것을 알 수 있다. `current.next == null`은 `NullPointerException`이 발생할 수 있으니 첫 번째 방법을 활용하는 것이 좋다.

**경계 조건1 비어있는 연결 리스트**에선 `removeFirst()`와 같이 `null`을 반환해주면 된다.

**경계 조건2 요소가 한개**일 때는 `removeFirst()`와 `removeLast()`는 똑같은 메서드이다.

**경계 조건3 첫 번째 요소를 제거**할 때는 **경계 조건2**와 똑같은 상황이다.

```java
public E removeLast() {
    if (head == null) {
        return null;
    }

    if (head == tail) {
        // head = tail = null; 을 직접 작성해도 좋으나 미리 작성한 메서드를 활용
        return removeFirst();
    }

    Node<E> current = head;
    Node<E> previous = null;

    while (current != tail) {
        previous = current;
        current = current.next;
    }

    previous.next = null;
    tail = previous;
    currentSize--;

    return current.data;
}
```

<br>

# 📌 remove와 find

## remove(E obj)

무언가를 제거하려면 제거하고자 하는 요소의 위치를 알아야 한다. 이를 위해 Comparable인터페이스를 사용한다.

```java
if (((Comparable<E>)obj).compareTo(current.data) == null) {
        ...
    }
    ...
}
```

> `compareTo()`는 같은 객체일 때 `0`을 반환한다.

`removeLast()`와 같이 current와 previous 포인터가 필요하다.

**경계 조건1 비어있는 연결 리스트**에는 제거할 요소가 존재하지 않기 때문에 `null`을 반환한다.

**경계 조건2 요소가 한개**일 때는 `removeFirst()`를 사용하면 된다.

**경계 조건3 첫 번째 요소**를 제거할 때도 `removeFirst()`를 사용하면 된다.

**경계 조건4 마지막 요소**를 제거할 때는 `removeLast()`를 사용하면 된다.

**경계 조건5 중간 요소**를 제거할 때를 생각해 봐야 한다.

```java
public E remove(E obj) {
    Node<E> current = head;
    Node<E> previous = null;

    while (current != null) {
        if (((Comparable<E>obj).compareTo(current.data) == null) {
            if (current == head) { // 경계 조건 2, 3
                return removeFirst();
            }
            if (current == tail) { // 경계 조건 4
                return removeLast();
            }

            currentSize--;
            previous.next = current.next;

            return current.data; // 리스트에 존재했던 객체 반환
        }
        // 경계 조건 5
        previous = current;
        current = current.next;
    }

    return null; // 경계 조건 1
}
```

`remove()`는 객체를 한 개 받고 객체를 반환하는 메서드이다. 왜 둘 다 같은 타입인가?

같은 타입을 반환하지만 **매개변수로 들어온 객체를 반환하는 것이 아니다**. 매개변수로 들어온 객체가 아닌, **리스트에 존재했던 객체를 반환**하는 것이다. 리스트에 존재했던 객체를 반환해야 해당 객체와 관련된 다른 정보(데이터)를 알 수 있다.

## find(E obj)

객체가 리스트에 존재하는지 확인할 때는 그저 리스트를 순회하며 객체의 존재 여부만 따지면 된다. 경계 조건을 충족하는지 따질 필요가 없다는 뜻이다. 객체가 존재하면 `true`, 존재하지 않으면 `false`를 반환하면 된다.

```java
public boolean contains(E obj) {
    Node<E> current = head;

    while (current != null) {
        if (((Comparable<E>obj).compareTo(current.data) == null) {
            return true; // 리스트에 존재했던 객체 반환
        }
        current = current.next;
    }

    return false;
}
```

<br>

# 📌 peek 메소드

`peek` 메소드들은 하나의 요소를 살펴보기 위한 메소드이다. 읽기 전용이지 자료 구조의 정보를 수정(제거)하면 안된다.

첫 번째 요소의 데이터를 읽어오는 `peekFirst()`를 살펴보자. 먼저 리스트가 비어 있는지 확인해야 한다. 확인하지 않는다면 `NullPointerException`이 발생한다.

```java
public E peekFirst() {

    if (head == null) {
        return null;
    }

    return head.data;
}
```

마지막 요소의 데이터를 읽어오는 `peekLast()`는 임시 포인터 tmp를 이용한다. 마지막 요소를 확인하기 위해서는 tmp를 이용해 리스트를 순회해야 한다.

```java
// 1번
while (tmp.next != null)

// 2번
while (tmp != null)
```

리스트를 순회하는 두 반복문의 차이는 무엇인가?

- 1번: 마지막 노드에서 `tmp.next == null` 이 되므로 tmp는 마지막 노드에서 멈추게 된다.
- 2번: 마지막 노드가 가리키는 `null`에서 `tmp == null`이 되므로 마지막 노드가 가리키는 `null`에서 멈추게 된다.

1번 방법을 사용해 연결 리스트를 순회해야 한다.

1번 방법을 사용하는 `peekLiast()`는 n개의 요소를 모두 탐색하므로 $O(n)$ 시간 복잡도를 가진다. 좀 더 효율적인 구현을 위해 tail포인터를 이용한다.

```java
public E peekLast() {

    if (tail == null) {
        return null;
    }

    return tail.data;
}
```

tail포인터를 사용하면 $O(1)$ 시간 복잡도를 가지게 된다.



<br>

> 참고 URL
>
> > [네이버 부스트 코스: 자바로 구현하고 배우는 자료구조](https://www.boostcourse.org/cs204/joinLectures/145114)