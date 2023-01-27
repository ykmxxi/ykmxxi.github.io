---
title: "[Spring] Transaction, @Transactional"
categories: [Spring]
tag: ["Spring", "SpringBoot"]
toc: true
toc_sticky: true
author_profile: false
sidebar:
    nav: "docs"
---

# 📌 Transaction?

- **Transaction**: 데이터베이스 관리 시스템 또는 유사한 시스템에서 상호작용의 단위. 즉, 데이터베이스의 상태를 변화시키기 해서 수행하는 작업의 단위
  - 모든 작업들이 성공적으로 완료되어야 작업 묶음의 결과를 적용하고, 어떤 작업에서 오류가 발생했을 때는 이전에 있던 모든 작업들이 성공했어도 없었던 일처럼 되돌림
  - DB를 다룰 때 트랜잭션을 적용하면 CRUD 등으로 이루어진 작업을 처리하던 중 오류가 발생했을 때 모든 작업들을 원상태로 되돌릴 수 있다(rollback).
- `@Transactional`: 스프링이 제공하는 선언적 트랜잭션 처리를 지원하는 annotation
  - DB와 관련된 서비스 클래스 or 메서드에 사용

<br>

# 📌 트랜잭션의 4가지 특성 ACID

- **Atomic(원자성)**: 트랜잭션과 관련된 작업들이 부분적으로 실행되다가 중단되지 않는 것을 보장하는 능력. 즉, All or Nothing의 개념으로 작업 단위를 일부분만 실행하지 않는다는 것을 의미.
  - 100개 중 1개 라도 실패하면 모두 실패로 간주해 트랜잭션 시작 전 상태로 rollback
  - 이전 수행 작업을 **임시 영역(rollback segment)**에 따로 저장하고, 현재 수행하고 있는 트랜잭션에서 오류가 발생하면 현재 내역을 날려버리고 임시 영역에 저장했던 상태로 rollback
  - 사람이 설계한 논리적인 작업 단위이기 때문에 일처리가 작업 단위 별로 이루어져야 사람이 다루는데 무리가 없다.
- **Consistency(일관성)**: 트랜잭션이 실행을 성공적으로 완료하면 언제나 일관성 있는 데이터베이스 상태로 유지하는 것
  - 시스템(DB)이 가지고 있는 고정요소는 수행 전후의 상태가 같아야 하며 트랜잭션의 작업 결과는 항상 일관성이 있어야 한다.
  - 작업이 진행되는 동안 DB가 변경되더라도 업데이트된 DB로 트랜잭션이 진행되는 것이 아니라 처음 실행 시점의 상태로 진행된다.
  - 트랜잭션의 일관성을 보장하기 위한 방법은 어떤 이벤트와 조건이 발생했을 때, **트리거(Trigger)**를 통해 보장하는 것이다. 트리거는 '방아쇠'라는 뜻인데, 데이터베이스 시스템이 자동적으로 수행할 동작을 명시할 때 사용된다. 즉, 어떤 행위의 시작을 알리는 것이다.
- **Isolation(독립성, 고립성)**: 트랜잭션을 수행 시 다른 트랜잭션의 연산 작업이 끼어들지 못하도록 보장, 이것은 트랜잭션 밖에 있는 어떤 연산도 중간 단계의 데이터를 볼 수 없음을 의미.
  - 트랜잭션끼리 서로를 간섭할 수 없다.
  - DB는 클라이언트들이 같은 데이터를 공유하는 것이 목적이므로 여러 트랜잭션이 동시에 수행되어야 한다. 이때 트랜잭션은 상호 간의 존재를 모르고 독립적으로 수행
  - 각각의 트랜잭션은 다른 트랜잭션의 수행에 영향을 받지 않고 독립적으로 수행
  - OS의 세마포어(semaphore)와 비슷한 개념으로 lock&excute unlock을 통해 고립성을 보장할 수 있다. 즉, 데이터를 사용할 때는 문을 잠궈서 다른 트랜잭션이 접근하지 못하도록 고립성을 보장하고, 수행을 마치면 언락(unlock)을 통해 데이터를 다른 트랜잭션이 접근할 수 있도록 허용하는 방식
  - 트랜잭션에서는 데이터를 읽을 때: 여러 트랜잭션이 읽을 수는 있도록 허용하는 공유 록(shared_lock)을 한다. 즉, 공유 록은 데이터 쓰기를 허용하지 않고 오직 읽기만 허용
  - 트랜잭션에서 데이터를 쓸 때: 다른 트랜잭션이 읽고 쓸 수 없도록 하는 배타 록(exclusive_lock)을 사용
