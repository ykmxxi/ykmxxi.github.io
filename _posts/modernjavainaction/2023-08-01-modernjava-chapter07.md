---
title: "[모던 자바 인 액션] 07. 병렬 데이터 처리와 성능"
categories: [Modern Java in Action]
tag: ["Java"]
author_profile: false
sidebar:
    nav: "counts"
---

> Reference
> [모던 자바 인 액션: 람다, 스트림, 함수형, 리액티브 프로그래밍으로 새로워진 자바 마스터하기](https://www.yes24.com/Product/Goods/77125987?pid=123487&cosemkid=go15646485055614872&gclid=Cj0KCQjw2qKmBhCfARIsAFy8buL-QGAkh5m5YK8mj5vAgRgCKdSRIHzDDWdAYOLPFSuQNlzVVcFFdbYaAsh1EALw_wcB)

<br>

외부 반복(명시적 반복)을 내부 반복(스트림)으로 바꾸면 네이티브 자바 라이브러리가 스트림 요소의 처리를 제어할 수 있음을 확인했다.

Java 7이 등장하기 전에는 데이터 컬렉션을 병렬로 처리하기 어려웠다. 컬렉션을 분할하고 각각의 쓰레드로 할당하면 의도치 않은 경쟁 상태(race condition)이 발생할 수 있어 적절한 동기화가 필요하고, 각각 처리된 결과를 다시 병합하는 과정이 필요했다.

Java 7은 더 쉽게 병렬화를 수행하면서 에러를 최소화할 수 있도록 **fork/join framework** 기능을 제공한다.

스트림을 이용하면 순차 스트림을 병렬 스트림으로 자연스럽게 바꿀 수 있는데, 어떻게 이런 일이 가능한지fork/join framework와 내부적인 병렬 스트림 처리가 어떤 관계인지 살펴봐야한다.

# 📌 병렬 스트림

- **병렬 스트림(Parallel Stream)**: 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림
  - 컬렉션에 `parallelStream`을 호출하면 생성됨

병렬 스트림을 이용하면 모든 멀티코어 프로세서(CPU)가 각각의 청크를 처리하도록 할당할 수 있다

먼저 1부터 n까지 모든 숫자의 합계를 구하는 메서드를 순차 스트림으로 구현해본다

```java
	public long sequentialSum(long n) {
		return Stream.iterate(1L, i -> i + 1) // 무한 자연수 스트림 생성
			.limit(n) // n개 이하로 제한
			.reduce(0L, Long::sum); // 모든 숫자를 더한 스트림 BinaryOperator 리듀싱 연산
	}
```

n이 매우 커진다면 이 연산을 병렬로 처리하는 것이 좋을 것인데, 무엇부터 건드려야 할까? 결과 변수는 어떻게 동기화해야 할까? 몇 개의 스레드를 사용해야 할까? 등 많은 것을 고려해야할 것 같지만, Java가 제공하는 병렬 스트림을 사용하면 이 문제들을 고민 없이 해결할 수 있다

## 순차 스트림을 병렬 스트림으로 변환하기

- `parallel()`를 호출하면 기존의 함수형 리듀싱 연산이 병렬로 처리된다

병렬 스트림으로 변환하면 스트림이 여러 청크로 분할되어 리듀싱 연산을 병렬로 수행한다. 연산이 끝나면 생성된 부분 결과를 다시 리듀싱 연산으로 합쳐 전체 스트림의 리듀싱 결과를 도출한다.

```java
	public static long parallelSum(long n) {
		return Stream.iterate(1L, i -> i + 1)
			.limit(n)
			.parallel() // 스트림을 병렬 스트림으로 변환
			.reduce(0L, Long::sum);
	}
```

1 부터 1억 까지 더했을 때 병렬 스트림이 확실히 좋은 성능을 보인 것을 확인할 수 있다.

<center><img src="/assets/images/modernjavainaction/ch07/1.png"></center>

<center><img src="/assets/images/modernjavainaction/ch07/2.png"></center>

순차 스트림에 `parallel()`을 호출해도 스트림 자체에는 아무 변화도 일어나지 않는다. 내부적으로 연산이 병렬로 수행해야 함을 의미하는 boolean 플래그가 설정되고, 반대로 sequential로 병렬을 순차로 바꿀수도 있다.

parallel과 sequential 중 최종적으로 호출된 메서드가 전체 파이프라인에 영향을 준다

```java
Stream.parallel()
	.filter()
	.sequential()
	.map()
	.parallel() // 마지막 병렬 호출 -> 파이프라인은 전체적으로 병렬로 실행
	.reduce();
```

> 병렬 스트림은 내부적으로 ForkJoinPool 스레드 풀을 사용한다. ForkJoinPool은 프로세서 수를 `Runtime.getRuntime().availableProcessors()`가 반환하는 값에 상응하는 스레드를 갖는다. `System.setProperty()`를 통해 병렬 스트림에 사용할 수 있는 쓰레드의 수를 특정한 값으로 지정할 수 있는데, 일반적으로 기기의 프로세서 수와 같으므로 특별한 이유가 없으면 기본값을 그대로 사용할 것을 권장

## 스트림 성능 측정: 병렬이 무조건 빠른가?

병렬화를 이용하면 성능이 더 좋아질 것이라 추측 가능한데, 소프트웨어 공학에서 추측은 매우 위험한 방법이다. 성능을 최적화할 때는 **측정이 꼭 필요하고 매우 중요**하다.

JMH(Java Microbenchmark Harness) 라이브러리를 이용하면 작은 벤치마크를 애노테이션 기반으로 구현할 수 있다.

> JVM 기반 프로그램을 벤치마크하는 작업은 쉽지 않다. 핫스팟(HotSpot)이 바이트코드를 최적화 하는데 필요한 준비 시간(warm-up), GC로 인한 오버헤드 등과 같은 여러 요소를 고려해야 하기 때문이다.

For-Loop를 이용한 메서드가 기본값을 박싱하거나 언박싱할 필요 없어 더 빠를 것이라 예상할 수 있는데 실제로 순차 스트림보다 약 4배 빠르다. 병렬 스트림은 순차 스트림보다 더 빠를거라 예상할 수 있는데 실제 측정 결과 병렬 스트림이 순차 스트림보다 5배 느린 결과가 나온다. 이유는 무엇일까?

1. 반복 결과로 박싱된 객체가 만들어지므로 숫자를 더하려면 언박싱이 필요
2. 반복 작업(`iterate`)은 병렬로 수행할 수 있는 독립 단위로 나누기 어렵다

이전 연산의 결과에 따라 다음 함수의 입력이 달라지기 때문에 iterate 연산을 청크로 분할하기가 어렵다. 1 부터 n 까지 이미 준비된 숫자 리스트가 아니므로 스트림을 병렬로 처리할 수 있도록 청크로 분할할 수 없다. 합계 연산만 다른 스레드에서 수행되었지 순차 스트림과 큰 차이가 없고, 스레드를 할당하는 오버헤드만 증가하게 되어 이런 결과가 나오게 되었다.

이처럼 **병렬 프로그래밍은 까다롭고 이해하기 어려운 함정이 숨어있다**. 따라서 `parallel()` 호출 시 내부에서 무슨 일이 일어나는지 이해하고 있어야 필요한 곳에 병렬 스트림을 사용해 좋은 성능을 이끌어낼 수 있다.

## 특화된 메서드를 사용한 성능 최적화

멀티코어 프로세서를 활용해 효과적으로 위 합계 연산을 병렬로 실행하려면 어떻게 해야 할까?

`iterate`가 아닌 `LongStream.rangeClosed` 메서드를 사용하면 다음과 같은 장점을 얻을 수 있다.

- 기본형 long을 직접 사용해 박싱과 언박싱 오버헤드가 사라진다
- 쉽게 청크로 분할할 수 있는 숫자 범위를 생산한다
  - ex: n = 20, 1 ~ 5, 6 ~ 10, 11 ~ 15, 16 ~ 20 까지 4개의 청크로 분할

```java
	/**
	 * 병렬 합계 연산 최적화
	 * LongStream.rangeClosed() 사용
	 */
	public static long rangedSum(long n) {
		return LongStream.rangeClosed(1, n)
			.reduce(0L, Long::sum);
	}
```

<center><img src="/assets/images/modernjavainaction/ch07/3.png"></center>

기존의 `iterate` 팩토리 메서드로 생성한 순차 스트림보다 훨씬 빠르다. 이처럼 상황에 따라서 어떤 알고리즘을 병렬화하는 것보다 적절한 자료구조를 사용하는것이 더 중요하다는 사실을 보여준다.

여기에 `parallel()`을 추가해 병렬로 수행시켜도 기존 순차 스트림, 병렬 스트림보다 훨씬 빠른 것을 확인할 수 있다. **올바른 자료구조를 선택해야 병렬 실행도 최적의 성능을 발휘할 수 있다**

<center><img src="/assets/images/modernjavainaction/ch07/4.png"></center>

병렬화는 완전 공짜는 아니라는 사실을 기억해야 한다. 병렬화를 이용하려면 스트림을 재귀적으로 분할해야 하고, 각 서브 스트림을 서로 다른 스레드에 할당하고 이들을 다시 합치는 과정을 거쳐야 한다. 멀티코어 간 데이터 이동은 생각보다 비용이 비싸다. 따라서 데이터 전송 시간보다 훨씬 오래 걸리는 작업을 병렬로 수행해야 병렬 프로그래밍의 진가를 확인할 수 있다.

## 병렬 스트림에서 발생하는 문제

- **병렬 스트림과 병렬 계산에서는 공유된 가변 상태를 피해야 한다!**

병렬 스트림을 잘못 사용하면서 발생하는 많은 문제는 공유된 상태를 바꾸는 알고리즘을 사용하기 때문에 일어난다. 아래는 n 까지의 합계 연산을 공유된 누적자를 바꾸는 프로그램을 구현한 코드로 병렬 스트림에서 발생할 수 있는 문제를 살펴본다.

```java
	/**
	 * 병렬화 X: 아직 문제가 없음
	 */
	public static long sideEffectSum(long n) {
		Accumulator accumulator = new Accumulator();
		LongStream.rangeClosed(1, n)
			.forEach(accumulator::add);
		return accumulator.total;
	}

	/**
	 * 병렬화 O: 문제 발생
	 */
	public static long sideEffectParallelSum(long n) {
		Accumulator accumulator = new Accumulator();
		LongStream.rangeClosed(1, n)
			.parallel()
			.forEach(accumulator::add);
		return accumulator.total;
	}

	static class Accumulator {

		public long total = 0;

		public void add(long value) {
			total += value;
		}

	}

}
```

<center><img src="/assets/images/modernjavainaction/ch07/5.png"></center>

위 코드는 본질적으로 순차 실행할 수 있도록 구현되어 있는데 **병렬로 실행하면 참사**가 발생한다. 병렬로 실행하면 다수의 스레드가 동시에 같은 데이터에 접근해 데이터 경쟁 문제가 발생한다. 동기화로 이 문제를 해결하다보면 결국 병렬화라는 특성이 없어진다.

결국 여러 스레드에서 공유하는 객체의 상태를 바꾸는 `forEach`블록 내부에서 연산을 실행하면서 이 같은 문제가 발생한다. 이 예제처럼 병렬 스트림을 사용할 때는 **상태 공유에 따른 부작용**을 피해야 한다. 병렬 스트림과 병렬 계산에서는 공유된 가변 상태를 피해야 한다

## 병렬 스트림을 효과적으로 사용하는 방법

어떤 절대적인 데이터 수량을 기준으로 병렬 스트림 사용을 결정하는 것은 적절하지 않다. 프로그램이 실행되는 기기도 다르고, 연산이 프로그램마다 모두 다르기 때문이다.

그래도 병렬 스트림을 사용했을 때 좋은 상황은 있다

- 병렬 스트림을 사용할 때는 직접 측정하자
  - 병렬 스트림이 순차 스트림보다 항상 빠른 것은 아니다
  - 적절한 벤치마크로 직접 성능을 측정하는 것이 좋다
- 박싱을 주의해야 한다
  - 자동 박싱과 언박싱은 성능을 크게 저하시킬 수 있는 요소
  - 기본형 특화 스트림 `IntStream`, `LongStream`, `DoubleStream`을 사용하자
- 병렬 스트림에서 성능이 떨어지는 연산이 있다
  - 요소의 순서에 의존하는 연산은 병렬 스트림에서 비효율적
  - `limit`, `findFirst` 등
- 전체 파이프라인 연산 비용을 고려해야 한다
  - 전체 스트림 비용(N * Q) = 처리해야 할 요소 N개 * 하나의 요소를 처리하는데 드는 비용 Q
  - Q 값이 높아지면 병렬 스트림으로 성능을 개선할 수 있는 가능성이 있음
- 소량의 데이터에서는 병렬 스트림이 도움 되지 않는다
  - 병렬로 얻는 이득이 병렬화 과정에서 드는 비용보다 작다
- 효율적으로 분할할 수 있는 자료구조에 병렬 스트림이 좋다
  - 분해성 훌륭: `ArrayList`, `IntStream.range`
  - 분해성 좋음: `HashSet`, `TreeSet`
  - 분해성 나쁨: `LinkedList`, `Stream.iterate`
- 스트림의 특성과 파이프라인 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분할 과정의 성능이 달라진다
  - SIZED 스트림은 정확히 같은 크기의 두 스트림으로 분할 가능해 효과적인 병렬 처리 가능
  - 필터 연산은 스트림의 길이를 예측할 수 없어 효과적으로 병렬 처리를 할 수 있을지 모름
- 최종 연산의 병합 과정 비용 계산
  - 연산 후 분할된 결과의 병합 과정이 비싸면 비효율적이다

<br>

# 📌 Fork/Join Framework

포크/조인 프레임워크는 병렬화할 수 있는 작업을 재귀적으로 분할해 서브태스크 각각의 결과를 합쳐서 전체 결과를 만들도록 설계되었다. 포크/조인 프레임워크는 서브태스크를 스레드 풀(ForkJoinPool)의 작업자 스레드에 분산 할당하는 `ExecutorService` 인터페이스를 구현한다

## RecursiveTask 활용

- 스레드 풀을 이용을 위해 `RecursiveTask<V>`의 서브클래스 구현이 필요하다
  - V: 병렬화된 태스크가 생성하는 결과 형식, 결과가 없을 때는 `RecursiveAction` 형식
  - `compute()` 추상 메서드를 구현해야 한다

<center><img src="/assets/images/modernjavainaction/ch07/6.png"></center>

`compute()`는 태스크를 서브태스크로 분할하는 로직과 개별 서브태스크의 결과를 생산할 알고리즘을 정의한다. 대부분 아래와 같은 의사코드 형식을 유지한다

```java
if (태스크가 충분히 작거나 더 이상 분할할 수 없으면) {
	순차적으로 태스크를 계산
} else {
	태스크를 두 서브태스크로 분할
	태스크가 다시 서브태스크로 분할되도록 재귀적으로 이 메서드를 호출
	모든 서브태스크의 연산이 완료될 때까지 기다린다
	각 서브태스크의 결과를 병합
}
```

위 코드는 분할 정복(Divide-and-Conquer)의 병렬화 버전이다.

포크/조인 프레임워크를 이용해 범위 숫자를 더하는 문제를 구현해본다

<center><img src="/assets/images/modernjavainaction/ch07/7.png"></center>

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;
import java.util.stream.LongStream;

public class ForkJoinSumCalculator extends RecursiveTask<Long> {

	public static final long THRESHOLD = 10_000; // 이 값 이하의 서브태스크는 더 이상 분할 X
	private final long[] numbers; // 더할 숫자 배열
	private final int start; // 서브태스크에서 처리할 배열의 초기 위치
	private final int end; // 서브태스크에서 처리할 최종 위치

	public ForkJoinSumCalculator(long[] numbers) {
		this(numbers, 0, numbers.length);
	}

	public ForkJoinSumCalculator(long[] numbers, int start, int end) {
		this.numbers = numbers;
		this.start = start;
		this.end = end;
	}

	@Override
	protected Long compute() {
		int length = end - start;
		if (length <= THRESHOLD) {
			return computeSequentially(); // 기준값보다 작거나 같으면 순차적으로 결과를 계산
		}

		ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length / 2);
		leftTask.fork(); // 다른 스레드로 새로 생성한 태스크를 비동기로 실행

		ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length / 2, end);
		Long rightResult = rightTask.compute();
		Long leftResult = leftTask.join();
		return leftResult + rightResult;
	}

	private long computeSequentially() {
		long sum = 0;
		for (int i = start; i < end; i++) {
			sum += numbers[i];
		}
		return sum;
	}

	private static long forkJoinSum(long n) {
		long[] numbers = LongStream.rangeClosed(1, n)
			.toArray();
		ForkJoinSumCalculator task = new ForkJoinSumCalculator(numbers);
		return new ForkJoinPool().invoke(task);
	}

}
```

스레드 풀에서 실행되는 마지막 `invoke()`의 반환값은 `RecursiveTask<Long>`에서 정의한 태스크 결과인 Long

일반적으로 애플리케이션에서는 둘 이상의 ForkJoinPool을 사용하지 않는다. 한 번만 인스턴스화해서 정적 필드에 싱글톤으로 ForkJoinPool을 저장한다. JVM에서 이용할 수 있는 모든 프로세서가 자유롭게 스레드 풀에 접근할 수 있다. 더 정확하게는 `Runtime.availableProcessors`의 반환값으로 풀에 사용할 스레드 수를 결정한다

> `availableProcessors`는 이름과 다르게 실제 프로세서외에 하이퍼스레딩과 관련된 가상 프로세서도 개수에 포함한다

`ForkJoinSumCalculator`를 스레드 풀에 전달하면 스레드가 `compute()`를 실행해 작업을 수행한다. 이때 태스크의 크기가 크면 배열을 반으로 분할해 새로운 `ForkJoinSumCalculator`로 할당하고, 이 과정이 재귀적으로 반복되면서 주어진 조건을 만족할때 까지 태스크 분할을 반복한다. 각 서브태스크는 순차적으로 처리되며 포킹 프로세스로 만들어진 이진트리의 태스크를 루트에서 역순으로 방문해 최종 결과를 계산한다

## 포크/조인 프레임워크를 제대로 사용하는 방법

- `join()`을 태스크에 호출하면 태스크가 생산하는 결과가 준비될 때까지 호출자를 블록 시켜야 한다
  - 그렇지 않으면 서브태스크가 다른 태스크가 끝나길 기다리는 일이 발생해 느려진다
- `RecursiveTask` 내에서는 `ForkJoinPool.invoke()`를 호출하지 말아야 한다. 대신 `compute()` or `fork()`를 호출할 수는 있다
  - `invoke()`는 순차 코드에서 병렬 계산을 시작할 때만 사용
- 서브태스크에 `fork()`를 호출해 스레드 풀의 일정을 조절할 수 있다
  - 한쪽 작업에는 `compute()`, 다른 쪽에서는 `fork()`를 호출해 풀에서 불필요한 태스크를 할당하는 오버헤드를 피할 수 있음

<center><img src="/assets/images/modernjavainaction/ch07/8.png"></center>

- 포크/조인 프레임워크는 디버깅이 어렵다
  - fork라 불리는 다른 스레드에서 compute를 호출해 스택 트레이스가 도움이 되지 않음
- 멀티코어에 포크/조인 프레임워크를 사용하는 것이 순차 처리보다 무조건 빠르지는 않다
  - 병렬 스트림처럼 무조건 빠른 것은 아님
  - 성능 측정이 필요

## 작업 훔치기(Work Stealing)

코어 개수와 관계없이 적절한 크기로 분할된 많은 태스크를 포킹하는 것이 바람직하다. 이론적으로 코어 개수만큼 병렬화된 태스크로 작업을하면 모든 CPU 코어에서 태스크를 실행해 각각의 태스크는 같은 시간에 종료될 것이라고 생각할 수 있다. 하지만 현실에서는 각각의 서브태스크의 완료 시간이 크게 달라질 수 있다.

분할 기법이 효율적이지 않거나, 디스크 접근 속도가 저하되었거나 외부 서비스와 연동하는 과정에서 지연이 생길수도 있기 때문이다.

포크/조인 프레임워크는 이 문제를 **작업 훔치기(Work Stealing)** 기법으로 해결한다. 이 기법에서 스레드는 자신에게 할당된 태스크를 포함한 Doubly-Linked List를 참조하면서 작업이 끝날 때마다 큐의 헤드에서 다른 태스크를 가져와 작업한다. 각 코어마다 작업 처리 시간이 다른데, 이때 작업이 먼저 끝난 코어를 유휴 상태로 바꾸지 않고 다른 스레드 큐의 tail(꼬리)에서 작업을 훔쳐온다. 모든 태스크가 작업을 끝날때 까지, 이 과정을 반복한다. 따라서 태스크 크기를 작게 나누어야 작업자 스레드 간 작업부하를 비슷한 수준으로 유지할 수 있다

<center><img src="/assets/images/modernjavainaction/ch07/9.png"></center>

<br>

# 📌 Spliterator 인터페이스

분할 로직을 개발하지 않고도 병렬 스트림을 이용할 수 있는 이유는 `Spliterator`를 통해 스트림을 자동으로 분할해주는 기능을 Java가 제공하기 때문이다.

- `Spliterator` 인터페이스: 요소를 개별적 or 일괄적으로 순회할 수 있는 요소 탐색 기능을 제공
  - Java 8 부터 제공하며 분할할 수 있는 반복자라는 의미를 지님
  - **병렬 작업에 특화되어 있음**
  - 데이터를 분할하고 병렬 처리하기 위해 사용

Java 8은 모든 컬렉션 프레임워크에 포함된 자료구조에 사용할 수 있는 Default Spliterator 구현을 제공한다. 컬렉션은 `spliterator()` 메서드를 제공하는 `Spliterator` 인터페이스를 구현한다

```java
public interface Spliterator<T> {

