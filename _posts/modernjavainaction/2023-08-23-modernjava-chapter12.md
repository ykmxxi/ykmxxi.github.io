---
title: "[모던 자바 인 액션] 12. 새로운 날짜와 시간 API"
categories: [Modern Java in Action]
tag: ["Java"]
author_profile: false
sidebar:
    nav: "counts"

---

> Reference
>
> [모던 자바 인 액션: 람다, 스트림, 함수형, 리액티브 프로그래밍으로 새로워진 자바 마스터하기](https://www.yes24.com/Product/Goods/77125987?pid=123487&cosemkid=go15646485055614872&gclid=Cj0KCQjw2qKmBhCfARIsAFy8buL-QGAkh5m5YK8mj5vAgRgCKdSRIHzDDWdAYOLPFSuQNlzVVcFFdbYaAsh1EALw_wcB)

Java 8에서는 지금까지의 날짜와 시간 문제를 개선하는 새로운 날짜와 시간 API를 제공한다

Java 1.0은 `java.util.Date` 클래스 하나로 관련 기능을 제공했다. 이 API는 이름과 달리 특정 시점을 날짜가 아닌 밀리초(ms) 단위로 표현했고, 여러 모호한 설계로 유용성이 떨어졌다.

결국 Java 1.1에서 새로운 `java.util.Calendar` 클래스를 대안으로 제공했지만 역시 쉽게 에러를 일으키는 설계 문제를 갖고 있었다.

> Date와 Calendar 클래스는 **가변(mutable)** 클래스로 유지보수에 어려움을 준다

Java 8은 새로운 `java.time` 패키지를 만들어 새로운 날짜와 시간 API를 제공한다.

<br>

# 📌 LocalDate, LocalTime, Instant, Duration, Period 클래스

`java.time` 패키지는 LocalDate, LocalTime, Instant, Duration, Period 등 새로운 **불변 클래스**를 제공한다

## LocalDate & LocalTime 클래스

- **LocalDate**: 시간을 제외한 날짜를 표현하는 불변 객체
  - LocalDate 객체는 어떤 시간대 정보도 포함하지 않음
  - `LocalDate.of()`: 정적 팩토리 메서드로 LocalDate 인스턴스 생성 가능
  - 연도, 달, 요일 등을 반환하는 메서드를 제공
  - `LocalDate.now()`: 정적 팩토리 메서드로 시스템 시계의 정보를 이용해 현재 날짜 정보를 얻음

<center><img src="/assets/images/modernjavainaction/ch12/1.png"></center>

```java
LocalDate date = LocalDate.of(2023, 8, 22);

int year = date.getYear();
Month month = date.getMonth();
int dayOfMonth = date.getDayOfMonth();

DayOfWeek dayOfWeek = date.getDayOfWeek();
int length = date.lengthOfMonth(); // 해당 월의 수 (30 or 31 or 28)
boolean leapYear = date.isLeapYear(); // 윤년 여부

LocalDate now = LocalDate.now();
```

<center><img src="/assets/images/modernjavainaction/ch12/2.png"></center>

- **LocalTime**: 시간을 표현하는 불변 객체
  - 오버로드 버전의 두 가지 정적 메서드 `of()`를 제공
  - `LocalTime.of(int hour, int minute)`
  - `LocalTime.of(int hour, int minute, int second)`

```java
// 정적 팩토리 메서드 of()의 두 가지 오버로드 버전을 제공
LocalTime timeWithoutSec = LocalTime.of(15, 46);
LocalTime timeWithSec = LocalTime.of(15, 46, 33);

int hour = timeWithSec.getHour(); // 15
int minute = timeWithSec.getMinute(); // 46
int second = timeWithSec.getSecond(); // 33
// int second1 = timeWithoutSec.getSecond(); // 초가 없으면 0을 반환
```

- `parse()`: 날짜와 시간을 나타내는 두 객체 모두에 존재하는 정적 메서드로, 문자열을 받아 인스턴스를 생성
  - 형식에 맞지 않아 파싱할 수 없으면 `DateTimeParseException`을 일으킴
  - `RuntimeException`을 상속받은 예외

```java
// parse() 정적 메서드: 지정한 문자열을 파싱해 인스턴스 생성
LocalDate parseDate = LocalDate.parse("2023-12-12");
LocalTime parseTime = LocalTime.parse("11:22:33");
```

## LocalDateTime 클래스: 날짜와 시간 조합

- **LocalDateTime**: LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스
  - 날짜와 시간을 모두 표현할 수 있는 클래스

```java
LocalDate date = LocalDate.of(2023, 8, 22);
LocalTime time = LocalTime.of(11, 22, 33);

LocalDateTime dateTime = LocalDateTime.of(date, time);

LocalDate localDate = dateTime.toLocalDate(); // 2023-08-22
LocalTime localTime = dateTime.toLocalTime(); // 11-22-33

LocalDateTime dateTime1 = date.atTime(time);
LocalDateTime dateTime2 = date.atTime(11, 22, 33);
LocalDateTime dateTime3 = time.atDate(date);

LocalDateTime now = LocalDateTime.now();
```

## Instant 클래스: 기계의 날짜와 시간

사람은 보통 주, 날짜, 시간, 분으로 시간을 계산한다. 하지만 기계는 연속도니 시간에서 특정 지점을 하나의 큰 수로 표현하는 것이 가장 자연스러운 시간 표현 방법이다.

- **Instant**: 유닉스 에포크 시간(Unix epoch time)을 기준으로 특정 지점까지의 시간을 초로 표현
  - 유닉스 에포크 시간: 1970년 1월 1시 0시 0분 0초 UTC
  - Instant 클래스는 나노초(10억분의 1초)의 정밀도를 제공
  - `Instant.ofEpochSecond()`: Instant 객체를 생성하는 정적 팩토리 메서드

```java
Instant instant = Instant.ofEpochSecond(3); // 에포크 기준 3초 뒤: 1970-01-01T00:00:03Z

Instant instant1 = Instant.ofEpochSecond(3, 0); // 1970-01-01T00:00:03Z
// 에포크 기준 2초 뒤 + 1억 나노초 뒤 = 에포크 기준 3초 뒤
Instant instant2 = Instant.ofEpochSecond(2, 1_000_000_000); // 1970-01-01T00:00:03Z

Instant now = Instant.now();
```

Instant 클래스는 기계 전용의 유틸리티이다. 즉, 초와 나노초 정보를 포함해 사람이 읽을 수 있는 시간 정보를 제공하지 않는다. 예를 들어 다음 코드를 컴파일하면 예외가 발생한다

```java
int day = Instant.now().get(ChronoField.DAY_OF_MONTH);
```

<center><img src="/assets/images/modernjavainaction/ch12/3.png"></center>

## Duration & Period 클래스

- **Temporal 인터페이스**: 특정 시간을 모델링하는 객체의 값을 어떻게 읽고 조작할지 정의한 인터페이스

지금까지 살펴본 LocalDate, LocalTime, LocalDateTime, Instant는 모두 Temporal 인터페이스를 구현한다.

- **Duration**: 두 시간 객체 사이의 지속시간을 나타내는 객체
  - `Duration.between()`: 정적 팩토리 메서드로 두 시간 객체 사이의 지속시간을 생성
  - 인자로 두 개의 LocalTime 또는 두 개의 LocalDateTime 또는 두 개의 Instant
  - 초와 나노초로 시간 단위를 표현해 LocalDate는 사용할 수 없다

```java
LocalTime time1 = LocalTime.of(11, 22, 33);
LocalTime time2 = LocalTime.of(11, 22, 44);
Duration between1 = Duration.between(time1, time2);

LocalDateTime dateTime1 = LocalDateTime.now();
LocalDateTime dateTime2 = LocalDateTime.now(ZoneId.systemDefault());
Duration between2 = Duration.between(dateTime1, dateTime2);
```

- **Period**: 사람의 관점으로 두 시간 객체 사이의 지속시간을 나타내는 객체
  - 년, 월, 일로 시간을 표현
  - 두 개의 LocalDate를 인자로 받음

```java
Period tenDays = Period.between(LocalDate.of(2023, 8, 1), LocalDate.of(2023, 8, 11);

Period threeDays = Period.ofDays(3); // P3D
Period threeWeeks = Period.ofWeeks(3); // P21D
Period twoYearsSixMonthsOneDay = Period.of(2, 6, 1); // P2Y6M1D
```

지금까지 살펴본 LocalDate, LocalTime, LocalDateTime, Instant 등은 모두 불변 클래스이다. 불변 클래스는 함수형 프로그래밍 그리고 스레드 안전성과 도메인 모델의 일관성을 유지하는 데 좋은 특징이다.

<br>

# 📌 날짜 조정, 파싱, 포매팅

새로운 날짜와 시간 API에서는 변경된 객체 버전을 만들 수 있는 메서드를 제공한다. 예를 들어 LocalDate 객체에 3일을 더해야 하는 상황이 발생하면, 메서드 호출을 통해 3일을 더해줄 수 있다.

`withAttribute()`로 기존의 LocalDate를 바꾼 버전을 직접 간단하게 만들 수 있다. 이 방법은 절대적인 방식으로 LocalDate의 속성을 바꾼다.

- 객체를 바꾸는 것이 아니라 필드를 갱신한 복사본을 만든다는 사실을 기억하자
- `date1.with(ChronoField.MONTH_OF_YEAR, 1);`: 새로운 인스턴스를 생성하므로 변수에 할당하지 않으면 아무런 일도 일어나지 않음. 즉, 원본인 date1에는 아무런 영향을 주지 않는다.

```java
LocalDate date1 = LocalDate.of(2023, 8, 22);

LocalDate date2 = date1.withYear(2000); // 2000-08-22
LocalDate date3 = date1.withMonth(1); // 2023-01-22
LocalDate date4 = date1.withDayOfMonth(11); // 2023-08-11

LocalDate date5 = date1.with(ChronoField.MONTH_OF_YEAR, 1); // 2023-01-22
```

또 선언형으로 LocalDate를 사용하는 방법도 있다. 즉, 상대적인 방식으로 LocalDate의 속성을 바꾼다

```java
LocalDate date6 = date1.plusWeeks(1); // 2023-08-29
LocalDate date7 = date1.minusDays(10); // 2023-08-12
LocalDate date8 = date1.minus(6, ChronoUnit.MONTHS);// 2023-02-22
```

## TemporalAdjusters 사용하기

좀 더 복잡한 날짜 조정 기능이 필요할 땐 오버로드된 버전의 with 메서드에 좀 더 다양한 동작을 수행할 수 있도록 하는 기능을 제공하는 `TemporalAdjuster`를 전달하는 방법을 사용할 수 있다.

날짜와 시간 API는 다양한 상황에서 사용할 수 있도록 다양한 `TemporalAdjuster`를 제공한다.

`TemporalAdjuster`는 인터페이스며, `TemporalAdjusters`는 여러 `TemporalAdjuster`를 반환하는 정적 팩토리 메서드를 포함하는 클래스이므로 서로 혼동하지 않도록 주의해야 한다.

`TemporalAdjusters` 클래스에는 다양한 팩토리 메서드가 존재한다.

```java
import static java.time.temporal.TemporalAdjusters.lastDayOfMonth;
import static java.time.temporal.TemporalAdjusters.nextOrSame;

import java.time.DayOfWeek;
import java.time.LocalDate;

LocalDate date1 = LocalDate.of(2023, 8, 23); // 2023-08-23 수요일

LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY)); // 2023-08-27 일요일

LocalDate date3 = date2.with(lastDayOfMonth()); // 2023-08-31
```

<center><img src="/assets/images/modernjavainaction/ch12/4.png"></center>

이렇게 `TemporalAdjuster`를 이용하면 복잡한 날짜 조정 기능을 직관적으로 해결하 수 있다. 그뿐만 아니라 정의되어 있지 않을 때는 비교적 쉽게 커스텀 `TemporalAdjuster`를 구현할 수 있다

```java
@FunctionalInterface
public interface TemporalAdjuster {
	Temporal adjustInto(Temporal temporal);
}
```

`TemporalAdjuster` 인터페이스 구현은 Temporal 객체를 어떻게 다른 Temporal 객체로 변환할지 정의한다. 즉, `TemporalAdjuster` 인터페이스를 `UnaryOperator<Temporal>`과 같은 형식으로 간주할 수 있다.

```java
/**
 * 커스텀 TemporalAdjuster 구현
 * 날짜를 하루씩 다음날로 바꿈
 * 토요일과 일요일은 건너뜀
 * 월 -> 화 -> ... -> 금 -> 월
 */
public class NextWorkingDay implements TemporalAdjuster {

	@Override public Temporal adjustInto(Temporal temporal) {
		DayOfWeek of = DayOfWeek.of(temporal.get(ChronoField.DAY_OF_WEEK)); // 현재 날짜 읽기

		int dayToAdd = 1; // 보통은 하루 추가
		if (of == DayOfWeek.FRIDAY) { // 금요일이면
			dayToAdd = 3;
		} else if (of == DayOfWeek.SATURDAY) { // 토요일이면
			dayToAdd = 2;
		}
		return temporal.plus(dayToAdd, ChronoUnit.DAYS);
	}

}
```

## 날짜와 시간 객체 출력과 파싱

날짜와 시간 관련 작업에서 포매팅과 파싱은 서로 떨어질 수 없는 관계다.

Java는 전용 패키지인 `java.time.format`을 제공하고 그 중 가장 중요한 클래스가 `DateTimeFormatter`다

- **DateTimeFormatter**: 정적 팩토리 메서드와 상수를 이용해 쉽게 포매터를 만들 수 있는 클래스
  - BASIC_ISO_DATE, ISO_LOCAL_DATE
  - 날짜나 시간을 특정 형식의 문자열로 만들 수 있음

`DateTimeFormatter`는 기존의 DateFormat 클래스와 달리 스레드에서 안전하게 사용할 수 있다. 아래 예제처럼 특정 패턴으로 포매터를 만들 수 있는 정적 팩토리 메서드도 제공한다

```java
LocalDate date = LocalDate.of(2023, 8, 23);

String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE); // 20230823
String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE); // 2023-08-23

// parse()를 이용해 문자열을 날짜 객체로도 만들 수 있음
LocalDate date1 = LocalDate.parse("20230823", DateTimeFormatter.BASIC_ISO_DATE);
LocalDate date2 = LocalDate.parse("2023-08-23", DateTimeFormatter.ISO_LOCAL_DATE);

// 패턴으로 날짜 생성
DateTimeFormatter pattern = DateTimeFormatter.ofPattern("yyyy:MM:dd");
String formattedDate = date.format(pattern); // 2023:08:23
LocalDate date3 = LocalDate.parse(formattedDate, pattern); // 2023-08-23

// Locale 정보로 포매터 생성
DateTimeFormatter koreaFormatter = DateTimeFormatter.ofPattern("yyyy년 MM월 dd일", Locale.KOREA);
String koreaDate = date.format(koreaFormatter); // 2023년 08월 23일
```

<br>

# 📌 다양한 시간대와 캘린더 활용 방법

새로운 날짜와 시간 API의 장점은 시간대를 간단하게 처리할 수 있다는 점이다

기존의 `java.util.TimeZone`을 대체하는 `java.time.ZoneId` 클래스가 새롭게 제공된다. 새로운 클래스를 이용하면 유럽의 서머타임(DST) 같은 복잡한 사항이 자동으로 처리된다. ZoneId도 불변 클래스 중 하나다.

## 시간대 사용하기

표준 시간이 같은 지역을 묶어 **시간대(time zone)** 규칙 집합을 정의한다. ZoneRules 클래스는 약 40개 정도의 시간대가 존재하면, ZoneId의 `getRules()`를 이용해 해당 시간대의 규정을 획득할 수 있다.

```java
ZoneId romeZone = ZoneId.of("Europe/Rome");
ZoneId seoulZone = ZoneId.of("Asia/Seoul");
```

- 지역 ID: `지역/도시` 로 이루어지며 IANA Time Zone Database에서 제공하는 지역 집합 정보를 사용한다
- ZoneId 객체를 얻은 다음 LocalDate, LocalDateTime, Instant 등을 이용해 ZonedDateTime 인스턴스로 변환할 수 있다. ZonedDateTime은 지정한 시간대에 상대적인 시점을 표현한다
  - ZonedDateTime = LocalDateTime + ZoneId

```java
ZoneId seoulZone = ZoneId.of("Asia/Seoul");
LocalDate date = LocalDate.of(2023, 8, 23);

// 2023-08-23T00:00+09:00[Asia/Seoul]
ZonedDateTime seoulDate = date.atStartOfDay(seoulZone);
```

ZoneId를 이용해 LocalDateTime을 Instant로 바꾸는 방법도 있다. 기존의 Date 클래스를 처리하는 코드를 사용해야 하는 상황이 있을 수 있어 Instant로 작업하는 것이 유리하다

```java
Instant now = Instant.now();

// 2023-08-23T14:56:08.011375
LocalDateTime nowSeoul = LocalDateTime.ofInstant(now, seoulZone);
```

## UTC/Greenwich 기준의 고정 오프셋

종종 UTC(Universal Time Coordinated, 협정 세계시)/GMT(그리니치 표준시)를 기준으로 시간대를 표현하기도 한다. 예를 들어 뉴욕은 런던보다 5시간 느리다라고 표현할 수 있다

ZoneId의 서브클래스인 ZoneOffset 클래스로 런던의 그리니치 0도 자오선과 시간값의 차이를 표현할 수 있다.

```java
ZoneOffset newYorkOffset = ZoneOffset.of("-05:00");
```

하지만 위 ZoneOffset은 서머타임을 제대로 처리할 수 없어 권장하지 않는 방식이다.

또 ISO-8601 캘린더 시스템에서 정의하는 UTC/GMT와 오프셋으로 날짜와 시간을 표현하는 OffsetDateTime을 만들수도 있다.

```java
ZoneOffset newYorkOffset = ZoneOffset.of("-05:00");

LocalDateTime date = LocalDateTime.of(2023, 8, 23);
OffsetDateTime dateTimeInNewYork = OffsetDateTime.of(date, newYorkOffset);
```

새로운 날짜와 시간 API는 ISO 캘린더 시스템에 기반하지 않은 정보도 처리할 수 있는 기능을 제공한다

## 대안 캘린더 시스템 사용하기

Java 8에서는 4개의 캘린더 시스템을 제공한다

- ThaiBuddhistDate: 태국의 불교 달력을 기반으로 하는 클래스
- MinguoDate: 중화민국(대만)의 민국 달력을 나타내는 클래스
- JapaneseDate: 일본의 달력을 표현하는 클래스
- HijrahDate: 히지라(이슬람) 달력을 기반으로 하는 클래스

위 4개의 클래스와 LocalDate 클래스는 ChronoLocalDate 인터페이스를 구현하는데, ChronoLocalDate는 임의의 연대기에서 특정 날짜를 표현할 수 있는 기능을 제공하는 인터페이스다.

LocalDate를 이용해 이들 4개의 클래스 중 하나의 인스턴스를 만들 수 있다.

```java
LocalDate date = LocalDate.of(2023, 8, 23);
JapaneseDate japaneseDate = JapaneseDate.from(date);
```

또는 특정 Locale과 Locale에 대한 날짜 인스턴스로 캘린더 시스템을 만드는 방법도 있다. 새로운 날짜와 시간 API에서 Chronology는 캘린더 시스템을 의미하며 정적 팩토리 메서드 ofLocale을 이용해 인스턴스를 획득할 수 있다.

```java
Chronology japaneseChronology = Chronology.ofLocale(Locale.JAPAN);
ChronoLocalDate now = japaneseChronology.dateNow();
```

날짜와 시간 API의 설계자는 ChronoLocalDate보다는 **LocaleDate를 사용하길 권장**한다. 프로그램의 입출력을 지역화하는 상황을 제외하고는 모든 데이터 저장, 조작, 비즈니스 로직에 LocaleDate를 사용해야 한다.

<br>

# 📌 Summary

- Java 8 이전에는 java.util.Date 클래스와 관련 클래스에서 여러 불일치점들과, 가변성, 어설픈 오프셋, 기본값, 잘못된 이름 결정 등의 결함이 존재했다
- 새로운 날짜 시간 API에서 날짜와 시간 객체는 모두 **불변(Immutable)**이다.
- 날짜와 시간 객체를 절대적인 방법과 상대적인 방법으로 처리할 수 있고 기존 인스턴스를 변환하지 않도록 처리 결과로 새로운 인스턴스가 생성된다
- TemporalAdjuster를 이용하면 단순히 값을 바꾸는 것 이상의 복잡한 동작을 수행할 수 있다
- 날짜와 시간 객체를 특정 포맷으로 출력하고 파싱하는 포매터를 정의할 수 있다. 패턴을 이용하거나 프로그램으로 포매터를 만들 수 있고 포매터는 스레드 안전성을 보장한다