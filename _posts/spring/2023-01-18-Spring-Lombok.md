---
title: "[Spring] Lombok"
categories: [Spring]
tag: ["Spring", "SpringBoot"]
author_profile: false
sidebar:
    nav: "counts"
---



# 📌 Lombok

- 에디터나 빌드툴에 자동으로 연결되어 Java 구현의 생산성을 높이는데 도움을 주는 라이브러리
- 자주 반복되는 getter/setter, toString, equals와 같은 코드를 작성하지 않고 하나의 annotation 기반으로 자동완성을 제공
- 예시로 Java Bean 규약의 getter/setter를 매번 만들고, 유지보수 시 번거로움이 존재
  - 에디터에서 Refactor 기능을 제공하지만 여전히 변수명이 수정되면 관련된 모든 코드가 알맞게 수정되었는지 확인해야 함.

```java
@Setter
@Getter
public class User {
    private String name;
    private Integer age;
    private String email;
}
```

<br>

# 📌 많이 쓰이는 Annotation

## @Getter/@Setter

getter/setter 메서드를 자동완성 시켜주는 annotation. 접근 제어자 `public`, `protected`, `private`에서 사용 가능하며 명시적으로 지정하지 않으면 `public` 메서드가 생성된다.

- Vanilla Java

```java
public class User {
    private String name;
    private Integer age;
    private String email;

    public String getName() {
        return this.name;
    }

    public Integer getAge() {
        return this.age;
    }

    public String getEmail() {
        return this.email;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

- With Lombok

```java
import lombok.Getter;
import lombok.Setter;

@Setter
@Getter
public class User {
    private String name;
    private Integer age;
    private String email;
}
```

## @NoArgsConstructor

매개변수가 없는 기본 생성자를 자동완성 시켜주는 annotation.

- Vanilla Java

```java
public class User {
    private String name;
    private Integer age;
    private String email;

    public User() {
    }
}
```

- With Lombok

```java
import lombok.NoArgsConstructor;

@NoArgsConstructor
public class User {
    private String name;
    private Integer age;
    private String email;
}
```

## @AllArgsConstructor

필드의 모든 변수를 매개변수로 지정한 생성자를 자동완성 시켜주는 annotation.

- Vanilla Java

```java
public class User {
    private String name;
    private Integer age;
    private String email;

    public User(String name, Integer age, String email) {
        this.name = name;
        this.age = age;
        this.email = email;
    }
}
```

- With Lombok

```java
import lombok.AllArgsConstructor;

@AllArgsConstructor
public class User {
    private String name;
    private Integer age;
    private String email;
    
}
```

## @RequiredArgsConstructor

특정 변수만을 매개변수로 하는 생성자를 자동완성 시켜주는 annotation. `@NonNull` annotation을 붙여서 해당 변수를 생성자의 파라미터로 추가할 수 있다. 또는 `final`로 선언해도 의존성을 주입받을 수 있다.

- Vanilla Java

```java
import lombok.NonNull;

public class User {
    @NonNull
    private String name;
    private Integer age;
    private String email;

    public User(@NonNull String name) {
        this.name = name;
    }
}
```

- With Lombok

```java
import lombok.NonNull;
import lombok.RequiredArgsConstructor;

@RequiredArgsConstructor
public class User {
    @NonNull
    private String name;
    private Integer age;
    private String email;

}
```

## @Data

@ToString, @EqualsAndHashCode, @Getter, @Setter, @RequiredArgsConstructor 를 모두 자동완성 시켜주는 annotation.

- Vanilla Java

```java
public class User {
    private String name;
    private Integer age;
    private String email;

    public User() {
    }

    public String getName() {
        return this.name;
    }

    public Integer getAge() {
        return this.age;
    }

    public String getEmail() {
        return this.email;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public boolean equals(final Object o) {
        if (o == this) {
            return true;
        }
        if (!(o instanceof User)) {
            return false;
        }
        final User other = (User) o;
        if (!other.canEqual((Object) this)) {
            return false;
        }
        final Object this$name = this.getName();
        final Object other$name = other.getName();
        if (this$name == null ? other$name != null : !this$name.equals(other$name)) {
            return false;
        }
        final Object this$age = this.getAge();
        final Object other$age = other.getAge();
        if (this$age == null ? other$age != null : !this$age.equals(other$age)) {
            return false;
        }
        final Object this$email = this.getEmail();
        final Object other$email = other.getEmail();
        if (this$email == null ? other$email != null : !this$email.equals(other$email)) {
            return false;
        }
        return true;
    }

    protected boolean canEqual(final Object other) {
        return other instanceof User;
    }

    public int hashCode() {
        final int PRIME = 59;
        int result = 1;
        final Object $name = this.getName();
        result = result * PRIME + ($name == null ? 43 : $name.hashCode());
        final Object $age = this.getAge();
        result = result * PRIME + ($age == null ? 43 : $age.hashCode());
        final Object $email = this.getEmail();
        result = result * PRIME + ($email == null ? 43 : $email.hashCode());
        return result;
    }

    public String toString() {
        return "User(name=" + this.getName() + ", age=" + this.getAge() + ", email=" + this.getEmail() + ")";
    }
}
```

- With Lombok

```java
import lombok.Data;

@Data
public class User {
    private String name;
    private Integer age;
    private String email;
    
}
```

## @Builder

해당 클래스 객체의 생성에 Builder 패턴을 적용시켜주는 annotation.

- With Lombok

```java
@Builder
public class User {
    private String name;
    private Integer age;
    private String email;
    
}

// 빌더 패턴을 이용한 객체 생성
User user2 = User.builder()
                .name("Lee")
                .age(22)
                .email("eamil2@email.com")
                .build();
```

## @Slf4j

Slf4j(Simple Logging Facade for Java)는 다양한 로깅 프레임워크(java.util.logging, log4j)에 대한 추상화 역할을 하는 인터페이스.

- With Lombok

```java
public class Application {
    public static void main(String[] args) {
        User user1 = new User("Kim", 25, "eamil@email.com");
        log.info(user1.getEmail());
        // 16:38:21.516 [main] INFO com.devmaker.dmaker.testtest - eamil@email.com
}
```