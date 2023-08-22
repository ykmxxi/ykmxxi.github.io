---
title: "[우테코-프리코스] 4주 차 회고"
categories: [wooteco]
tag: [wooteco]
author_profile: false
sidebar:
    nav: "counts"
---

# 1. `getter`를 사용하는 대신 객체에 메시지를 보내자

> **자바 빈 설계 규약**에 따르면 클래스의 멤버변수의 접근제어자는 `private` 이며, 모든 멤버변수에 대해 `get/set` 메소드가 존재하야 한다.

상태값을 갖는 객체에서 상태값을 외부에서 직접 접근해 변경하지 못하도록 메소드만 노출시킨다. 이때, 멤버변수는 접근 제한자를 `private` 으로 설정해 직접적인 접근을 막고, `getter/setter` 를 이용해 변수에 접근이 가능하도록 한다.

```java
class Member {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

`getter`를 통해 가져온 값을 통해 로직을 수행하는 경우가 존재한다. `getter`의 접근 제어자는 `public`으로 **모든 곳에서 접근**이 가능한데, 이는 부정적인 효과를 낳을 수 있지 않을까?

## 📌 객체에 메시지를 보내 객체가 로직을 수행하도록 하자

객체는 `역할, 책임, 협력` 을 지니고 있다. 객체는 다른 객체와 협력을 위해 `메시지` 를 주고 받는다. 이를 통해 각자의 역할(책임)에 맞는 `행동(로직)`을 수행하게 되고, 이 때 필요하면 값을 스스로 변경하기도 한다. 객체지향 프로그래밍은 객체에 `역할, 책임, 협력` 을 부여해 스스로 일을 하도록 하는 프로그래밍이다.

모든 멤버변수에 `getter`를 생성해 놓고 상태값을 꺼내 그 값으로 객체 외부에서 로직을 수행한다면, 객체가 로직(행동)을 갖고 있는 형태가 아니고 메시지를 주고 받는 형태도 아니게 된다. 또한, 객체 스스로 상태값을 변경하는 것이 아니고, 외부에서 상태값을 변경할 수 있는(`public`이기 때문) `위험성` 생길 수 있다.

또한, `getter` 를 남용하게 되면, **디미터의 법칙**을 위반할 가능성도 생기고, 가독성이 떨어진다.

포비(박재성님)는 이렇게 말했다.

```
상태를 가지는 객체를 추가했다면 객체가 제대로 된 역할을 하도록 구현해야 한다.
객체가 로직을 구현하도록 해야한다.
상태 데이터를 꺼내 로직을 처리하도록 구현하지 말고 객체에 메시지를 보내 일을 하도록 리팩토링한다.
```

## 📌 **디미터의 법칙**

객체 구조의 변화와 불안정이 자주 발생되는 것을 보고 객체 연결에 관한 정보를 지닌 코드는 불안정하다는 사실을 접하면서 만들어짐. **객체들의 협력 경로를 제한하면 결합도를 효과적으로 낮출 수 있다**. 

**Don’t Talk to Strangers** : 긴 객체 구조의 경로를 따라 멀리 떨어져 있는 간접적인 객체에 메시지를 보내는 설계는 피하라, 이러한 설계는 일반적으로 불안정한 지점으로, 객체 구조의 변화에 부서지기 쉽다. 그렇다면 어떤 객체들에게만 메시지를 보내야 하는가?

- this(or self) 객체
- 메소드의 매개변수
- this의 속성
- this의 속성인 컬렌션의 요소
- 메소드 내에서 생성된 객체

객체는 다른 객체를 탐색해 뭔가를 일어나게 해서는 안 된다. ‘묻지 말고 말하라’ 그저 메시지를 주고 받을 뿐이다. **객체는 내부적으로 보유하고 있거나 메시지를 통해 확보한 정보만 가지고 의사 결정을 내려야 한다.**

모듈은 자신이 조작하는 객체의 속사정을 모야라 한ㄷ다. 객체는 자료를 숨기고 함수를 공개한다. 즉, 객체는 조회 함수로 내부 구조를 공개하면 안된다는 의미다.

- **협력 경로를 제한**하는 것은 무엇인가?

  협력하는 객체의 내부 구조에 대한 결합으로 인해 발생하는 문제를 해결하기 위해 제안된 것이 **디미터의 법칙**이다. 객체의 내부 구조에 강하게 결합되지 않도록 협력 경로를 제한하라. 객체의 모든 메서드는 다음에 해당하는 메서드만을 호출하는 것이 좋다.

  - 자신
  - 메서드로 넘어온 인자
  - 자신이 생성한 객체
  - 직접 포함하고 있는 객체

  이는 함수를 호출하는 클래스의 응답집합 크기를 줄일 수 있기 때문에 좀 더 에러가 적은 클래스들을 만들 수 있다.

- 객체의 내부 구조를 묻지 말고, **말해라(시켜라)**

객체가 자기 자신을 책임지는 자율적인 존재여야 한다. 정보를 처리하는 데 필요한 책임을 정보를 알고 있는 객체에게 할당하기 때문에 응집도가 높은 객체가 만들어진다.

**객체의 내부 구조를 묻는 메시지가 아니라 수신자에게 무언가를 시키는 메시지가 더 좋은 메시지**이다.

- 그렇다면 디미터의 법칙을 위반한 코드는 무엇인가?

  법칙을 따르지 않으면 **메시지 체인(Message Chains)(or 기차 충돌)**이란 악취가 나게 된다. `object.getChild().getContent().getItem().getTitle();` → `getter` 가 줄줄이 이어진 모습이 기차와 닮아서 ‘열차 전복’코드라고도 한다.

  원거리의 간접적인 객체에 메시지를 보내기 위해 객체의 연결 경로를 따라 더 멀리 순회한다. 이러한 설계는 객체들이 어떻게 연결되어 있는지를 나타내는 특정한 구조와 결합된다. **프로그램 순회의 경로가 길어질수록 프로그램은 더 불안정**해진다. 그 이유는 객체 구조(연결)는 변경될 수 있기 때문이다.

  그렇다면 아래의 코드는 디미터의 법칙을 위반한 코드인가?

  ```java
  IntStream.of(1, 15, 20, 3, 9)
      .filter(x -> x > 10)
      .distinct()
      .count();
  ```

  **아니다**. 위 코드에서 `of`, `filter`, `distinct` 메서드는 모두 `Instream` 이라는 동일한 클래스의 인스턴스를 반환한다. 즉, 이들은 `Instream` 의 인스턴스를 또다른 `Instream`의 인스턴스로 변환한 것이다. 따라서 디미터 법칙을 위반한 것이 아니다. 디미터 법칙은 결합도가 문제가 되는 것은 객체의 내부 구조가 외부로 노출되는 경우를 말한다. `Instream` 내부 구조가 외부로 노출되지 않았다. 단지 `Instream` 간 변화만 존재할 뿐이지, 객체를 둘러싸고 있는 캡슐은 그대로 유지된다.

- 메시지 체인의 리팩토링은 어떻게 할까?

  클라이언트가 한 객체의 제 2의 객체를 요청하면, 제 2의 객체가 제 3의 객체를 요청하고, 제 3의 객체는 제 4의 객체를 요청하는 연쇄적 요청이 발생한다. 이런 요청의 왕래로 인해 클라이언트는 그 왕래 체제에 구속된다. 관계에 수정이 발생한다면 모든 왕래 체제에 수정이 필요하다. 이는 매울 비효율적이다.

  이럴 때는 **대리 객체 은폐**를 실시해야 한다. 이 기법은 체인을 구성하는 모든 객체에 적용할 수 있지만, 그렇게 하면 모든 중간 객체가 중개 메서드로 변해 과잉 중개 메서드의 구린내를 풍기게 된다. 차라리 결과 **객체가 어느 대상에 사용되는지를 알아내는 방법**이 더 낫다. 그렇게 알아낸 객체가 사용되는 코드 부분을 메서드 추출을 통해 별도의 메서드로 빼낸 후 메서드 이동을 실시해 체인 아래로 밀어낼 수 있는지 여부를 검사해야 한다. 만약 체인에 속한 객체 중 한 객체의 여러 클라이언트가 나머지 객체들에 왕래한다면 그 기능을 수행하는 메서드를 추가하면 된다.

## 📌 `getter`를 무조건 사용하지 말아라?

**아니다.** getter를 무조건 사용하지 않고 코딩하는 것은 불가능할 것이다. 출력을 위한 값 등 순수한 프로퍼티를 가져오기 위해서라면 어느정도 `getter` 를 사용해도 된다.

그러나,  `Collection` 인터페이스를 사용하는 경우 외부에서 `getter` 로 얻은 값을 통해 상태값을 변경할 수 있다. 이는 매우 위험한 상황을 초래하므로 외부에서 변경하지 못하도록 하는 게 좋다.

`Unmodifiable Collecion` 을 이용해 **방어적 복사**를 하자. 생성자의 인자로 받은 객체의 복사본을 만들어 내부 필드를 초기화하거나, `getter` 메서드에서 내부의 객체를 반환할 때, 객체의 복사본을 만들어 반환하는 것이다. 방어적 복사를 사용할 경우, 외부에서 객체를 변경해도 내부의 객체는 변경되지 않는다.

- 아래와 같이 생성자에서 인자를 받으면서 `new ArrayList<>()` 를 통해 `names` 를 초기화 한다.

```java
import java.util.ArrayList;
import java.util.List;