- **Durability(지속성)**: 성공적으로 수행된 트랜잭션은 영원히 반영되어야 함을 의미, 시스템 문제, DB 일관성 체크 등을 하더라도 유지되어야 함을 의미한다. 전형적으로 모든 트랜잭션은 로그로 남고 시스템 장애 발생 전 상태로 되돌릴 수 있다. 트랜잭션은 로그에 모든 것이 저장된 후에만 commit 상태로 간주될 수 있다.
  - 트랜잭션의 성공 결과 값은 장애 발생 후에도 변함없이 보관되어야 한다는 것으로 트랜잭션이 정상적으로 완료된 경우에는 버퍼의 내용을 하드디스크(데이터베이스)에 확실히 기록해야 하며, 부분 완료(Partial Commit)된 경우에는 작업을 취소(Aborted)
    - Aborted(철회): 롤백 연산을 수행한 상태
  - 정상적으로 완료 혹은 부분 완료된 데이터는 DBMS가 책임지고 데이터베이스에 기록하는 성질이 지속성이며 영속성이라고 표현

<br>

# 📌 @Transactional

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class DmakerService {
    private final DeveloperRepository developerRepository;
    private final EntityManager em;

    public void createDeveloper() {
        EntityTransaction transaction = em.getTransaction();

        try {
            transaction.begin();

            // business logic start
            Developer developer = Developer.builder()
                    .developerLevel(DeveloperLevel.JUNIOR)
                    .developerSkillType(DeveloperSkillType.BACK_END)
                    .experiencedYears(2)
                    .name("Kim")
                    .age(28)
                    .build();

            developerRepository.save(developer); // 영속화, DB에 저장
            // business logic end

            transaction.commit();
        } catch (Exception e) {
            log.info(e.getMessage());
        }

    }
}
```

- business logic start ~ end 부분을 제외하고 위 아래를 살펴보면 트랜잭션 처리 코드는 동일함을 유추할 수 있다 → AOP 적용 가능(@Transactional 사용)

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class DmakerService {
    private final DeveloperRepository developerRepository;

    @Transactional
    public void createDeveloper() {
        // business logic start
        Developer developer = Developer.builder()
                .developerLevel(DeveloperLevel.JUNIOR)
                .developerSkillType(DeveloperSkillType.BACK_END)
                .experiencedYears(2)
                .name("Kim")
                .age(28)
                .build();

        developerRepository.save(developer); // 영속화, DB에 저장
        // business logic end
    }
}
```

## @Transactional 실행 과정

- `TransactionInterceptor.java` 를 살펴보면 실행 과정을 알 수 있음.

```java
	@Override
	@Nullable
	public Object invoke(MethodInvocation invocation) throws Throwable {
		// Work out the target class: may be {@code null}.
		// The TransactionAttributeSource should be passed the target class
		// as well as the method, which may be from an interface.
		Class<?> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);

		// Adapt to TransactionAspectSupport's invokeWithinTransaction...
		return invokeWithinTransaction(invocation.getMethod(), targetClass, new CoroutinesInvocationCallback() {
			@Override
			@Nullable
			public Object proceedWithInvocation() throws Throwable {
				return invocation.proceed();
			}
			@Override
			public Object getTarget() {
				return invocation.getThis();
			}
			@Override
			public Object[] getArguments() {
				return invocation.getArguments();
			}
		});
	}
```

- `MethodInvocation`: @Transactional을 붙인 메서드를 불러오는 인터페이스
  - Method Invocation은 JoinPoint(AOP 연결 가능 지점)이다.
  - 즉, annotation이 붙은 메서드가 AOP 연결 가능 지점인 JoinPoint가 된다.

```java
public interface MethodInvocation extends Invocation {
}

public interface Invocation extends JoinPoint {
}
```

- `invokeWithinTransaction`: 트랜잭션을 가져와 작업을 모두 성공하면 결과를 return 하고 작업이 하나라도 실패하면 rollback