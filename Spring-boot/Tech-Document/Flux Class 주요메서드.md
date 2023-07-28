# Flux 클래스 주요 메서드

- interval : 주어진 값을 기준으로 주기적으로 값을 방출하는 메서드이다.  메서드는 일정한 시간 간격으로 숫자를 방출하는 무한한 Flux를 생성하며, 비동기 작업에 유용
  - 시그니처 메서드 : "**public static Flux<Long> interval(Duration period)**"
    - `Duration period`는 각 값 사이의 시간 간격을 나타내는 Duration 객체
    - ex.) Flux.interval(Duration.ofSeconds(1))를 호출하면 1초 간격으로 0, 1, 2, 3, ...과 같은 Long 값을 방출하는 Flux가 생성

- map : Flux 요소에 함수 연산을 수행하여 새로운 값을 생성하여 연산
  - 시그니처 메서드 : "**public final <V> Flux<V> map(Function<? super T, ? extends V> mapper)**"
    -  T는 현재 Flux의 원소의 타입을 나타내며, V는 새로운 Flux의 원소의 타입

- filter : Stream API의 filter 메서드와 유사
  - 시그니처 메서드 :**"public final Flux<T> filter(Predicate<? super T> predicate)"**

- flatmap: 각 요소를 반환하여 `Flux` 클래스 또는 `Mono` 타입으로 반환하여 이를 하나의 Flux로 만들어 반환한다.
  - 시그니처 메서드 : **"public final <R> Flux<R> flatMap(Function<? super T, ? extends Publisher<? extends R>> mapper)"**

- zip : 두개 이상의 Flux를 병합하여, 새로운 값 생성
  - 시그니처 메서드 : **"public static <T1, T2, V> Flux<V> zip(Publisher<? extends T1> source1, Publisher<? extends T2> source2,
    BiFunction<? super T1, ? super T2, ? extends V> zipper)"**

- merge : 두 개 이상의 Flux를 병합하지만 병렬로 값을 방출
  - 시그니처 메서드 : **"public static <T> Flux<T> merge(Publisher<? extends T>... sources)"**

- concat : 두개 이상의 Flux를 이어붙인다.
  - 시그니처 메서드 : **"public static <T> Flux<T> concat(Publisher<? extends T>... sources)"**

- combineLatest : 두개 이상의 Flux 중에서 가장 최근에 값을 방출한 요소들을 결합
  - 시그니처 메서드 : **"public static <T1, T2, V> Flux<V> combineLatest(Publisher<? extends T1> source1, Publisher<? extends T2> source2,
    BiFunction<? super T1, ? super T2, ? extends V> combinator)"**

- reduce : Flux의 요소들을 하나의 값을 줄여나가면서 최종 결과를 생성
  - 시그니처 메서드 : **"public final Mono<T> reduce(BiFunction<T, T, T> reducer)"**

- collectList or collectMap : Flux의 요소들을 리스트나 맵으로 변경시킨다.
  - 시그니처 메서드 : **"public final Mono<List<T>> collectList()"**
  - 시그니처 메서드 : **"public final <K> Mono<Map<K, T>> collectMap(Function<? super T, ? extends K> keyMapper)"**

- doOnNext, doOnError, doOnComplete: 각 상태별로 특정 작업을 수행합


## zip과 merge의 차이
zip은 여러 Flux를 조합하여 튜플로 만들고, merge는 여러 Flux를 병합하여 순서대로 방출(LinkedList의 성질).

**zip 메서드 사용예시**
```java
Flux<Integer> numbers1 = Flux.just(1, 2, 3);
Flux<Integer> numbers2 = Flux.just(10, 20, 30);

Flux<Tuple2<Integer, Integer>> zipped = Flux.zip(numbers1, numbers2);

zipped.subscribe(tuple -> System.out.println(tuple.getT1() + ", " + tuple.getT2()));
```

```markdown
// 실행 결과
1, 10
2, 20
3, 30
```

**merge 메서드 사용예시**
```java
Flux<Integer> numbers1 = Flux.just(1, 2, 3).delayElements(Duration.ofMillis(100));
Flux<Integer> numbers2 = Flux.just(10, 20, 30).delayElements(Duration.ofMillis(50));

Flux<Integer> merged = Flux.merge(numbers1, numbers2);

merged.subscribe(System.out::println);
```

```markdown
// 실행 결과
10   // numbers2에서 10이 방출됨
1    // numbers1에서 1이 방출됨 (numbers2의 10과 병합되는 시점)
20   // numbers2에서 20이 방출됨
2    // numbers1에서 2가 방출됨 (numbers2의 20과 병합되는 시점)
30   // numbers2에서 30이 방출됨
3    // numbers1에서 3이 방출됨 (numbers2의 30과 병합되는 시점)
```

## doOnNext, doOnError, doOnComplete

1. doOnNext
   - doOnNext 연산자는 Flux나 Mono가 데이터를 방출할 때 해당 데이터를 처리하기 전에 추가적인 동작을 수행하도록하는 메서드
   - 데이터가 방출될 때마다 호출되며, 데이터를 소비하지 않고 관찰
   - 보통 로그 출력이나 데이터 변환 등의 사이드 이펙트를 처리하는 데 사용

2. doOnError
   - doOnError 연산자는 Flux나 Mono에서 에러가 발생했을 때, 에러를 처리하기 전에 추가적인 동작을 수행하도록하는 메서드.
     - jquery의 error 성질
   - 에러가 발생했을 때 호출
   - 보통 에러 로깅이나 에러 처리에 사용된다.

3. doOnComplete
   - doOnComplete 연산자는 Flux나 Mono의 데이터 방출이 완료되었을 때 추가적인 동작을 하도록하며 doOnError와는 반대의 개념
     - jquery의 then 성질
   - 작업 완료시점의 수행가능한 메서드