	boolean tryAdvance(Consumer<? super T> action);

	Spliterator<T> trySplit();

	long estimateSize();

	int characteristics();
}
```

- `tryAdvance()`: Spliterator의 요소를 하나씩 순차적으로 소비하면서 탐색할 요소가 남아있으면 true, 그렇지 않으면 false를 반환
- `trySplit()`: Spliterator의 일부 요소를 분할해서 두 번째 Spliterator를 생성
- `estimateSize()`: 탐색해야 할 요소 수 정보(추정치)를 반환
  - 무한하거나, 알 수 없거나 계산하기 너무 비싼 경우 `Long.MAX_VALUE`를 반환
- `characteristics()`: Spliterator의 특성(특징)을 나타내는 정수 값을 반환

## 분할 과정

스트림을 여러 스트림으로 분할하는 과정은 재귀적으로 일어난다

1. 첫 번째 Spliterator에 `trySplit()` 호출시 두 번째 Spliterator가 생성된다
2. 두 개의 Spliterator에 `trySplit()` 호출시 네 개의 Spliterator가 생성된다

이 처럼 생성된 Spliterator가 계속 `trySplit()`의 결과가 null이 될 때까지 이 과정을 반복한다. null을 반환했다는 것은 더 이상 자료구조를 분할할 수 없음을 의미한다. Spliterator에서 호출한 모든 `trySplit()`결과가 null이면 재귀 분할 과정이 종료된다

이 분할 과정은 `characteristics()`로 정의하는 Spliterator의 특성에 영향을 받는다

## Spliterator 특성

`characteristics()`는 Spliterator 자체의 특성 집합을 포함하는 int를 반환한다. Spliterator를 사용하는 프로그램은 이 특성을 참고해서 제어하고 최적화할 수 있다

- ORDERED: 리스트처럼 요소에 정해진 순서가 있어 Spliterator는 요소를 탐색하고 분할할 때 이 순서에 유의해야 한다
- DISTINCT: x, y 두 요소를 방문했을 때 `x.equals(y)`는 항상 false를 반환
- SORTED: 탐색된 요소는 미리 정의된 정렬 순서를 따른다
- SIZED: 크기가 알려진 소스로 Spliterator를 생성했으므로 `estimatedSize()`는 정확한 값을 반환
- NON-NULL: 탐색하는 모든 요소는 null이 아니다
- IMMUTABLE: 이 Spliterator의 소스는 불변, 즉 요소를 탐색하는 동안 추가, 삭제, 갱신이 불가능하다
- CONCURRENT: 동기화 없이 Spliterator의 소스를 여러 스레드에서 동시에 고칠 수 있다
- SUBSIZED: 이 Spliterator 그리고 분할되는 모든 Spliterator는 SIZED 특성을 갖는다

## 커스텀 Spliterator 구현하기

문자열의 단어 수를 계산하는 단순한 메서드를 구현해 본다

먼저 반복 버전으로 구현해본다

```java
	public static int countWordsIteratively(String s) {
		int counter = 0;
		boolean lastSpace = true;

		for (char c : s.toCharArray()) {
			if (Character.isWhitespace(c)) {
				lastSpace = true;
			} else {
				if (lastSpace) { // 공백 문자를 만나면
					counter++; // 지금까지 탐색한 문자를 단어로 간주해 단어 수를 증가
				}
				lastSpace = false;
			}
		}
		return counter;
	}
