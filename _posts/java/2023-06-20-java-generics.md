---
title: "[자바] 제네릭(Generics)"
categories: [자바]
tag: ["자바", "Java"]
toc: true
toc_sticky: true
author_profile: false
sidebar:
    nav: "counts"
---

> [자바 제네릭(Generics 기초 - Tecoble)](https://tecoble.techcourse.co.kr/post/2020-11-09-generics-basic/)
>
> [자바의 정석, 도우출판, 남궁성 저](https://www.yes24.com/Product/Goods/24259565)

<br>

# 📌 제네릭(Generics) 이란?

**제네릭**은 `JDK 1.5`부터 도입한 클래스 내부에서 사용할 데이터 타입을 외부에서 지정하는 기법이다.

다양한 타입의 객체들을 다루는 메서드나 컬렉션 클래스에 컴파일 시의 타입체크(compile-time type check)를 해주는 기능을 제공해 객체의 타입을 컴파일 시에 체크해 객체의 타입 안정성을 높이고 형변환의 번거로움이 줄어든다.

- `JDK 1.5`부터 버그를 줄이고 유형에 대한 추가 추상화 계층을 추가하기 위해 Java Generics를 도입

자바 사용자는 제네릭이 정확히 무엇인지 몰라도 `List`, `Map` 과 같은 자료형을 사용할 때 자연스럽게 제네릭을 사용하고 있다.

```java
public interface List<E> extends Collection<E> { // E: Element
  ...
}

public interface Map<K, V> { // K: key, V: value
  ...
}
```

제네릭은 클래스나 메소드에서 사용할 내부 데이터 타입을 컴파일 시에 미리 지정하는데, 이렇게 컴파일 시에 미리 타입 검사(type check)를 수행하면 2가지 장점을 가진다.

1. 클래스나 메소드 내부에서 사용되는 **객체의 타입 안정성**을 높일 수 있다
2. 반환값에 대한 **타입 변환 및 타입 검사**에 들어가는 비용을 줄일 수 있다

제네릭이 나오기 전 자바에서는 여러 타입을 사용하는 대부분의 클래스나 메소드에서 인수나 반환값으로 최고 조상인 `Object` 타입을 사용 했다. 하지만 이 경우 Object 객체를 다시 원하는 타입으로 형변환 해야하는 번거로움이 존재하고, 이때 오류가 발생할 수 있었다.

```java
int sum = 0;

// 제네릭 사용 X: Object로 타입 지정
List numbers = new ArrayList(List.of(1, 2, 3, 4, 5));
for (Object number : numbers) {
  sum += (int)number;
}

// 제네릭 사용 O: Integer로 타입 지정
List<Integer> numbers = new ArrayList<>(List.of(1, 2, 3, 4, 5));
for (Integer number : numbers) {
  sum += number;
}
```

제네릭을 사용하니 불필요한 형 변환 과정이 없어지고 코드가 좀 더 깔끔해 졌다. 또한 제네릭은 타입 안정성을 보장해주는데, 위 코드에서 `numbers`를 선언할 때 `int`형으로 형 변환할 수 없는 타입이 들어온다면 컴파일 에러가 발생하지 않고 런타임에 `ClassCastException`이 발생하게 된다. 컴파일 시에 에러를 잡지 못하면 컴파일 언어의 장점을 버리는 것과 같다.

```java
List numbers = new ArrayList(List.of("1", "2", "3", "4", "5"));
for (Object number : numbers) {
  sum += (int)number;
}
```



<center><img src="/assets/images/java/generics/1.png"></center>

<br>

# 📌 제네릭 사용

제네릭은 클래스와 메서드에 선언할 수 있다.

- 여기서 `<T>` 는 **타입 변수(type variable)**를 의미
  - 꼭 `T`를 입력해야 동작하는 것은 아니다. 하지만 타입 파라미터 컨벤션이 존재하고 좋은 코드를 위해 컨벤션을 지켜야 한다
  - E: Element
  - K: Key
  - N: Number
  - T: Type
  - V: Value
- `JDK1.7`부터 타입이 추정 가능한 경우 타입 생략이 가능해졌다

```java
class Box<T> {

	T item;

	public T getItem() {
		return item;
	}

	public void setItem(T item) {
		this.item = item;
	}

}

public class Main {

	public static void main(String[] args) {
		Box<String> stringBox = new Box<>(); // String 타입의 Box 객체 선언, 타입 추정

		stringBox.setItem("String");
		stringBox.setItem(new Object()); // ERROR, String 타입만 지정 가능

		String item = stringBox.getItem();
	}

}

```

<center><img src="/assets/images/java/generics/2.png"></center>

여기서 다이아몬드 연산자 안에 들어간 `String`을 매개변수화된 타입(parameterized type)이라 부른다.

<br>

# 📌 제네릭 사용 제한

- 제네릭은 static 멤버에 사용할 수 없다

제네릭은 인스턴스별로 다르게 동작하도록 만든 기능이기 때문에 객체별로 다른 타입을 지정하는 것이 적절하다. 각 객체마다 다르게 동작하도록 기능을 제공하는데 static 변수와 같이 모든 객체에 동일하게 동작해야하는 경우는 제네릭을 사용할 수 없다.

```java
static T item; // ERROR

static int compare(T t1, T t2) { // ERROR
  ...
}
```

- 제네릭 타입의 배열 생성은 허용되지 않는다

제네릭 배열 타입의 참조변수를 선언하는 것은 가능하지만, 배열을 생성할 수 없다. 그 이유는 `new` 연산자 때문인데, 컴파일 시점에 제네릭 타입이 정확히 무슨 타입인지 알아야 하지만 제네릭 타입의 배열은 어떤 타입인지 알 수 없기 때문이다. 같은 이유로 `instanceof` 연산자도 제네릭을 사용할 수 없다.

```java
T[] boxArray; // OK

T[] toArray {
  T[] tmpArray = new T[boxArray.length]; // ERROR
}
```

- 제네릭 클래스가 상속관계이고, 대입된 타입이 같으면 객체 생성이 가능하다
  - 대입된 타입이 상속관계에 있어도 일치하지 않으면 ERROR가 발생한다

```java
class Box<T> {
}

class FruitBox<T> extends Box<T> {
}

Box<String> sb = new Box<>(); // OK
Box<String> sbfb = new FruitBox<>(); // OK
FruitBox<String> fb = new Box<>(); // ERROR

Box<Object> appleBox = new Box<String>(); // ERROR
```

- 대입된 타입과 같은 타입의 객체만 추가할 수 있다

```java
class Fruit {
}

class Apple extends Fruit {
}

class Grape extends Fruit {
}

Box<Apple> appleBox = new Box<>();
appleBox.add(new Apple()); // OK
appleBox.add(new Grape()); // ERROR

Box<Fruit> fruitBox = new Box<>();
fruitBox.add(new Fruit()); // OK
fruitBox.add(new Apple()); // OK
```

<br>

# 📌 제한된 제네릭 클래스

`extends`키워드를 사용하면 타입 매개변수 `<T>` 에 지정할 수 있는 타입의 종류를 제한할 수 있다.

- 제네릭 타입에 `extends` 키워드를 사용하면, 특정 타입의 자손들만 대입할 수 있다

```java
class FruitBox<T extends Fruit> {
  // FruitBox 의 타입은 Fruit과 그 자손 타입들만 대입할 수 있다
}

FruitBox<Fruit> fruitBox; // OK
FruitBox<Apple> appleBox; // OK
FruitBox<Grape> grapeBox; // OK

FruitBox<String> stringBox; // ERROR
```

- 객체를 추가할 때도 자손 타입을 추가할 수 있다

```java
FruitBox<Fruit> fruitBox = new FruitBox<>();

fruitBox.add(new Fruit()); // OK
fruitBox.add(new Apple()); // OK
fruitBox.add(new Grape()); // OK

fruitBox.add("Fruit"); // ERROR
```

- 인터페이스의 경우에도 제약이 필요하면 `extends` 키워드를 사용하고, 클래스와 인터페이스를 동시에 상속받고 구현해야 한다면 엠퍼센트(`&`) 기호를 사용하면 된다

```java
interface Eatable {}

class FruitBox<T extends Fruit & Eatable> {}
```

<br>

# 📌 와일드 카드

```java
class Juicer {
  
  static Juice makeJuice(FruitBox<Fruit> box) {
    String tmp = "";
    for (Fruit fruit : box.getList()) {
      tmp += fruit + " ";
    }
		
    return new Juice(tmp);		
  }

  static Juice makeJuice(FruitBox<Apple> box) {
    String tmp = "";
    for (Fruit fruit : box.getList()) {
      tmp += fruit + " ";		
    }
		
    return new Juice(tmp);	
  }
  
}
```

`makeJuice()`메서드를 여러 가지 타입의 매개변수를 갖는 메서드로 만들어 사용하면 **컴파일 에러**가 발생한다. 그 이유는 **제네릭 타입이 다른 것만으로는 오버로딩이 성립되지 않기 때문**이다. 컴파일러는 컴파일할 때만 제네릭 타입을 사용하고 제거하기 때문에 컴파일러 입장에서는 위 두 메서드가 오버로딩이 아닌 중복 정의가 된다.

이런 문제를 해결하기 위해 고안된 것이 **와일드 카드**이다.

- 와일드 카드: 모든 타입을 대신할 수 있는 타입, 기호 `?` 로 표현한다
  - 상한과 하한을 제한할 수 있다
  - `<? extends T>`: 상한 제한, `T`와 그 자손 타입만 가능
  - `<? super T>`: 하한 제한, `T`와 그 조상 타입만 가능
  - `<?>`: 제한이 없는 와일드 카드로 모든 타입이 가능, `<? extends Object>` 와 동일

<br>

# 📌 제네릭 메서드

제네릭 타입을 클래스 뿐만 아니라 메서드에서도 사용 가능하다. 메서드의 선언부에 제네릭 타입이 선언된 메서드를 정의할 수 있는데 꼭 제네릭 클래스가 아닌 일반 클래스에서도 정의될 수 있다.

```java
static <T> void sort(List<T> list, Comparator<? super T> c) {}
```

`Collections` 클래스의 `sort` 메서드는 제네릭 메서드이다. `Comparator`의 타입 매개변수에 와일드 카드의 하한 제한을 이용해 중복된 메서드 정의를 하지 않고 정렬기능을 이용할 수 있다. 만약 `Fruit`의 자손인 `Apple`을 매개변수 `T`에 대입한다면, 조상인 `Fruit`과 최고 조상 `Object`가 타입 매개변수로 올 수 있다.

지네릭 클래스에 정의된 타입 매개변수와 지네릭 메서드에 정의된 타입 매개변수는 전혀 별개의 것이다.

- static 멤버에 타입 매개변수를 사용할 수 없지만 **static 메서드에 제네릭 타입을 선언하고 사용하는 것은 가능**하다. 그 이유는 메서드에 선언된 제네릭 타입은 지역 변수를 선언한 것과 비슷해, 이 타입 매개변수는 메서드 내에서만 사용될 것이므로 메서드가 static 이건 아니건 상관이 없기 때문이다

```java
class FruitBox<T> {
	static T item; // ERROR
	static int compare(T t1, T t2) { // ERROR

	static <T> void sort(List<T> list, Comparator<? super T> c) { // OK
		...
	}
		
	...
}

// 지네릭 메서드로 makeJuice()선언
class Juicer {
	static <T extends Fruit> Juice makeJuice(FruitBox<T> box) {
		String tmp = "";
		for (Fruit fruit : box.getList()) {
			tmp += fruit + " ";
		}
    
		return new Juice(tmp);
	}
}

FruitBox<Fruit> fruitBox = new FruitBox<>();
Juicer.makeJuice(fruitBox)) // 컴파일러가 타입 추정 가능
```

- 컴파일러가 타입을 추정할 수 있는 경우 타입 생략이 가능하지만 대입된 타입을 생략할 수 없는 경우에는 참조변수는 클래스 이름을 생략할 수 없다
- 같은 클래스 내 멤버들끼리는 참조변수나 클래스이름을 생략하고 메서드 이름만으로 호출이 가능하지만, 대입된 타입이 존재하면 반드시 명시해줘야 한다

```java
<Fruit>makeJuice(fruitBox); // ERROR, 클래스 이름 생략불가
this.<Fruit>makeJuice(fruitBox); // OK
Juicer.<Fruit>makeJuice(fruitBox); // OK
```

그렇다면 제네릭 메서드는 언제 사용하는 것이 좋을까? **주로 매개변수의 타입이 복잡한 메서드에 유용하게 사용할 수 있다**

<br>

# 📌 제네릭 타입의 형변환

- 제네릭 타입과 원시 타입(raw type)간의 형변환은 항상 가능하다. 다만 경고가 발생
  - generics <-> non-generics

```java
Box box = null; // non-generics(raw type)
Box<Object> objectBox = null; // generics

box = (Box)objectBox; // 제네릭 -> 원시 타입
objectBox = (Box<Object>)box; // 원시 타입 -> 제네릭
```

- 대입된 타입이 다른 제네릭 타입 간에는 형변환이 불가능하다
  - `Box<String>` <-> `Box<Object>` 불가능

```java
Box<Object> objectBox = null;
Box<String> stringBox = null;

objectBox = (Box<Object>)stringBox; // ERROR
stringBox = (Box<String>)objectBox; // ERROR
```

- 와일드 카드가 사용된 제네릭 타입으로는 형변환이 가능하다
  - Fruit 클래스와 Fruit을 상속받은 자손 클래스들(Apple, Grape)

```java
Box<? extends Object> wildBox = new Box<String>(); // OK

FruitBox<? extends Fruit> fruitBox = null;
FruitBox<Apple> appleBox = (FruitBox<Apple>)fruitBox; // OK, 미확인 타입으로 경고 발생
```

<br>

# 📌 제네릭 타입의 제거

컴파일러는 제네릭 타입을 이용해서 소스파일을 체크하고, 필요한 곳에 형변환을 넣어준다.

그리고 제네릭 타입을 제거하는데, 즉 컴파일된 파일 `.class`에는 제네릭 타입에 대한 정보가 없다. 그 이유는 제네릭이 도입되기 이전의 소스 코드와의 호환성을 유지하기 위해서이다.

제네릭 타입의 기본적인 제거 과정을 다음과 같다.

**1: 제네릭 타입의 경계(bound) 제거**

```java
// 1. 경계 제거
class Box<T extends Fruit> {
  
  void add(T t) {}
}

class Box {
  
  void add(Fruit t) {}
}
```

**2: 제네릭 타입을 제거한 후 타입이 일치하지 않으면 형변환을 추가한다**

```java
T get(int idx) {
  return list.get(idx);
}

Fruit get(int idx) {
  return (Fruit)list.get(idx);
}
```

**3: 와일드 카드가 포함된 경우 적절한 타입으로 형변환을 추가한다**

```java
static Juice makeJuice(FruitBox<? extends Fruit> box) {
  String tmp = "";
  for(Fruit fruit: box.getList()) {
    tmp += f + " ";
  }
  
  return new Juice(tmp);
}

static Juice makeJuice(FruitBox box) {
  String tmp = "";
  Iterator it = box.getList().iterator();

  while (it.hasNext()) {
    tmp += (Fruit)it.next() + " ";
  }
  
  return new Juice(tmp);
}
```

<br>
