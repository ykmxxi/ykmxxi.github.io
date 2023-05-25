---
title: "[객체지향] 객체와 객체 지향, 책임, 의존"
categories: [객체지향]
tag: ["객체지향"]
toc: true
toc_sticky: true
author_profile: false
sidebar:
    nav: "counts"
---

> **Reference**
>
> 개발자가 반드시 정복해야 할 객체 지향과 디자인 패턴, 인투북스, 최범균 지음

<br>

객체 지향 언어를 사용한다고 객체 지향 기법을 적용한 것이 아니다. 객체 지향 언어를 사용해도 절차 지향적으로 프로그램을 작성하면 실제 결과물은 객체 지향과 거리가 멀어질 수 있다.

**객체 자체와 객체 지향의 핵심인 캡슐화 및 추상화가 적용되지 않으면 객체 지향 기법을 사용한 것이 아니다.**

# 📌 절차 지향 vs 객체 지향

## 절차 지향(Procedural Oriented)

- SW를 구현한다는 것은 SW를 구성하는 데이터와 데이터를 조작하는 코드를 작성하는 것이다.
- 함수 or 프로시저(procedure): 데이터를 조작하는 코드
  - 프로시저는 다른 프로시저를 사용할 수도 있고, 각각의 프로시저가 같은 데이터를 사용할 수도 있다.
- 절차 지향: 프로시저로 프로그램을 구성하는 기법
  - 절차 지향 프로그래밍은 영어 단어 Procedural을 우리말로 번역한 것뿐이지 순서에 따라 프로그래밍하는 방식이 아니다
  - 프로시저를 이용한 프로그래밍 기법을 의미하기 때문에 사실 절차지향 보다는 프로시저 지향 이라 부르는 것이 더 의미에 맞는다

<center><img src="/assets/images/oo/object/1.png"></center>

- 각 프로시저는 데이터를 사용해서 기능을 구현, 필요에 따라 다른 프로시저를 사용
- 다수의 프로시저들이 데이터를 공유하는 방식으로 만들어져 절차 지향 프로그래밍은 데이터를 중심으로 구현하게 된다

## 절차 지향의 문제점

- 데이터와 데이터를 다루는 프로시저를 작성하는 것은 자연스러운 방식이다. 하지만 프로그램 규모가 커져 데이터 종류가 증가하고 이를 사용하는 프로시저가 늘어날 수록 문제들이 발생한다

1. 데이터 타입이나 의미를 변경해야 할 때 함께 수정해야 하는 프로시저가 증가한다
   - 데이터를 사용하는 프로시저가 많을수록 그 데이터의 타입을 변경하기 여럽다
2. 같은 데이터를 프로시저들이 다른 의미로 사용하는 경우가 발생한다

- 소프트웨어는 변화하는 요구사항에 빠르게 대응해야 한다. 요구 사항이 생겨 수정이 필요할 때 변경이 어렵다면 좋은 소프트웨어를 만들기 어려워지고, 그 비용은 점점 늘어나게 된다.

## 객체 지향(Object Oriented)

- **객체 지향**: 데이터 및 데이터와 관련된 프로시저를 객체(Object) 라고 불리는 한 단위로 묶은 것들이 모여 프로그램을 구성한다
  - 객체는 프로시저를 실행하는데 필요한 만큼의 데이터를 가진다
  - 객체 지향 기법을 적용한 프로그램은 데이터와 프로시저를 함께 갖는 객체들의 네트워크로 구성된다

<center><img src="/assets/images/oo/object/2.png"></center>

- 각 객체는 자신만의 데이터와 프로시저를 갖고, 자신만의 기능을 제공한다
- 객체는 서로 연결되어 다른 객체가 제공하는 기능을 사용할 수 있게 된다
  - 객체의 프로시저는 자신이 속한 객체의 데이터만 접근할 수 있고 다른 객체에 속한 데이터에는 접근할 수 없다

## 객체 지향의 장단점

- 장점
  - 객체의 데이터를 변경해도 해당 객체로만 변화가 집중되고 다른 객체에 영향을 주지 않는다
  - 즉, 요구 사항의 변화가 있을 때 절차 지향보다 더 쉽게 변경할 수 있다 (캡슐화와 관련)
- 단점
  - 설계가 어렵다
- 객체 지향은 설계하는데 더 많은 노려이 들어갈 수 있지만 프로그램을 상대적으로 더 쉽게 수정할 수 있는 유연함을 제공한다. 따라서 변화된 요구 사항을 빠르게 반영할 수 있도록 만들어주는 기법이다

<br>

# 📌 객체(Object)

## 객체의 핵심은 기능 제공

- 객체는 데이터와 그 데이터를 조작하는 프로시저(메서드, 함수)로 구성되는데 이는 객체의 물리적 특징일 뿐이다.
- 실제로 객체를 정의할 때 사용되는 것은 **객체가 제공해야 할 기능**이며, 객체가 내부적으로 어떤 데이터를 갖고 있는 지로는 정의되지 않는다.
- 객체가 내부적으로 데이터를 어떤 타입으로 저장하는지, 어떻게 기능을 수행하는지 알 수 없다. 단지 어떤 기능을 제공한다는 것이 중요하다.