public class Name {
    private final String name;

    public Name(String name) {
        this.name = name;
    }
}

public class Names {
    private final List<Name> names;

    public Names(List<Name> names) {
        this.names = new ArrayList<>(names);
    }
}
```

- `new ArrayList<>()` 를 통해 원본과의 주소 공유를 끊어냈기 때문에 `crewNames` 의 `names`값들이 변하지 않는다.

```java
import java.util.ArrayList;
import java.util.List;

public class Application {
    public static void main(String[] args) {
        List<Name> originalNames = new ArrayList<>();
        originalNames.add(new Name("Pobi"));
        originalNames.add(new Name("Json"));

        Names crewNames = new Names(originalNames);
        originalNames.add(new Name("Ykm"));
    }
}
```

**방어적 복사는 깊은 복사가 아니다.** 서로 일급 컬렉션의 주소만 공유하지 않을 뿐, 내부 값들의 주소는 공유되고 있다. 내부 요소들의 주소가 공유되고 있으므로 원본의 내부 요소를 바꾸면 복사본의 값도 바뀌기 때문에 깊은 복사가 아니다.

## 📌 Unmodifiable Collection

외부에서 변경 시 예외 처리되기 때문에 안정하게 보장할 수 있다.

`unmodifiableList()` 메서드를 통해 리턴되는 리스트는 읽기 전용이다. 즉, `set()`, `add()`, `addAll()` 등의 메서드는 리스트에 변경을 주는 메서드 이기 때문에 사용시 `UnsupportedOperationException` 이 발생한다.

하지만 불변을 보장하지는 않는다. 원본 자체의 수정이 발생하면 복사본의 값도 변경된다.

그렇다면 언제 **Unmodifiable Collection**과 **방어적 복사**를 해야하는가?

객체 내부의 값을 외부로부터 보호하는것이 핵심이다.

- 생성자의 인자로 객체를 받았을 때 외부에서 넘겨졌던 객체를 변경해도 내부 객체는 변하지 않아야 한다. 따라서 **방어적 복사**를 해야 한다.
- `getter` 를 통해 객체를 넘겨줄 때 내부 객체는 변하지 않아야 한다. **방어적 복사**를 이용해도 좋고 객체 값을 읽기 전용으로 넘겨주는 **Unmodifiable Collection** 도 좋다.

# 2. 테스트 하기 어려운 코드를 테스트 하기

## 📌 메서드 시그니처를 수정해 테스트하기 좋은 메서드로 만들자

3주 차 미션의 `Lotto` 클래스 이다.

```java
import camp.nextstep.edu.missionutils.Randoms;

