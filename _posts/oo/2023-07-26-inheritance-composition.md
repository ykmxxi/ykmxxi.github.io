---
title: "[객체지향] 재사용: 상속 vs Composition"
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

# 📌 상속과 재사용

객체 지향의 주요 특징 중 하나인 **재사용**을 말할 때 그 예시로 **상속**을 드는 경우가 있다. 상속을 사용하면 상위 클래스 기능을 그대로 재사용할 수 있어 상속을 하면 재사용을 쉽게 할 수 있다는 것은 분명하다.

하지만 클래스 상속은 기능 재사용에 있어 강력한 도구는 맞지만 문제점이 많다. 실제로 `extends` 상속은 캡슐화를 위반할 수 있어 `implements` 구현을 권장하기도 한다.

**상속(extends)보단 조립(Composition)**을 사용하는 것이 좋다

상속을 사용하면 쉽게 다른 클래스의 기능을 재사용하면서 추가 기능을 확장할 수 있기 때문에, 상속은 기능을 재사용하는 매력적인 방법이다. 하지만, 상속은 변경의 유연함이라는 측면에서 치명적인 단점을 갖는다.

- 상속은 코드 재사용만을 위한 수단이 아니다, 상속은 **확장(extends)의 관점**에서 사용해야 한다

## 상속을 통한 재사용의 문제 1: 상위 클래스 변경의 어려움

- **상속은 상위 클래스의 변경을 어렵게 만든다**

예를 들어 상위 클래스의 구현을 일부 변경하거나 일부 메서드의 시그니처를 변경했다고 가정한다. 이때 해당 클래스를 상속 받은 클래스는 상위 클래스 변경에 영향을 받게 된다.

**즉, 계층도를 따라 변경의 여파가 하위 클래스에 전파되는 것이다.** 최악의 경우 상위 클래스의 변화가 모든 하위 클래스에 영향을 줄 수 있다. 이는 클래스 계층도에 있는 클래스들을 한 개의 거대한 단일 구조처럼 만들어주는 결과를 초래한다. 이런 이유로 클래스 계층도가 커질수록 상위 클래스를 변경하는 것은 점점 어려워진다.

<center><img src="/assets/images/oo/composition/1.png"></center>

## 상속을 통한 재사용의 문제 2: 클래스의 불필요한 증가

- **유사한 기능을 확장하는 과정에서 클래스의 개수가 불필요하게 증가할 수 있다**

예를 들어 파일 보관소를 구현한 클래스가 있다고 가정한다. 제품 출시 이후 보관소 용량을 아끼기 위해 압축 기능을 추가한 클래스와 보안 문제를 위해 암호화 저장을 제공하는 클래스를 파일 보관소 클래스를 상속받아 추가했다.

```java
class Storage {

	public void store() {
	}

}

class CompressedStorage extends Storage {

	@Override
	public void store() {
	}

}

class EncryptedStorage extends Storage {

	@Override
	public void store() {
	}

}
```

만약 여기서 압축과 암호화를 동시에 진행해 달라는 요청이 들어오면 어떻게 해야하나?

```java
class CompressedEncryptedStorage extends CompressedStorage {

	@Override
	public void store() {
	}

	// 암호화 기능 구현
	public void encrypt() {
	}
}
```

Java 에서는 다중 상속이 불가능하기 때문에 만약 압축 저장을 제공하는 클래스를 상속받아 구현하면 암호화에 대한 기능을 구현하기 위해 암호화 기능을 직접 구현해야 한다.

이렇게 필요한 기능의 조합이 증가할수록 **상속을 통한 기능 재사용을 하면 클래스의 개수는 함께 증가**하게 된다.

## 상속을 통한 재사용의 문제 3: 상속의 오용

- **상속 자체를 잘못 사용할 수 있다**

컨테이너 수화물 목록을 관리하는 클래스가 필요하다고 가정할 때, 이 클래스는 세 가지 기능을 제공할 수 있다

1. 수화물을 넣는다
2. 수화물을 뺀다
3. 수화물을 넣을 수 있는지 확인한다

개발자가 이 기능을 구현할 때 목록 관리 기능을 직접 구현하지 않고 ArrayList를 상속 받아 사용하기로 결정했다면 다음과 같다.

```java
import java.util.ArrayList;

class Container extends ArrayList<Luggage> {

	private int maxSize;
	private int currentSize;

	public Container(int maxSize) {
		this.maxSize = maxSize;
	}

	public void put(Luggage luggage) {
		if (!canContain(luggage)) {
			throw new IllegalArgumentException();
		}
		super.add(luggage);
		currentSize += luggage.size();
	}

	public void extract(Luggage luggage) {
		super.remove(luggage);
		currentSize -= luggage.size();
	}

	public boolean canContain(Luggage luggage) {
		return maxSize >= currentSize + luggage.size();
	}

}

class Luggage {

	private final int size;

	Luggage(int size) {
		this.size = size;
	}

	public int size() {
		return size;
	}

}
```

