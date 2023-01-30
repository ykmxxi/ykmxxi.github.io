---
title: "[우테코-프리코스] 3주 차 회고"
categories: [wooteco]
tag: [wooteco]
author_profile: false
sidebar:
    nav: "counts"
---

# 1. 제이슨 강의

## 📌 요구사항을 읽고 기능 목록 작성

- 문서를 꼼꼼히 읽는다, 미션의 README 파일 중 요구사항들을 그대로 복붙해와 천천히 읽으며 구현해야할 기능 목록을 작성한다.
  - 이를 기반으로 skeleton code를 작성한다. 실제로 건물을 올릴 때도 뼈대를 먼저 잡고 한층 씩 만들어 나간다.
- build.gradle 에서 dependencies 에 추가한 내용들이 repositories 에 저장되어 자바 프로젝트의 틀을 만들어줌

```java
package baseball;

public class Application {
		public static void main(String[] args) {
				System.out.println("Hello, World!");
		}
}
```

- package 는 이 클래스가 어느 프로그램의 클래스 인지 알려주는 역할

## 📌 프로그램 시작

- model(domain 이라고도 사용) : 실제 다루고자 하는 서비스의 비즈니스 로직이 들어있는 영역, 계산을 하는 등 행위들이 담기는 곳
  - `baseball/model(domain)` : 숫자 야구 게임의 비즈니스 로직이 들어있는 영역
- java는 클래스, 메소드, 변수를 어디까지 오픈할지 설정을 한다 by 접근 제어자 (public, private, etc…)
- 클래스는 기능(메소드)들의 모음, 이 모음집 클래스를 new 연산자를 사용해 변수에 저장(복사)하면 해당 변수를 인스턴스라고 한다.

## 📌 객체지향 프로그래밍

1. 기능을 가지고 있는 클래스를 인스턴스화(=객체)한다.
2. 필요한 기능을 (역할에 맞는)인스턴스가 수행하게 한다.
3. 각 결과를 종합한다.

## 📌 테스트코드 리팩토링

- `@ParameterizedTest` 를 이용해 하드코딩을 줄이자

```java
class RefereeTest {
    private static final List<Integer> ANSWER = Arrays.asList(1, 2, 3);
    private Referee referee;

    @BeforeEach
    void beforeEach() {
        referee = new Referee();
    }

    @ParameterizedTest
    @CsvSource({"1, 2, 3, 0볼 3스트라이크", "7, 8, 9, 낫싱", "2, 3, 1, 3볼 0스트라이크", "1, 3, 2, 2볼 1스트라이크"})
    void compare(int number1, int number2, int number3, String expected) {
        String actual = referee.compare(ANSWER, Arrays.asList(number1, number2, number3));
        assertThat(actual).isEqualTo(expected);
    }

    @Test
    void 스트라이크3() {
        String result = referee.compare(Arrays.asList(1, 2, 3), Arrays.asList(1, 2, 3));
        assertThat(result).isEqualTo("0볼 3스트라이크");
    }

    @Test
    void 낫싱() {
        String result = referee.compare(Arrays.asList(1, 2, 3), Arrays.asList(4, 5, 6));
        assertThat(result).isEqualTo("낫싱");
    }
}
```

# 2. 객체지향? by 객체지향의 사실과 오해 (조영호 저)

대부분의 사람들은 “객체지향이란 실세계를 직접적이고 직관적으로 모델링할 수 있는 패러다임”이라는 설명과 마주한다. 하지만 실세계의 모방이라는 개념은 객체지향의 기반을 이루는 철학적인 개념을 설명하는데 유용할 뿐, 실용적인 관점에서는 적합하지 않다.

객체지향의 목표는 실세계를 모방하는 것이 아니다. **모방이 아닌 새로운 세계를 창조하는 것이다.** 단순히 SW 안으로 실세계를 옮겨 담는 것이 아니라 고객과 사용자를 만족시킬 수 있는 신세계를 창조하는 것이다.

- 객체를 현실 세계의 생명체에 비유하는 것은 ‘상태’와 ‘행위’를 ‘캡슐화’하는 SW 객체의 ‘자율성’을 설명
- 암묵적인 약속과 명시적인 계약을 기반으로 렵력하며 목표를 달성해 나가는 과정은 ‘메시지’를 주고받으며 공동의 목표를 달성하기 위해 ‘협력’하는 객체들의 관계와 비슷

일상에서 발생하는 대부분의 문제는 개인 혼자만의 힘으로 해결하기 버거울 정도로 복잡하다. 이를 위해 ‘협력’이 필요하고, 협력을 위해서는 ‘요청’이 필요하다. 아래 예시로 살펴보자.

손님 → (커피를 주문) → 캐시어 → (커피를 제조하라) → 바리스타

손님 ← (커피 완성) ← 캐시어 ← (커피 완성) ← 바리스타

손님은 캐시어에게 커피를 주문한다는 ‘요청’을 한다. 캐시어가 요청을 받고 커피를 만드는 문제를 해결하기 위해 또 다시 바리스타에게 ‘요청’을 한다. 바리스타는 요청을 받아 문제를 해결하고 이를 캐시어에게 알려준다. 캐시어는 이를 손님에게 알려주고 문제는 해결된다. **요청과 응답을 통해 다른 사람과 협력(collaboration)할 수 있는 능력은 복잡한 문제를 해결할 수 있는 공동체를 형성할 수 있게 만든다.**

- 역할 : 협력하는 과정 속에서 특정한 임무를 부여받음, 어떤 협력에 참여하는 특정한 사람이 협력 안에서 차지하는 책임이나 임무를 의마한다.
  - 손님은 커피를 주문할 역할(책임)이 있고, 캐시어는 주문을 받을 역할(책임)이 있고, 바리스타는 커피를 만들 역할(책임)이 있다.
- 책임 : 역할이라는 단어는 의미적으로 책임이라는 개념을 내포한다. 특정한 역할은 특정한 책임을 암시한다.