```

이제 함수형으로 단어 수를 세는 메서드로 구현해본다

```java
public class WordCounter {

	private final int counter;
	private final boolean lastSpace;

	public WordCounter(int counter, boolean lastSpace) {
		this.counter = counter;
		this.lastSpace = lastSpace;
	}

	public WordCounter accumulate(Character c) {
		if (Character.isWhitespace(c)) {
			return lastSpace ? this : new WordCounter(counter, true);
		} else {
			return lastSpace ? new WordCounter(counter + 1, false) : this;
		}
	}

	public WordCounter combine(WordCounter wordCounter) {
		return new WordCounter(counter + wordCounter.counter, wordCounter.lastSpace);
	}

	public int getCounter() {
		return counter;
	}

}
```

- `accumulate()`: 새로운 WordCounter 객체를 어떤 상태로 생성할 것인지 정의
- `combine()`: 문자열 서브 스트림을  처리한 결과를 합침

```java
		final String SENETENCE = "Nel mezzo del cammin di nostra vita "
			+ "mi ritrovai per una selva oscura "
			+ "ch  la diritta via era smarrita ";

		Stream<Character> stream = IntStream.range(0, SENETENCE.length())
			.mapToObj(SENETENCE::charAt);
		WordCounter wordCounter = stream.reduce(new WordCounter(0, true),
			WordCounter::accumulate,
			WordCounter::combine);

		int count = wordCounter.getCounter();
		System.out.println("count = " + count); // 19 -> Correct Answer
