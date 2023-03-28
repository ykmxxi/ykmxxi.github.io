---
title: "[자바] 객체의 동일성(Identity)과 동등성(Equality)"
categories: [자바]
tag: ["자바", "Java"]
toc: true
toc_sticky: true
author_profile: false
sidebar:
    nav: "counts"
---

> Reference
>
> [리차드 님의 블로그: 동일성(Identity) vs 동등성(Equality) - feat. equals() hashCode()](https://creampuffy.tistory.com/140)

<br>

# 📌 동일성(Identity)

- 메모리 내 **주소값(참조하는 값)**이 같은지 비교
  - `==` 비교를 통해 객체의 메모리 내 주소값이 같은지 식별
  - 주민등록 번호를 생각
- 두 변수가 **같은 객체의 주소를 참조**하고 있으면 **동일**하다고 한다.

<center><img src="/assets/images/java/Identity.png"></center>

<br>

# 📌 동등성(Equality)

- **논리적 지위**가 동등한지 비교
  - `equals()`와`hasCode()`메서드를 통해 논리적으로 같은 지위를 지녔는지 확인
  - 논리적 지위의 기준은 프로그래머가 정의
- 즉, 두 객체가 주소가 다르더라도 **정의한 기준에 따라 같은 정보**를 갖고 있으면 동등하다고 판단
- **동일한 객체는 동등**하지만, **동등한 객체가 무조건 동일한 것은 아니다**.

<br>

# 📌 == vs. equals()

- `==` 연산자는 기본적으로 객체의 동일성 판단. 즉, 객체가 가리키는 메모리 주소가 같은지 다른지를 판단한다.
- `equals()` 메서드는 동등성 판단. 즉, 두 객체의 논리적 지위가 같은지 다른지를 판단한다.

하지만, 모든 객체의 최고 조상인 `Object` 클래스의 `equals()`는 `==` 연산자를 사용해 동등성을 판단한다.

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

`equals()` 메서드를 오버라이딩 하지 않으면 **동일성 비교와 동등성 비교를 나누는 것은 아무 의미가 없어진다**. 객체 클래스를 정의할 때 프로그래머는 객체의 특성에 맞게 `equals()`를 재정의해 동등성 판단을 진행해야 한다.

<br>

# 📌 equals()

- Object클래스의 `equals()`는  `==` 연산자를 이용해 객체의 주소를 비교(동일성)

`Person` 이라는 객체가 이름(name), 주민번호(identificationNumber), 주소(address), 성별(gender)을 속성으로 가질 때, 우리는 두 사람 객체가 동일한지 비교할 때 메모리 주소를 비교하는 것은 의미가 없다.

```java
class Person {

	public String name;
	public String identificationNumber;
	public String address;
	public String gender;

	public Person(String name, String identificationNumber, String address, String gender) {
		this.name = name;
		this.identificationNumber = identificationNumber;
		this.address = address;
		this.gender = gender;
	}
}

public class Main {
	public static void main(String[] args) {
		Person person1 = new Person("홍길동", "111-111", "서울", "남자");
		Person person2 = new Person("홍길동", "111-111", "서울", "남자");

		if (person1.equals(person2)) {
			System.out.println("같은 사람이다.");
		} else {
			System.out.println("다른 사람이다."); // 두 객체는 주소가 달라 이 메서드가 실행됨
		}
	}
}
```

이름과 주민번호가 같아야 한다는 기준에 맞게  `equals()`를 오버라이딩해 사용해야 한다.

```java
class Person {

	public String name;
	public String identificationNumber;
	public String address;
	public String gender;

	public Person(String name, String identificationNumber, String address, String gender) {
		this.name = name;
		this.identificationNumber = identificationNumber;
		this.address = address;
		this.gender = gender;
	}

	@Override
	public boolean equals(Object obj) {
		if (this == obj) { // 동일한 객체이면 early return
			return true;
		}
		if (obj == null || this.getClass() != obj.getClass()) { // null 또는 클래스가 다르면 early return
			return false;
		}

		Person p = (Person)obj;
		return this.name.equals(p.name) && this.identificationNumber.equals(p.identificationNumber);
	}
}

public class Main {
	public static void main(String[] args) {
		Person person1 = new Person("홍길동", "111-111", "서울", "남자");
		Person person2 = new Person("홍길동", "111-111", "서울", "남자");

		if (person1.equals(person2)) {
			System.out.println("같은 사람이다.");
		} else {
			System.out.println("다른 사람이다.");
		}
	}
}
```

<center><img src="/assets/images/java/result1.png"></center>

`IPhone`클래스가 존재할 때 실세계에서는 기기별 고유 ID가 존재해 같은 객체라 정의할 수 없지만, 여기서는 version이 같다면 같은 객체로 보기로 가정한다.

- 두 객체가 같다는 기준이 될 값들을 비교하도록 재정의 필요: version이 같아야 한다.
  - `Object.equals()`: `==` 비교를 통해 주소값을 비교 (동일성 검사)

```java
class IPhone {

	public String version;

	public IPhone(String version) {
		this.version = version;
	}
}

public class SameAndEqual {

	public static void main(String[] args) {
		String one = "hello world";
		String two = "hello world";

		if (one.equals(two)) {
			System.out.println("동등성: 논리적 지위가 같다");
		} else {
			System.out.println("동등성 X, 논리적 지위가 다르다");
		}

		if (one == two) {
			System.out.println("동일성: 메모리 내 주소값이 같다");
		} else {
			System.out.println("동일성 X, 메모리 내 주소값이 다르다");
		}

		System.out.println();

		IPhone iPhone1 = new IPhone("아이폰11");
		IPhone iPhone2 = new IPhone("아이폰11");

		// new 키워드로 두 번 생성 -> 서로 다른 주소값을 갖음
		if (iPhone1.equals(iPhone2)) {
			System.out.println("동등성: 논리적 지위가 같다");
		} else {
			System.out.println("동등성 X, 논리적 지위가 다르다");
		}

		if (iPhone1 == iPhone2) {
			System.out.println("동일성: 메모리 내 주소값이 같다");
		} else {
			System.out.println("동일성 X, 메모리 내 주소값이 다르다");
		}
	}
}
```

<center><img src="/assets/images/java/result2.png"></center>

- 버전이 같으면 같은 객체로 인식하도록 `equals()` 오버라이딩

```java
class IPhone {

	public String version;

	public IPhone(String version) {
		this.version = version;
	}

	@Override
	public boolean equals(Object obj) {
		if (this == obj) {
			return true;
		}
		if (obj == null || this.getClass() != obj.getClass()) {
			return false;
		}

		return version.equals(((IPhone)obj).version);
	}
}

public class SameAndEqual {

	public static void main(String[] args) {
		IPhone iPhone1 = new IPhone("아이폰11");
		IPhone iPhone2 = new IPhone("아이폰11");

		if (iPhone1.equals(iPhone2)) {
			System.out.println("동등성: 논리적 지위가 같다");
		} else {
			System.out.println("동등성 X, 논리적 지위가 다르다");
		}

		if (iPhone1 == iPhone2) {
			System.out.println("동일성: 메모리 내 주소값이 같다");
		} else {
			System.out.println("동일성 X, 메모리 내 주소값이 다르다");
		}
	}
}
```

<center><img src="/assets/images/java/result3.png"></center>

<br>

# 📌 hasCode()

- `hashCode()`: 객체의 hashCode를 반환하는 메서드
  - hashCode는 각 객체의 **주소값을 변환하여 생성한 객체의 고유한 정수값**
- hashCode()를 재정의 하지 않으면 **주소값을 기반으로 고유값을 생성**하니 동일하지 않은 객체는 무조건 동등하지 않게 된다. 객체의 동등성을 프로그래머가 **정의한 기준에 맞게 재정의**해야 한다.
- 두 객체가 같다의 기준이 될 필드들의 값으로 hasCode를 만들도록 재정의
  - 여기서는 `version`이 같으면 동일한 해시값을 갖도록 재정의한다.

```java
class IPhone {

	public String version;

	public IPhone(String version) {
		this.version = version;
	}

	@Override
	public int hashCode() {
		return Objects.hash(version); // 버전이 같으면 hash 값이 같도록 재정의
	}

	@Override
	public boolean equals(Object obj) {
		if (this == obj) {
			return true;
		}
		if (obj == null || this.getClass() != obj.getClass()) {
			return false;
		}

		return version.equals(((IPhone)obj).version);
	}
}

public class SameAndEqual {

	public static void main(String[] args) {
		IPhone iPhone1 = new IPhone("아이폰11");
		IPhone iPhone2 = new IPhone("아이폰11");

		if (iPhone1.equals(iPhone2)) {
			System.out.println("동등성: 논리적 지위가 같다");
		} else {
			System.out.println("동등성 X, 논리적 지위가 다르다");
		}

		if (iPhone1 == iPhone2) {
			System.out.println("동일성: 메모리 내 주소값이 같다");
		} else {
			System.out.println("동일성 X, 메모리 내 주소값이 다르다");
		}

		System.out.println();
		Set<IPhone> iPhoneSet = new HashSet<>();
		iPhoneSet.add(iPhone1);
		iPhoneSet.add(iPhone2);

		System.out.println("iPhoneSet.size() = " + iPhoneSet.size());
	}
}
```

<center><img src="/assets/images/java/result4.png"></center>