## 인터페이스와 클래스

- 객체는 객체가 제공하는 기능으로 정의되고, 보통 객체가 제공하는 기능을 오퍼레이션(operation)이라 부른다
  - 객체(object) == 오퍼레이션(operation)
- 객체를 사용한다는 것은 오퍼레이션을 사용한다는 뜻이고, 이를 위해 오퍼레이션의 사용법을 알아야 한다
- 오퍼레이션의 사용법은 총 세 개로 구성되며, 이 세 가지를 합쳐서 시그니처(Signature)라 부른다
  - 익히 아는 메서드 시그니처를 뜻한다

- **오퍼레이션 시그니처(Operation Signature)**

  - 기능 식별 이름(method name)

  - 파라미터 및 파라미터 타입

  - 기능 실행 결과 값
    - void
    - Primitive type
    - Wrapper class


- **인터페이스(interface)**: 객체가 제공하는 모든 오퍼레이션 집합
  - 객체를 사용하기 위한 일종의 명세서
  - 자바의 인터페이스가 아니라 객체 지향에서 오퍼레이션 집합을 표현할 때 사용되는 의미의 인터페이스
- **타입(type)**: 서로 다른 인터페이스를 구분할 때 사용하는 명칭
- **클래스(class)**: 실제 객체의 구현을 정의
  - 명세서인 인터페이스를 기반으로 실제로 동작하는 것을 만드는 것

```java
/* 소리 크기 인터페이스(명세서)

	- increaseVolume(), 파라미터 없음, 결과 없음
	- decreaseVolume(), 파라미터 없음, 결과 없음
	- mute(), 파라미터 없음, 결과 없음
	- setVolume(), 파라미터 int newVolume, 결과 없음
	- getVolumeRate(), 파라미터 없음, 결과 double
*/

class VolumeController implements Volume {

  private int volume;

  void increaseVolume() {
    // ...
  }

  void decreaseVolume() {
    // ...
  }

  void mute() {
    // ...
  }

  void setVolume(int newVolume) {
    // ...
  }

  double getVolumeRate() {
    // ...
  }
}
```

<center><img src="/assets/images/oo/object/3.png"></center>

## 메시지

- 객체 지향은 기능을 제공하는 여러 객체들이 모여서 완성된 어플리케이션을 구성하게 된다
- 즉, 객체는 **메시지(message)**를 보내 **오퍼레이션(operation)**의 실행을 **요청(request)**한다

**특정 파일에서 데이터를 읽어와 암호화한 뒤 다른 파일에 쓰는 프로그램**

<center><img src="/assets/images/oo/object/4.png"></center>

```java
FileInputStream fis = new FileInputStream("example.txt");
byte[] data = new byte[512];

// fis가 참조하는 객체에 read() 오퍼레이션을 실행해 달라는 메시지를 보낸다
int readBytes = fis.read(data);
```

<br>

# 📌 객체의 책임과 크기

- 객체의 정의 → 객체가 **제공하는 기능** → 객체마다 자신만의 **책임(responsibility)**이 있다
- 객체가 **책임**을 갖는다는 것은 객체가 **역할(role)**을 수행한다는 의미이다

<center><img src="/assets/images/oo/object/5.png"></center>

- 한 객체가 갖는 책임을 정의한 것이 타입/인터페이스 라 생각하면 된다.
- 그렇다면 책임은 어떻게 정하나? 이 결정을 내리는 것이 객체 지향 설계의 출발점이다
  - 기능 목록을 정의하고 변화가 발생했을 때 함께 수정되어야 하는 것들을 묶는다

## 예시

**특정 파일에서 데이터를 읽어와 암호화한 뒤 다른 파일에 쓰는 프로그램**

- 기능 목록 정의
  - 파일의 byte 데이터를 제공
  - 파일에 byte 데이터 쓰기
  - 암호화해서 새로운 byte 데이터 생성
  - 전체 흐름 제어
- CASE 1
  - 객체 1: 흐름 제어, byte 암호화
  - 객체 2: 파일 읽기, 파일 쓰기
- CASE 2
  - 객체 1: 흐름 제어
  - 객체 2: byte 암호화
  - 객체 3: 파일 읽기, 파일 쓰기
- CASE 3
  - 객체 1: 흐름 제어
  - 객체 2: byte 암호화
  - 객체 3: 파일 읽기
  - 객체 4: 파일 쓰기

## 객체가 갖는 책임의 크기는 작을수록 좋다