```

- 먼저 String을 스트림으로 변환
  - `Stream<Character>`

위 메서드를 실행해 보면 19개의 단어가 있다고 정확한 결과가 출력된다.

그런데 위 메서드를 `parallel()`을 통해 병렬 스트림으로 처리하면 어떻게 될까?

```java
		final String SENETENCE = "Nel mezzo del cammin di nostra vita "
			+ "mi ritrovai per una selva oscura "
			+ "ch  la diritta via era smarrita ";

		Stream<Character> stream = IntStream.range(0, SENETENCE.length())
			.mapToObj(SENETENCE::charAt);
		WordCounter parallelCounter = stream.parallel()
			.reduce(new WordCounter(0, true),
				WordCounter::accumulate,
				WordCounter::combine);

		int parallelCount = wordCounter.getCounter();
		System.out.println("parallelCount = " + parallelCount); // Wrong Answer
```

<center><img src="/assets/images/modernjavainaction/ch07/10.png"></center>

잘못된 결과가 나온다.. 그 이유는 원래 문자열을 임의의 위치에서 둘로 나누다보니 예상치 못하게 하나의 단어를 둘 이상으로 계산하는 상황이 발생할 수 있다. 즉, 순차 스트림을 병렬 스트림으로 바꿀 때 스트림 분할 위치에 따라 잘못된 겨로가가 나올 수 있다

임의의 위치에서 분할하지 않고 단어가 끝나는 위치에서만 분할을 진행하면 이 문제를 해결할 수 있다

```java
import java.util.Spliterator;
import java.util.function.Consumer;
import java.util.stream.Stream;
import java.util.stream.StreamSupport;