public class Lotto {
    private List<Integer> numbers;

    public Lotto() {
        this.numbers = Randoms.pickUniqueNumbersInRange(1, 45, 6);
    }
}
——————

public class LottoMachine {
    public void execute() {
        Lotto lotto = new Lotto();
    }
}
```

생성자에서 `numbers` 에 `Randoms.pickUniqueNumbersInRange(1, 45, 6)` 을 이용해 값을 생성해 주고 있다. 이러한 경우 매번 `Lotto` 객체를 생성할 때 값이 달라지므로 원하는 값이 들어가는지 테스트를 작성하기가 어려울 것이다.

올바른 로또 번호가 생성되는 것을 테스트하기 어려우면, 테스트하기 어려운 것을 클래스 내부가 아닌 외부로 분리하는 시도를 해 본다.

굳이 `Lotto` 에서 값을 생성할 필요가 없다. 로또를 자동 생성해 주는 `LottoMachine` 클래스를 만들고 자동 생성할 역할(책임)을 부여한다. 이러면 `Lotto` 가 올바른 값을 가지는지 테스트를 할 수 있다.

```java
public class Lotto {
    private List<Integer> numbers;

    public Lotto(List<Integer> numbers) {
        this.numbers = numbers;
    }
}
——————
import camp.nextstep.edu.missionutils.Randoms;

