# ConcurrentHashMap는 어떻게 Thread-safe 한가?

## ConcurrentHashMap 동작 원리

### 1. HashMap vs ConcurrentHashMap
우선 HashMap은 동기화 관련 코드가 없기에 Multi-thread 환경에서 사용한다면 다음과 같이 전체에 lock을 걸어야 한다.
![](https://velog.velcdn.com/images/alsgus92/post/92bded22-6a1b-49be-b8e5-d3630248071c/image.png)

하지만, ConcurrentHashMap은 각각의 Bucket 별로 동기화를 진행하기에 다른 Bucket에 속해있을 경우엔 별도의 lock없이 운용한다.
[](https://velog.velcdn.com/images/alsgus92/post/afa39653-8186-4d0d-a97b-06b8ae830b50/image.png)

### 2. put
값을 넣기위해 put을 사용하면, ConcurrentHashMap에서 내부적으로 putval(key, value, onlyIfAbsent)가 호출되는데 해당 메서드에는
synchronized 키워드가 선언되어 있지 않지만, 멀티스레드 환경에서 작업이 가능하다.

어떻게 가능한것일까?

(2-1). 이전 ConcurrentHashMap 관련 포스팅시에 `Compare and Swap`을 이용하여 lock을 사용하지 않고, 삽입작업을 수행한다라고 말했었다.

![](https://velog.velcdn.com/images/alsgus92/post/ec4150e3-5372-4dd8-bea4-cc5d65538025/image.png)

1. table은 ConcurrentHashMap에서 내부적으로 관리하는 Node의 가변 Array이며, 무한 loop를 통해 삽입될 bucket를 확인한다.

2. 새로운 Node를 삽입하기 위해, tabAt()을 통해 해당 bucket을 가져오고 bucket == null로 비어 있는지 확인한다.

3. bucket이 비어 있는 경우 casTabAt()을 통해 Node를 담고 있는 volatile 변수에 접근하고 Node와 기대값 null 을 비교하여 같으면 Node를 생성해 넣고, 아니면 (1)번으로 돌아가 재시도 한다.

4. 그리고 tabAt() 과 casTabAt() 은 다음과 같고 직접 memory를 Handling 하는 UnSafe 를 이용하고 있다.

(2-2). 이미 Bucket에 Node가 존재 하는 경우 synchronized를 이용해 하나의 thread만 접근할 수 있도록 제어하며, 서로 다른 thread가 같은 Hash Bucket에 접근할 때만 해당 block이 잠기게 된다.

![](https://velog.velcdn.com/images/alsgus92/post/d38ca654-da16-47eb-a472-04f51a7d2915/image.png)

1. f는 비어있지 않은 Node<K,V> type의 hash bucket을 의미하고 이것을 통해 동기화를 한다.
2. 같은 Key 값이 들어올 경우 새로운 Node로 교체하고, putIfAbsent(key, value) 일 경우엔 값을 변경하지 않는다.
3. Hash type fh 값이 양수이고 Hash 충돌일 경우엔 Seperate Chaining에 추가한다.
4. Hash type fh 값이 음수일 경우엔 Tree에 추가한다.
5. Hash bucket의 수 binCount에 따라 Linked List로 운용할지 Tree로 운용할지 정한다.

특히, (3), (4)에선 Separate Chaining에 추가 하거나 Tree에 추가하는 것을 알 수 있는데 이는 HashMap의 내부 구현과 같다.

### 3. get
ConcurrentHashMap에서의 get을 살펴보면, synchroized keyword를 발견할 수 없다는 것을 알 수 있다. 즉, get은 가장 최신의 value 값을 return한다.

### 4. ConcurrentHashMap 생성자
ConcurrentHashMap는 총 5개 생성자를 가지고 있다.

그 중 default 생성자의 주석을 보면 초기 table size는 16으로 이루어져 있다는 것을 알수 있는데 이는 bucket의 수가 16이고 16개의 thread가 동시 쓰기를 할 수 있다는 것을 의미한다.

### 5. ConcurrentHashMap의 entrySet(), keySet(), values()
ConcurrentHashMap은 다른 HashMap 종류와 달리 자체적인 entrySet(), keySet(), values()을 가지고 있다.
즉, entrySet()에서 remove()가 발생할 경우 CocurrentHashMap에 정의되있는 remove()가 수행되기 때문에 remove()로 인한 동기화 문제는 발생하지 않게된다.