public class WordCounterSpliterator implements Spliterator<Character> {

	private final String string;
	private int currentChar = 0;

	public WordCounterSpliterator(String string) {
		this.string = string;
	}

	@Override
	public boolean tryAdvance(Consumer<? super Character> action) {
		action.accept(string.charAt(currentChar++)); // 현재 문자를 소비
		return currentChar < string.length(); // 소비할 문자가 남아있으면 true 반환
	}

	@Override
	public Spliterator<Character> trySplit() {
		int currentSize = string.length() - currentChar;

		// 파싱할 문자열을 순차 처리할 수 있을 만큼 충분히 작으면 null 반환
		if (currentSize < 10) {
			return null;
		}

		int start = currentSize / 2 + currentChar;
		for (int splitPos = start; splitPos < string.length(); splitPos++) {
			if (Character.isWhitespace(string.charAt(splitPos))) {
				// 문자열을 파싱할 새로운 WordCounterSpliterator 생성
				Spliterator<Character> spliterator =
					new WordCounterSpliterator(string.substring(currentChar, splitPos));
				currentChar = splitPos; // 시작 위치를 분할 위치로 설정
				return spliterator; // 공백을 찾았고 문자열을 분리했으니 루프를 종료
			}
		}
		return null;
	}

	@Override
	public long estimateSize() {
		return string.length() - currentChar;
	}