컨테이너 관리 클래스 구현이 끝나고 개발자들은 만족하며 이 클래스를 사용하기 시작했다.

최신 IDE는 친절하게도 해당 인스턴스에서 사용할 수 있는 기능들을 자동 완성 기능을 통해 제공하는데, 원래 제공하려면 `put()`, `extract()`, `canContain()` 기능 이외에도 수 많은 메서드가 목록에 나타난다.

<center><img src="/assets/images/oo/composition/2.png"></center>

만약 다른 팀의 개발자가 관리 클래스 매뉴얼을 숙지하지 않아 컨테이너에 물건을 넣기위해 `put()`을 사용해야 하는데 ArrayList에서 제공하는 `add()` 기능을 사용하면 겉보기에는 물건이 정상적으로 들어간것 처럼 보이지만, 컨테이너 전체 무게에 아무런 영향을 주지 않아 심각한 서비스 장애로 이어질 수 있다.

위 예시는 **상속을 오용한 전형적인 예시**인데, 매뉴얼을 숙지하지 않아 장애를 일으킨 개발자의 잘못도 크지만 오용의 여지를 준 컨테이너 클래스 개발자도 큰 잘못이 있다.

Container 클래스는 목록 기능을 제공하는 ArrayList로 사용하기 위해 만들어진 것이 아니다. 따라서 Container 객체를 ArrayList에 할당해 사용하면 원래 Container가 제공하려던 기능이 정상적으로 동작하지 않는다

위 같은 문제가 발생하는 이유는 Container는 ArrayList가 아니기 때문이다. 상속은 IS-A 관계가 성립할 때 사용해야 하는데 "Container is a ArrayList"는 틀린 관계이기 때문이다. 즉, "Conainer is not a ArrayList"인데 상속을 사용하고 있다.

Container는 짐을 보관하는 책임을 갖고, ArrayList는 목록을 관리하는 책임을 갖는다. 서로 다른 책임을 갖는데 Container가 ArrayList라는 것은 명백히 틀린 관계이다.

이렇게 같은 종류가 아닌 클래스의 구현을 재사용하기 위해 상속을 받으면 잘못된 사용으로 인해 문제가 발생하게 된다.

<br>

# 📌 조립(Composition)을 이용한 재사용

- **객체 조립(Composition)**: 여러 객체를 묶어서 더 복잡한 기능을 제공하는 객체를 만드는 것
  - 클래스가 다른 클래스의 객체를 포함하여 더 복잡한 객체를 만드는 방법

객체 지향 언어에서 composition은 보통 필드에서 다른 객체를 참조하는 방식으로 구현된다. 한 객체가 다른 객체를 조립해 필드고 갖는다는 것은 다른 객체의 기능을 사용한다는 의미를 내포한다.

## composition의 장점

그렇다면 앞에서 살펴본 Storage 클래스를 상속이 아닌 composition을 이용해 구현해 본다.

```java
class Storage {

	private Compressor compressor;
	private Encryptor encryptor;

	public void store() {
		compressor.compress();
		encryptor.encrypt();
	}

}

class Compressor {

	public void compress() {
	}

}

class Encryptor {

	public void encrypt() {
	}

}
```

이렇게 composition을 이용하면 저장 기능을 제공하는 객체에 압축과 암호화 기능을 제공하는 객체를 필드로 가져 기능을 사용할 수 있다. 이제 기능을 새로 추가한다고 해도 Storage 클래스가 증가하지 않는다.

상속을 사용하면 Storage 클래스의 기능이 추가로 요구될 때 마다 상속받은 클래스들이 증가했는데 composition을 이용하면 불필요한 클래스 증가를 방지할 수 있다.

또 composition을 이용해 상속의 오용을 방지할 수 있다. 상속을 이용하면 객체의 역할과 책임을 온전히 수행하지 못할 가능성이 커진다. 앞서 Container가 ArrayList를 상속받았을 때 처럼 본래의 목적과 다르게 사용될 수 있는 문제를 composition을 통해 방지할 수 있다.

상속의 경우 구현시 관계가 형성되기 때문에 런타임에 상위 클래스를 교체할 수 없다. 반면 composition을 사용하면 얼마든지 런타임에 교체가 가능하다. 또 상속을 통한 변경의 여파가 존재하지 않기 때문에 유지보수 관점에서도 더 쉽고 유연한 변경이 가능하다.

상속 대신 composition을 사용했을 때 장점을 정리하면 다음과 같다.

