---
title: "[자바의 정석] 01. 자바를 시작하기 전에"
categories: [자바의 정석]
tag: ["Java"]
toc: true
toc_sticky: true
author_profile: false
sidebar:
    nav: "docs"
---

# 1. 자바(Java Programming Language)

**자바 프로그램의 실행 과정**

![http://www.tcpschool.com/lectures/img_java_programming.png](http://www.tcpschool.com/lectures/img_java_programming.png)

> 출처: [TCP 스쿨](http://www.tcpschool.com/java/java_intro_programming)

## 1.1 자바란?

자바의 가장 중요한 특징은 **운영체제(Operating System)에 독립적**이라는 것이다. 자바로 작성된 프로그램은 운영체제의 종류에 관계없이 실행이 가능하다. 즉, 운영체제에 따라 프로그램을 전혀 변경하지 않고도 실행이 가능하다. 이러한 장점으로 인해 자바는 다양한 기종의 컴퓨터와 운영체제가 공존하는 인터넷 환경에 적합한 언어로써 많은 사용자층을 확보할 수 있게 되었다.

자바는 풍부한 클래스 라이브러리 (Java API)를 통해 프로그래밍에 필요한 요소들을 기본적으로 제공하기 때문에 이를 잘 활용한다면 원하는 기능을 충분히 구현해 낼 수 있다.

## 1.2 자바의 역사

자바는 1991년 Oak 라는 언어로부터 시작 되었다. C++의 장점을 도입하고 단점을 보완한 새로운 언어를 개발하기에 이르렀고 이는 자바라는 언어의 탄생을 이끌어냈다.

## 1.3 자바언어의 특징

1. **운영체제에 독립적이다**

   기존의 언어는 한 OS에 맞게 개발된 프로그램을 다른 운영체제에 적용하는 일은 고된 작업이었다. 자바에서는 일종의 에뮬레이터인 자바가상머신(JVM)을 통해서 이러한 고된 작업과정이 없이 가능하게 되었다. 자바 애플리케이션은 OS, Hardware가 아닌 JVM하고만 통신하고, JVM이 전달받은 명령을 해당 운영체제가 이해할 수 있도록 변환하여 전달한다. 자바로 작성된 프로그램은 운영체제에 독립적이지만 JVM은 운영체제에 종속적이어서 각 OS 환경에 맞는 가상머신이 필요하고 이는 제공되고 있다.

   그래서 자바로 작성된 프로그램은 운영체제와 하드웨어에 관계없이 실행 가능하며 이것을 **'한번 작성하면, 어디서나 실행된다.' (Write once, run anywhere)** 고 표현한다.

2. **객체지향 언어**

   자바는 객체지향 프로그래밍언어(object-oriented programming language)중 하나이다.
   객체지향개념의 특징인 상속, 캡슐화, 다형성이 잘 적용된 순수한 객체지향언어이다.

3. **자동 메모리 관리(Garbage Collection)**

   자바는 **garbage collector가 자동적으로 메모리를 관리**해주기 때문에 **프로그래머는 메모리를 따로 관리 하지 않아도 된다**는 장점이 있다.
   만약 g.c가 존재하지 않았다고 프로그래머가 사용하지 않는 메모리를 직접 체크하고 반환하는 일을 수동적으로 처리해야할 것이다. 다소 비효율적인 면도 있지만, 프로그래머는 온전히 개발에 집중할 수 있다.

4. **네트워크와 분산처리를 지원한다**

   인터넷과 대규모 분산환경을 염두에 둔 까닭인지 풍부하고 다양한 네트워크 프로그래밍 라이브러리를 통해 개발할 수 있도록 지원한다.

5. **멀티쓰레드를 지원한다**

   시스템과는 관계없이 구현가능하며, 관련된 라이브러리가 제공되므로 멀티쓰레드의 구현이 쉽다. 여러 쓰레드에 대한 스케줄링(scheduling)을 자바 인터프리터가 담당하게 된다.

6. **동적 로딩(Dynamic Loading)을 지원한다**

   자바로 작성된 애플리케이션은 여러 개의 클래스로 구성되어 있다. 자바는 동적 로딩을 지원하기 때문에 실행 시 모든 클래스가 로딩되지 않고 필요한 시점에 클래스를 로딩하여 사용할 수 있다는 장점이 존재한다. 그 외에도 일부 클래스가 변경되어도 전체 애플리케이션을 다시 컴파일하지 않아도 되며, 변경 사항이 발생해도 비교적 적은 작업만으로 처리가 가능하다.

> 참고: 자바의 단점으로는 속도문제가 가장 대표적이다. 이를 바이트코드를 하드웨어의 기계어로 바로 변환해주는 JIT 컴파일러와 Hostspot과 같은 신기술 도입으로 어느부분 개선되었다.

## 1.4 JVM (Java Virtual Machine)

JVM은 자바를 실행하기 위한 가상 머신이다. 자바로 작성된 애플리케이션은 모두 이 가상 머신(JVM)에서만 실행되게 때문에, 자바 프로그램이 실행되기 위해서는 반드시 JVM이 필요하다.

일반 애플리케이션의 코드는 OS만 거치고 하드웨어로 전달되는데 비해 Java 애플리케이션은 JVM을 한번 더 거치기 때문에, 그리고 하드웨어에 맞게 완전히 컴파일된 상태가 아니고 실행 시 해석(interpret)되기 때문에 느리다는 단점을 가지고 있다.

일반 애플리케이션은 OS와 맞붙어 있기 때문에 OS종속적이다. Java 애플리케이션은 JVM하고만 상호작용을 하기 때문에 OS와 하드웨어에 독립적이라 다른 OS에서도 프로그램의 변경없이 실행이 가능한 것이다. 단, JVM은 OS에 종속적이기 때문에 해당 OS에서 실행가능한 JVM이 필요하다.

![http://www.tcpschool.com/lectures/img_java_jvm.png](http://www.tcpschool.com/lectures/img_java_jvm.png)

> 출처: [TCP 스쿨]( http://www.tcpschool.com/java/java_intro_programming)

<br>

# 2. 자바의 프로그램작성

## 2.1 Hello.java

```java
class Hello {
  public static void main(String[] args) {
    System.out.println("Hello World!!!");
  }
}
```

- Hello.java 작성 →(javac.exe) 컴파일 → Hello.class 생성 → (java.exe) 실행 → `"Hello World!!!"` 출력
- 소스파일의 이름은 public class 의 이름과 일치해야 한다.
  - 하나의 소스파일에 둘 이상의 public class가 존재하면 안된다.
- `public static void main(String[] args)` 는 main 메서드의 선언부 이며, 프로그램을 실행할 때 java.exe 에 의해 호출될 수 있도록 미리 약속된 부분이므로 항상 똑같이 적어주어야 한다.

## 2.2 자주 발생하는 에러와 해결방법

1. `cannot find symbol` 또는 `cannot resolve symbol`

   지정된 변수나 메서드를 찾을 수 없다는 뜻으로, 선언되지 않은 변수나 메서드를 사용하거나 변수 또는 메서드의 이름을 잘못 사용한 경우 발생

2. `" ; " expected`

   세미콜론 (`;`) 이 필요한 곳에 없다는 뜻

3. `Exception in thread "main" java.lang.NoSuchMethodError : main`

   main 메서드를 찾을 수 없다는 뜻

4. `Exception in thread "main" java.lang.NoClassDefFoundError : Hello`

   Hello 클래스를 찾을 수 없다는 뜻.

5. `illegal start of expression`

   수식의 앞부분이 문법에 맞지 않는다는 의미 (수식에 문법적 오류가 있다는 뜻)

6. `class, interface, or enum expected`

   키워드 class 나 interface 또는 enum이 없을때 발생, 보통 괄호의 개수가 일치 하지 않는 경우에 발생

## 2.3 자바프로그램의 실행과정

Java 애플리케이션을 실행시켰을 때

1. 프로그램의 실행에 필요한 클래스(*.class 파일)를 로드한다

2. 클래스파일을 검사한다 (파일형식, 악성코드 체크)

3. 지정된 클래스에서 `main(String[] args)`를 호출한다

   만일 지정된 클래스에 main메서드가 없다면 에러 메시지가 발생

<br>

# 3. JDK (자바개발도구)

- Java Development Kit, JVM과 자바클래스 라이브러리외 필요 프로그램들을 담고 있음
- JDK의 bin 디렉토리에 있는 주요 실행파일들
  - javac.exe : 자바 컴파일러, 자바소스코드를 바이트코드로 컴파일
  - java.exe : 자바 인터프리터, 컴파일러가 생성한 바이트코드를 해석하고 실행
  - javap.exe : 역 어셈블러, 컴파일된 클래스파일을 원래의 소스로 변환
    - javap Hello > Hello.java : Hello.class 파일이 역컴파일되어 Hello.java에 저장됨
  - javadoc.exe : 자동문서생성기, 소스파일에 있는 주석을 이용해 Java API 문서와 같은 형식의 문서를 자동으로 생성
  - jar.exe : 압축프로그램, 클래스파일과 프로그램의 실행에 관련된 파일을 하나의 jar파일 ( .jar )로 압축하거나 압축해제
    - 압축할 때 : jar cvf Hello.jar Hello1.class Hello2.class
    - 압축 될 때 : jar xvf Hello.jar

> - JRE (Java Runtime Environment) : 자바로 작성된 응용프로그램이 실행되기 위한 최소환경
> - JDK = JRE + 개발에 필요한 실행파일(javac.exe, javap.exe 등)
> - JRE = JVM + Java API