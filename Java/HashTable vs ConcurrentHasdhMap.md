TODO 리스트를 제작하는 스터디를 진행하며 In-Memory 환경으로 DB를 구성해야 하기 위해 구조체에 대한 고민이 있었다.

맨 처음의 경우 HashTable을 사용하려고 했다.

얼핏 아는 지식으로 HashMap의 경우 멀티 스레드 환경에서는 사용할 경우 동시성이 보장되지 않기 때문에 고려 대상이 아니었기에 HashTable을 사용하고자 하였다.

하지만 이에 대해 공부를 하며 알게된 사실에 대해 정리하고자한다.

> 🤔우선 왜 HashMap은 안씀?

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/db74ba2d-24e4-4d4b-ae5c-07d99cd33ca1/09c8a81f-afc4-46fc-b96f-ab181ffa9ebe/Untitled.png)

- HashMap의 경우 비동기화되어 `synchronized`를 보장받지 못해 멀티스레드 환경에서 안전성을 보장받지 못한다.
- 여러 스레드가 동시에 `HashMap` 객체를 수정하거나 삽입할 수 있어 무결성에 문제가 생김
    - 물론 투두 리스트가 그만한 동시성이슈가 발생할까? 라는 고민은 있지만 스터디가 목적이기에 고려사항으로 두어야함
- 또한 가장 중요한 부분은 null을 Key값으로 허용하며, value로서도 허용함
- `싱글 쓰레드 환경`에서 사용하는 게 좋다. 한편, 동기화 처리를 하지 않기 때문에 데이터를 탐색하는 속도가 빠르다
- 멀티 스레드 환경에서 사용하기 위해선, 위 그림과 같이 전체에 Lock을 걸어야함
- 결론적으로 `HashTable`과 `ConcurrentHashMap`보다 데이터를 찾는 속도는 빠르지만, 신뢰성과 안정성이 떨어진다.

## HashMap의 내부구조

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {

    ...

    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1)
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) {
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

    ...
}
```

- Map의 put을 이용하여 값을 삽입하는순간 putavl이라는 메서드의 리턴 값을 전달한다.
- 로직을 보면 동기화처리가 별도로 존재하지 않기에 삽입 / 조회 연산이 빠르다.

# HashTable

- In-Memory로 과제를 진행하기 위해서 맨처음 로컬 스토리지로 생각했던 대안이 HashTable이다.
- HashTable의 경우 ThreadSafe할 뿐만 아니라 메소드를 호출하기 전에 쓰레드간 동기화 락을 걸기에 멀티 쓰레드 환경에서 무결성이 보장된다.
- 하지만, 동기화 락은 매우 느린 동작이라는 단점이 존재한다.

# ConcurrentHashMap

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/db74ba2d-24e4-4d4b-ae5c-07d99cd33ca1/8f2d4078-f58e-4850-b7b0-5d37f2b05167/Untitled.png)

- ConcurrentHashMap 또한 thread-safe하기 때문에, `멀티 쓰레드 환경`에서 사용할 수 있다.
- 각각의 Bucket 별로 동기화를 진행하기에 다른 Bucket에 속해있을 경우엔 별도의 lock없이 운용한다.
- 이 구현체는 HashMap의 동기화 문제를 보완하기 위해 나타났다
- 동기화 처리를 할 때, 어떤 Entry를 조작하는 경우에 해당 Entry에 대해서만 락을 건다. 그래서 HashTable보다 데이터를 다루는 속도가 빠르다.
    - 즉, Entry 아이템별로 락을 걸어 멀티 쓰레드 환경에서의 성능을 향상시킨다

# 지표로 분석해보자

```java
public class MapConcurrencyTest {
    static final int ITERATION_COUNT = 10000000;
    static CountDownLatch latch;
    static Map<Integer, Integer> map;
    static List<Long> times;

    private void init(Map<Integer, Integer> m) {
        latch = new CountDownLatch(20);
        map = m;
        for (int i = 1; i <= 1000; i++) {
            map.put(i, i);
        }
    }

    @Test
    public void test() throws InterruptedException {
        for (Map m : new Map[]{
                new HashMap<Integer, Integer>(),
                new ConcurrentHashMap<Integer, Integer>(),
                new Hashtable<Integer, Integer>()
        }) {
            times = new ArrayList<>();

            for (int t = 0; t < 10; t++) {
                init(m);

                long start = System.currentTimeMillis();

                // 쓰기 작업 쓰레드 10개
                for (int counter = 0; counter < 10; counter++) {
                    new Thread(() -> {
                        for (int i = 1; i <= ITERATION_COUNT; i++) {
                            int key = i % 1000 + 1;
                            map.put(key, key);
                        }
                        latch.countDown();
                    }).start();
                }

                // 읽기 작업 쓰레드 10개
                for (int counter = 0; counter < 10; counter++) {
                    new Thread(() -> {
                        for (int i = 1; i <= ITERATION_COUNT; i++) {
                            int key = i % 1000 + 1;
                            map.get(key);
                        }
                        latch.countDown();
                    }).start();
                }

                // 모든 쓰레드 작업 대기
                latch.await();

                long time = System.currentTimeMillis() - start;
                times.add(time);
            }

            System.out.println("Map: " + m.getClass().getSimpleName());
            System.out.println("Execution Times: " + Arrays.toString(times.toArray()));
            System.out.println("Average execution Time: " + times.stream().mapToLong(Long::longValue).average().getAsDouble() + "ms");
            System.out.println();
        }
    }
}
```

![스크린샷 2023-12-12 오후 2.44.43.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/db74ba2d-24e4-4d4b-ae5c-07d99cd33ca1/01f87e5d-1da2-4709-ac06-9a493614d5df/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-12-12_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_2.44.43.png)

이렇게 차이가 나는 이유는 `Hashtable`은 조회와 삽입 작업에 모두 `synchronized`를 통해 테이블 전체를 lock걸어  반면, `ConcurrentHashMap`은 조회 작업에는 lock을 걸지 않고, 삽입 작업에서만 사용되는 키 값에 따라 세분화된 lock을 제공하여 여러 스레드가 동시에 다른 부분을 수정할 수 있다

# **정리**

|  | HASHMAP | HASHTABLE | CONCURRENTHASHMAP |
| --- | --- | --- | --- |
| key와 value에 null 허용 | O | X | X |
| 동기화 보장(Thread-safe) | X | O | O |
| 추천 환경 | 싱글 쓰레드 | 멀티 쓰레드 | 멀티 쓰레드 |