	@Override
	public int characteristics() {
		return ORDERED + SIZED + SUBSIZED + NONNULL + IMMUTABLE;
	}

	public static void main(String[] args) {
		final String SENETENCE = "Nel mezzo del cammin di nostra vita "
			+ "mi ritrovai per una selva oscura "
			+ "ch  la diritta via era smarrita ";

		Spliterator<Character> spliterator = new WordCounterSpliterator(SENETENCE);
		Stream<Character> stream = StreamSupport.stream(spliterator, true); // 두 번째 인수가 true인 경우 병렬 스트림 생성
		WordCounter wordCounter = stream.reduce(new WordCounter(0, true),
			WordCounter::accumulate,
			WordCounter::combine);

		int parallelCount = wordCounter.getCounter();
		System.out.println("parallelCount = " + parallelCount); // 19 -> Correct Answer
	}

}
```

- `tryAdvance()`: 문자열에서 해당 인덱스에 해당하는 문자를 Counsumer에 제공한 다음 인덱스를 증가
  - 새로운 커서 `currentChar`의 위치가 전체 문자열의 길이보다 작으면 참을 반환해 더 탐색할 요소가 남아있음을 알린다
- `trySplit()`: 분할 동작을 중단할 한계값으로 10을 설정, 분할 과정에서 남은 문자 수가 한계값 이하이면 null을 반환해 분할을 중지한다
- `estimatedSize()`: Spliterator가 파싱할 문자열 전체 길이와 현재 반복 중인 위치의 차를 반환
- `characteristics()`: Spliterator의 특성을 알려줌
  - ORDERED, SIZED, SUBSIZED, NONNULL, IMMUTABLE

<br>

# 📌 Summary

- 내부 반복을 이용하면 명시적으로 다른 스레드를 사용하지 않고 스트림을 병렬로 처리할 수 있다
- 항상 병렬처리가 빠른 것은 아니다. 병렬 처리를 사용했을 때는 성능을 직접 측정해야 한다
- 병렬 스트림으로 데이터 집합을 병렬 실행할 때 특히 처리해야할 데이터가 아주 많거나 각 요소를 처리하는데 오랜 시간이 걸릴 때 성능을 높일 수 있다
- 기본형 특화 스트림을 사용하는 등 올바른 자료구조 선택이 병렬로 처리하는 것보다 성능적으로 더 큰 영향을 미칠 수 있다
- Fork/Join Framework에서는 병렬화할 수 있는 태스크를 작은 서브태스크로 분할해 각각의 스레드로 실행하며, 서브태스크 각각의 결과를 합쳐서 최종 결과를 생산한다
- Spliterator는 탐색하려는 데이터를 포함하는 스트림을 어떻게 병렬화할 것인지 정의한다