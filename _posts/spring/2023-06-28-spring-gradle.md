---
title: "[Spring] Gradle 이란?"
categories: [Spring]
tag: ["Spring", "SpringBoot"]
toc: true
toc_sticky: true
author_profile: false
sidebar:
    nav: "counts"
---

> [Gradle Github](https://github.com/gradle/gradle)
>
> [Gradle 공식 문서](https://docs.gradle.org/current/userguide/what_is_gradle.html)

<br>

# 📌 Introduce

인텔리제이 + 스프링 조합을 사용해본 백엔드 개발자라면 프로젝트 시작시 아래 그림을 많이 봤을 것이다.

<center><img src="/assets/images/spring/gradle/1.png"></center>

유명 강의, 서적, Youtube 에서  스프링 기반 백엔드 프로젝트를 시작하면 99%가 `Gradle-groovy`를 선택하는데 왜 Gradle을 선택하는지, Gradle이 정확히 무엇인지 알려주는 부분이 부재했다.

별개의 이야기로 과거 Android Studio에서 프로젝트를 진행한 적이 있는데 이때도 Gradle을 사용했고, 의존성을 추가하는데 자꾸 에러가 나는 어려움을 겪은 적이 있다.

그래서 Gradle이 정확히 무엇인지 한번 짚고 넘어가 보도록 한다.

<br>

# 📌 Gradle 이란?

공식 문서에서 Gradle은 모든 유형의 소프트웨어를 빌드할 수 있을 만큼 유연한 오픈 소스 **빌드 자동화 기술**이라 말한다. 모든 플랫폼에서 소프트웨어 구축, 테스트, 게시, 배포의 전체 개발 수명 주기를 지원할 수 있는 유연한 빌드 도구이다.

Gradle은 그루비(Groovy) 기반의 DSL(Domain Specific Language)을 사용한 빌드 자동화 도구이다. Gradle 이전에도 Ant, Maven 같은 빌드 도구가 존재했고, 이전 세대의 단점을 보완하고 더 유연한 기술을 제공하기 위해 탄생했다.

## Ant

- XML 기반 빌드 스크립트를 작성해 동작하는 빌드 도구
- 자유롭게 빌드 단위를 지정하고 사용하기 쉽다
- 프로젝트가 커지면 스크립트 관리나 빌드 과정이 복잡해지는 단점이 존재한다
- 생명주기(Lifecycle)가 없어 각각의 결과물에 대한 의존관계 등을 정의해야 한다

## Maven

- XML 기반으로 작성하는 빌드 도구
- 생명주기, 프로젝트 객체 모델 개념이 도입
- Ant의 빌드 스크립트를 개선
- `pom.xml`에 필요한 라이브러리를 선언해 자동으로 해당 프로젝트를 불러와 사용이 편리하다

<br>

Gradle은 Ant의 유연한 범용성, Maven의 구조화된 프레임워크의 장점을 가져와 설계되었다. 의존성 관리를 위해 XML 언어가 아닌 Groovy의 DSL을 사용한다.

JVM을 기반으로 동작하기에 자바 문법과 비슷해 자바 개발자는 쉽게 Gradle을 익힐 수 있고 Gradle Wrapper를 이용하면 Gradle이 설치되지 않은 시스템에서도 프로젝트를 빌드할 수 있다. 또 Maven의 `pom.xml`을 Gradle에 맞게 변환할 수 있어 Maven의 라이브러리를 그대로 가져다 사용할 수 있다.

## Gradle Design

- 고성능(High Performance): input 이나 output이 변경되어 수행되는 작업만 실행해 불필요한 작업을 방지하고, 캐시를 사용해 이전 빌드의 출력을 재사용한다.
- JVM 기반: Gradle은 JVM 기반으로 동작하는 Groovy의 DSL 언어를 기반으로 한다. 자바에 익숙한 개발자에게는 큰 힘이 될 것이다.
- 컨벤션(Convention): 프러그인은 빌드 스크립트를 최소화하기 위해 기본값을 설정했다. 하지만 이런 컨벤션은 제한적이지 않고 사용자가 원하는 모습으로 정의할 수 있다
- 확장성(Extensibility): Gradle은 플러그인(Plugin)을 사용해 쉽게 확장할 수 있다

<br>

# 📌 Gradle을 이해하기 위한 준비

공식문서에서 Gradle을 학습하기 위해서는 몇 가지 용어들을 이해하길 권장한다. 원활한 학습을 위해 차례대로 용어들을 정리한다.

## Projects(프로젝트)

- Gradle이 빌드하는 대상

스프링 프로젝트를 생성해보면 `build.gradle` 파일이 생성되는 것을 확인할 수 있다. 프로젝트에는 일반적으로 `build.gradle` 라는 이름의 빌드 스크립트 파일이 프로젝트의 루트 디렉토리에 위치하고 있다.

빌드 스크립트는 프로젝트에 대한 작업(tasks), 의존성(dependencies), 플러그인(plugins) 및 기타 구성 요소들이 포함되어 있다.

## Tasks(작업)

- 일부 작업을 실행하기 위한 로직

Tasks는 코드 컴파일, 테스트 실행, 배포와 같은 일부 작업을 실행하기 위한 로직이 포함되어 있다. 일반적으로 많이 사용되는 빌드 시스템 요구사항을 구현하는데 필요한 작업을 제공한다.

Tasks는 총 3가지의 구성요소를 가진다

1. **Actions**: 파일 복사 또는 소스 컴파일과 같이 작업을 수행하는 단위
2. **Inputs**: Actions이 사용하거나 조작하는 값, 파일 및 디렉토리
3. **Outputs**: Actions이 수정하거나 생성하는 파일 및 디렉토리

## Plugins(플러그인)

- 프로젝트에서 로직과 구성을 재사용할 수 있는 수단을 제공

플러그인은 tasks와 파일 및 dependencies 구성 이외에도 빌드에 필요한 새로운 개념들을 도입할 수 있게 도와준다.

플러그인을 사용하면 한 번 작성해 여러 빌드에서 사용할 수 있고, 로깅과 dependencies 관리, 버전 관리 같은 공통적인 구성 요소들을 한 곳에 모아 저장할 수 있다. 이를 통해 불필요한 중복을 줄이고 편의성과 효율성을 제공한다.

## Build Phases(빌드 단계)

- Gradle은 세 가지 단계를 통해 빌드 스크립트를 실행한다

1. **Initialization(초기화)**: 빌드를 위한 환경을 설정하고 어떤 프로젝트가 빌드에 참여하는지 결정한다. `settings.gradle` 파일에서 빌드에 필요한 프로젝트를 구성한다
2. **Configuration(구성)**: 빌드에 필요한 작업을 구성하고 사용자가 실행하려는 Tasks에 따라 어떤 작업이 어떤 순서로 실행되어야 하는지 결정한다
3. **Execution(실행)**: 구성된 Tasks들을 실행한다

## Builds(빌드)

- 빌드 단계에서 결정된 Tasks들을 실행

Gradle의 Tasks들을 CLI(Command Line Interface)나 IDE를 통해 실행한다. 요청된 작업과 해당 작업의 dependencies에 따라 최소한의 작업 set를 실핸한다

<br>

# 📌 Gradle의 장점

## 1. xml에서 벗어나 읽기 편하다

Gradle은 과거 빌드 도구인 Ant, Maven과 달리 xml 파일을 사용하지 않고 Groovy의 DSL을 기반으로 한다. xml 파일을 학습하지 않은 상태에서 읽어 보면 무엇을 의미하는지 알아보기가 어렵다. 또 학습을 한 상태에서도 읽기가 불편할 것 같이 생다.

반면 `build.gradle`파일을 열어보면 비교적 읽기가 쉽다. 어느 플러그인을 쓰고, 어떤 작업을 하고, 어떤 dependency가 있는지 읽기 편하다.

최근 Kotlin DSL이 제공되면서 더욱 편해지고 스크립트를 작성하기가 쉬워졌다.

## 2. 다양한 Repository를 사용할 수 있다

Gradle은 다양한 Repository를 사용할 수 있도록 지원한다. Maven의 Repository 뿐만 아니라 개발자가 직접 만든 Repository 또한 사용할 수 있다.

```groovy
repositories {
    mavenCentral() // Maven Central 
    myCustomCentral() // 커스텀 Central
}
```

## 3. 작업에 필요한 최소한의 라이브러리들만 가져올 수 있다

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-jdbc'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    testCompileOnly 'org.projectlombok:lombok'
    testAnnotationProcessor 'org.projectlombok:lombok'
}
```

`build.gradle`의 dependencies 관련 부분을 보면 라이브러리들에 implementation이 붙어 있는 것을 확인할 수 있다.

Gradle은 사용자가 필요한 라이브러리를 간편하게 가져와 작업을 수행할 수 있게 도와준다

# 📌 Spring 프로젝트에서 Gradle 분석하기

실제 스프링 프로젝트에서 Gradle의 구조가 어떻게 구성되어 있는지 분석해본다.

<center><img src="/assets/images/spring/gradle/2.png"></center>

## /.gradle & /gradle

Gradle 버전별 엔진과 설정 파일들이 담겨 있다. 위 그림에서는 Gradle 7.6.1 버전을 사용하고 있음을 알 수 있다

또 gradle 디렉토리 하위를 보면 Gradle Wrapper가 존재함을 확인할 수 있다. 이 파일을 통해 내장 Gradle을 제공해 별도의 Gradle 설치 없이 빌드를 할 수 있도록 지원한다.

## gradlew & gradlew.bat

내장 Gradle인 Gradle Wrapper의 사용을 간단하게 만들기 위해 만들어진 파일이다.

gradlew는 맥, 리눅스용 스크립트, gradlew.bat은 윈도우용 스크립트이다. 이 파일들은 Gradle 명령어를 이용해 내장 Gradle을 이용해 빌드하기 위한 초기화 단계부터 빌드 단계까지 모두 수행한다.

## build.gradle

빌드에 필요한 모든 기능들이 정의된 파일이다.

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '2.7.12'
    id 'io.spring.dependency-management' version '1.0.15.RELEASE'
}

group = 'hello'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '16'
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-jdbc'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    // 테스트에서 lombok 사용
    testCompileOnly 'org.projectlombok:lombok'
    testAnnotationProcessor 'org.projectlombok:lombok'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

- `plugins`: 미리 구성해 놓은 tasks. 빌드과정에 필요한 기본정보를 포함하고 필요에 따라 정보를 수정해 사용 가능하다.
  - 플러그인을 통해 프로젝트의 기능을 확장
  - 플러그인에 사용하는 기능을 명시해 불필요한 코드들을 줄일 수 있다
  - 위 파일에서는 java 언어를 사용하고, SpringBoot 2.7.12 버전을 사용하는 것을 확인할 수 있다
- `group`: 프로젝트의 group명
- `version`: 프로젝트의 현재 버전
- `java {sourceCompatibility = '16'}`: 현재 사용하고 있는 java version
- `configurations`: `build.gradle`파일 내부에서 사용되는 설정을 정의
- `repositories`: dependencies를 가져올 저장소를 정의
- `dependencies`: 프로젝트에서 사용하는 라이브러리 목록
  - implementation: 컴파일 타임과 런타임에 모두 쓰이는 라이브러리
  - compileOnly: 컴파일 타임에만 사용되는 라이브러리
  - runtimeOnly: 런타임에만 사용되는 라이브러리
  - annotationProcessor: 컴파일시 사용되는 라이브러리를 주입
  - testImplementation: 테스트시 사용하는 라이브러리
  - testCompileOnly: 테스트 파일 컴파일 타임에 사용되는 라이브러리
  - testAnnotationProcessor: 테스트 파일 컴파일시 사용되는 라이브러리를 주입

## settings.gradle

프로젝트의 구성 정보를 기록하는 파일이다. 어떤 프로젝트들이 어떤 관계로 구성되어 있는지를 저장해 빌드시 Gradle은 해당 파일에 저장된 정보대로 프로젝트를 구성한다.

```groovy
rootProject.name = 'example'

include 'java'
include 'kotlin'
```

- `rootProject.name`: 최상위 프로젝트의 이름
- `include`: 최상위 프로젝트의 하위 프로젝트