- 불필요한 하위 클래스 증가를 방지해 더 유연한 디자인
  - 강한 계층 구조가 형성되지 않아 느슨한 결합을 가능하게 하며 변경이나 수정이 더 쉽게 이루어진다
  - 더 쉬운 유지보수
  - composition은 HAS-A 관계를 형성해 해당 객체의 기능을 활용할 수 있어 더 작은 단위의 클래스들을 조합해 더 큰 기능을 가진 클래스를 구성할 수 있다
- 상속의 오용을 방지
  - 객체의 목적에 맞는 사용이 가능
- 런타임에 조립 대상 객체를 교체할 수 있다
  - 변경에 유연하게 대처 가능

composition을 사용하면 상속 기반의 재사용에서 발생한 여러 문제들을 해결할 수 있다. 물론 모든 상황에 composition을 사용해야 한다는 뜻은 아니다. 상속을 사용하다 보면 변경의 관점에서 유연함이 떨어질 가능성이 높으니 composition을 먼저 고려해보라는 뜻이다.

compotion도 상속에 비해 런타임 구조가 복잡해지고 구현이 더 여렵다는 단점이 있다. 하지만 변경의 유연함에서 오는 장점이 더 크기 때문에, 기능을 재사용할 경우 composition을 먼저 고려하자.

## 위임(delegation)

- **위임(delegation)**: 하나의 객체가 다른 객체에게 특정 작업을 넘긴다는 의미
  - composition의 한 형태

위임은 HAS-A 관계를 형성해 객체 간의 결합도를 낮추고 코드 재사용성과 유지 보수성을 향상 시킨다. 위임은 composition 처럼 요청을 위임할 객체를 필드로 연결한다.

```java
class Figure {

	private Bounds bounds = new Bounds(); // 위임 대상을 필드에 가짐

	public boolean contains(Point point) {
		// bounds 객체에 처리를 위임
		return bounds.contains(point);
	}

}
```

하지만 composition 처럼 꼭 필드로 정의해야 하는 것은 아니고 해당 객체가 다른 객체에 요청을 한다는 의미에 집중해야 한다. 객체를 새로 생성해 요청을 전달해도 위임이란 의미에서 벗어나지는 않는다

```java
class Figure {

	public boolean contains(Point point) {
		Bounds bounds = new Bounds();
		return bounds.contains(point);
	}

}
```

위임은 다음과 같은 특징을 갖는다

1. **기능 위임**: 자체적으로 처리할 수 없는 기능을 다른 객체에게 위임, 자신의 필드에 다른 객체를 갖고 있어 해당 객체의 기능을 호출해 작업을 처리
2. **더 작은 단위의 객체 조합**: 작은 단위의 객체를 조합해 더 복잡한 기능을 가진 객체를 구현할 수 있다
3. **낮은 결합도**: 객체 간의 결합도를 낮춰 변경에 유연하다

위임은 객체의 책임 분리를 이끌어낸다. 객체는 자신의 핵심적인 역할에만 집중하고, 그 외의 기능은 다른 객체에 위임해 단일 책임 원칙(SRP)을 따르는 설계를 할 수 있다.

위임은 다른 객체에게 요청을 하는 과정에서 메서드 호출이 추가되기 때문에 실행 시간이 다소 증가한다. 연산 속도가 매우 중요한 시스템에서는 많은 위임 코드가 성능에 문제를 일으킬 수 있지만 대부분의 경우 위임으로 인해 발생하는 미세한 성능 저하보다 위임을 통해 얻을 수 있는 유연함과 재사용의 장점이 더 크다.

## 그렇다면 상속은 언제 사용?

기능의 재사용에 상속 보다는 composition, 위임 이라는 것을 알게 되었다. 그렇다면 상속은 언제 사용해야 할까? 상속은 재사용의 관점이 아닌 **기능의 확장**이라는 관점에서 사용해야 한다. 또 명확한 IS-A 관계가 성립할 때 상속을 사용해야 한다.

IS-A 관계를 갖는 클래스 계층은 하위로 내려갈수록 상위 클래스의 기본적인 **기능을 그대로 유지하면서 그 기능을 확장**해 나간다는 점이다.

안드로이드(Android) UI 위젯과 관련된 클래스들의 계층도를 보면 최상위에 View 클래스로 화면에 보이는 위젯의 기본적인 속성을 표현하고, 그 밑으로 상속을 통해 텍스트나 버튼, 목록 등을 보여주는 기능을 확장해 나가면서 기능을 제공한다.

이처럼 상속은 명확한 IS-A 관계에서 점진적으로 상위 클래스의 기능을 확장해 나갈 때 사용할 수 있다. 단 처음 명확한 IS-A 관계에서 기능을 확장해 나가며 불필요하게 클래스가 많이 증가하고 상위 클래스의 변경이 여려워지면 composition으로 전환하는 것을 고려해야 한다.