- 상황에 따라 객체가 가져야 할 기능의 종류와 개수가 다르고, 모든 상황에 들어맞는 객체-책임 구성 규칙이 존재하는 것은 아니다.
- 하지만, **객체가 갖는 책임의 크기는 작을수록 좋다**. 책임의 크기가 작다는 것은 객체가 **제공하는 기능의 개수가 적다**는 것을 의미한다.
  - 크기가 작아질수록 객체 지향의 장점인 **변경의 유연함**을 얻을 수 있다 → SRP(단일 책임 원칙)
  - SRP(단일 책임 원칙): 한 객체는 한 개의 책임을 갖도록 구성한다
- 만약 한 객체에 기능이 많으면 해당 객체에는 그 기능과 관련된 모든 데이터가 들어가게 된다. 따라서 절차 지향적인 구조를 갖게 되고 절차 지향의 단점인 **변경의 어려움**이 발생한다.

<br>

# 📌 의존(Dependency)

- 객체 지향 프로그래밍을 하다 보면 한 객체가 다른 객체의 기능을 이용해서 자신의 기능을 완성하는 객체가 발생한다
  - 흐름 제어 객체는 파일 읽기, 쓰기, 암호화 객체 모두를 이용해서 프로그램의 실행 흐름 기능을 구현한다

```java
public class FlowController {
	
	public void process() {
		FileDataReader reader = new FileDataReader(fileName); // 다른 객체 생성
		byte[] plainBytes = reader.read(); // 다른 객체의 메서드를 호출
		
		ByteEncryptor encryptor = new ByteEncryptor(); // 다른 객체 생성
		byte[] encryptedBytes = encryptor.encrypt(plainBytes); // 다른 객체의 메서드를 호출
		
		FileDataWriter writer = new FileDataWriter(); // 다른 객체 생성
		writer.write(encryptedBytes); // 다른 객체의 메서드를 호출
		
	}
}
```

- 이렇게 한 객체가 **다른 객체를 생성**하거나 **다른 객체의 메서드를 호출**할 때 그 객체에 **의존(dependency)**한다고 말한다
  - 파라미터로 전달받는 경우에도 의존한다고 볼 수 있다
- 의존하는 형태가 어떻든 다른 타입에 의존한다는 것은 의존하는 타입에 변경이 발생할 때 함께 변경될 가능성이 높다는 것을 뜻한다
- 의존의 영향은 꼬리에 꼬리를 문 것처럼 전파된다
  - 아래 예제에서 `A` 에 변경이 발생하면 `B`, `C`에 차례대로 변경의 영향이 전파된다

<center><img src="/assets/images/oo/object/6.png"></center>

```java
class A {
}

class B {
  A a = new A();
}

class C {
  B b = new B();
}
```

- 이런 의존의 특징 때문에 순환 의존 문제가 발생할 수 있다. 즉, 변경의 여파가 나 자신에게 다시 영향을 줄 수 있다.
  - 순환 의존이 발생하지 않도록 하는 원칙 중 하나는 DIP(의존 역전 원칙)

## 의존의 양면성

```java
class Authenticator {

	public boolean authenticate(String id, String password) {

		Member member = findById(id);
		if (member == null) {
			return false;
		}

		return member.equalPassword(password);
	}
	
}

class AuthenticationHandler {

	public void handleRequest(String inputId, String inputPassword) {

		Authenticator auth = new Authenticator();

		if (auth.authenticate(inputId, inputPassword)) {
			System.out.println("인증 완료");
		} else {
			System.out.println("인증 실패");
		}
	}
	
}
```

<center><img src="/assets/images/oo/object/7.png"></center>

- 여기서 새로운 요구사항이 등장한다. 인증이 실패하면 아이디가 잘못 된 것인지, 패스워드가 잘못 된 것인지 로그를 남겨달라는 `AuthenticationHandler` 클래스 요구 사항이 추가된다면 어떻게 될까?

```java
class AuthenticationHandler {

	public void handleRequest(String inputId, String inputPassword) {

		Authenticator auth = new Authenticator();

		try {
			auth.authenticate(inputId, inputPassword);
		} catch (MemberNotFoundException e) {
			System.out.println("로그 = " + e);
		} catch (InvalidPasswordException e) {
			System.out.println("로그 = " + e);
		}
	}

}
```

- 그렇다면 `Authenticator` 클래스의 `authenticate()`는 `boolean` 값을 반환하면 안 되고 예외를 던져줘야 한다.

```java
class Authenticator {

	public void authenticate(String id, String password) {

		Member member = findById(id);
		
		if (member == null) {
			throw new MemberNotFoundException();
		}
		if (member.equalPassword(password)) {
			throw new InvalidPasswordException();
		}
		
	}

}
```

- `AuthenticationHandler` 클래스가 `Authenticator` 클래스에 의존하고 있는 상황에서 `AuthenticationHandler` 클래스의 변경의 여파가 `Authenticator` 클래스에 영향을 미치고 있다.
- 즉, 의존 관계가 있을 때 **서로 영향을 준다**는 것을 확인할 수 있다
  - 내가 변경되면 나에게 의존하고 있는 코드에 영향을 준다
  - 나의 요구가 변경되면 내가 의존하고 있는 타입에 영향을 준다