public class LottoMachine {
    public void execute() {
        List<Integer> numbers = Randoms.pickUniqueNumbersInRange(1, 45, 6);

        Lotto lotto = new Lotto(numbers);
    }
}
```

이렇게 되면 `Lotto` 클래스의 테스트를 작성하기가 쉬워진다. 또한 `Lotto`객체를  통해 `LottoMachine` 에서 올바른 값을 생성했는지 확인이 가능하므로 `LottoMachine` 테스트 작성도 가능해 진다.

메서드의 시그니처를 수정하는 것만으로 테스트하기 좋은 메서드로 만들 수 있다는 것을 알 수 있다. 하지만 모든 문제가 해결된 것은 아니다. 단지 랜덤값 생성을 외부로 분리한 것이지 실제로 `execute()` 를 호출하면 랜덤값을 주입해 주어야 하기 때문에, `LottoMachine` 은 랜덤값에 의존하게 된다.

이러한 의존을 어디에 두고, 어떻게 관리해야 할지 고민을 해야한다. 의존을 조금 더 줄일 수 있는 방법 중 인터페이스를 분리하는 방법이 있다.

# 3. IllegalArgumentException vs. IllegalStateException

## 📌 RuntimeException

실행 중에 발생하며 시스템 환경적으로나 입력 값이 잘못된 경우, 혹은 의도적으로 프로그래머가 잡아내기 위한 조건등에 부합할 때 발생(`throw`) 되게 만든다.

그렇다면 `unchecked exception` 은 무엇일까?

`Exception`은 `Checked Exception`과 `Unchecked Exception`으로 나눌수 있다.

- **Checked Exception** : 예외 처리를 필수적으로 해야하는 경우로 `Exception`을 상속 받는다.

  - 반드시 `try catch`로 감싸주던가, 해당 메소드에 `throw Exception`을 달아서 예외를 다시 호출자에게 미루는 방법을 사용해야 한다.

- **Unchecked Exception** : 예외 처리가 필수적이지 않은 경우로 `RuntimeException`을 상속받는다.
- `try cath` 또는 `throw Exception`을 강제하지 않는다.

## 📌 IllegalArgumentException

`RuntimeExcpetion` 을 상속받은 예외. `Unchecked Exception` 이고 처리를 해주지 않아도 컴파일에는 문제가 없다. **부정한 인수, 부적절한 인수를 메서드에 건네준 것을 나타내기 위해서 발생**된다. 즉, 매개변수가 의도하지 않은 상황을 유발시킬 때 또는 `null이 아닌 인자의 값`이 잘못되었을 때 사용한다.

예시는 다음과 같다.

- 양수값을 넣어줘야 하는데 음수값을 넣어준 경우
- 3이상의 값을 넣어줘야 하는데 3 미만의 값을 입력한 경우

## 📌 IllegalStateException

`RuntimeExcpetion` 을 상속받은 예외. `Unchecked Exception` 이고 처리를 해주지 않아도 컴파일에는 문제가 없다. 부정 또는 부적절한 때에 메서드가 불려 간 것을 나타낸다. 즉, Java 환경 또는 Java Aplication은 요구된 operation에 적절한 상태가 아니다는 것을 알려준다. 즉, 메소드를 호출하기 위한 상태가 아닐 때 사용한다.

예시는 다음과 같다.

- 사용하려는 메소드의 파라미터값이 아직 생성되지 않은 경우

<br>

> 참고 URL
>
> > [getter를 사용하는 대신 객체에 메시지를 보내자](https://tecoble.techcourse.co.kr/post/2020-04-28-ask-instead-of-getter/)
> >
> > [디미터 법칙](https://johngrib.github.io/wiki/law-of-demeter/)
> >
> > [방어적 복사와 Unmodifiable Collection](https://tecoble.techcourse.co.kr/post/2021-04-26-defensive-copy-vs-unmodifiable/)
> >
> > [메서드 시그니처를 수정하여 테스트하기 좋은 메서드로 만들기](https://tecoble.techcourse.co.kr/post/2020-05-07-appropriate_method_for_test_by_parameter/)
> >
> > [RuntimeException란 무엇인가?](https://nhj12311.tistory.com/204)
> >
> > [IllegalArgumentException vs IllegalStateException](https://velog.io/@injoon2019/IllegalArgumentException-vs-IllegalStateException)