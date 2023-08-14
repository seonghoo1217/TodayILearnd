ConcurrentHashMap 기존의 HashMap,HashTable과 유사한 구조를 가지고 있으나 멀티스레드환경에 동시성을 위해 설계된 클래스이다.
멀티스레드 애플리케이션에서 여러 스레드가 동시에 맵에서 데이터를 읽고 쓰거나, 따로 동기화 블록을 사용하는 대신 ConcurrentHashMap이 제공하는 동시성 제어 메커니즘을 활용하려면 ConcurrentHashMap을 사용할 수 있다.

## 유사 컬렉션 프레임워크

**HashTable** 
```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {

    public synchronized int size() { }

    @SuppressWarnings("unchecked")
    public synchronized V get(Object key) { }

    public synchronized V put(K key, V value) { }
}
```
Hashtable 클래스의 대부분의 API를 보면 위와 같이 메소드 전체에 synchronized 사용하기에, Multi-Thread 환경에서 효율이 좋을 것 같지만, 동시에 접근하여 작업시에
객체마다 가지고 있는 고유한 Lock 때문에 여러 작업 수행시에 **병목현상**이 발생할 수 밖에 없다.

> * 병목현상 : 메소드에 접근시 현재 작업이 수행중이라면, 다른 쓰레드는 Lock을 얻을 때까지 기다리는 현상

그렇기에 Hashtable 클래스는 Thread-safe 하다는 특징이 존재하지만, 멀티쓰레드 환경에서 사용하기에도 살짝 느리다는 단점이 있으며, 레퍼런스 또한 오래되어 잘 사용하지 않는 추세

**HashMap**
Java에서 Map의 역할을 수행할 때는 HashMap이 보통적으로 사용된다.

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

    public V get(Object key) {}
    public V put(K key, V value) {}
}
```

하지만, HashMap의 가장 큰 단점은 synchronized 즉, 더 빠른 접근 속도와 **해시충돌** 최적화등 성능은 가장 좋지만, Multi-Thread 환경에 사용할 수 없다.

멀티 쓰레드 환경이 아니라면 HashMap을 사용하기에 대체적으로 적합하겠지만, 멀티쓰레드의 환경이라면 HashMap 클래스도 Hashtable 클래스의 대안점으로는 불가능하다.


## ConcurrentHashMap
Hashtable 클래스의 단점을 보완하면서 Multi-Thread 환경에서 사용할 수 있도록 나온 클래스가 바로 ConcurrentHashMap으로 동시성을 높이기 위해 고안된 클래스이다.

```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {

    public V get(Object key) {}

    public boolean containsKey(Object key) { }

    public V put(K key, V value) {
        return putVal(key, value, false);
    }

    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
}
```
위는 ConcurrentHashMap 클래스의 일부 API이다.

ConcuurentHashMap에는 Hashtable 과는 다르게 synchronized 키워드가 메소드 전체에 붙어 있지않다.

**🧐그렇다면 대체 어떻게 Multi-Thread 환경에서 쉽게 사용될까?**
 
이는 ConcurrentHashMap의 특징 때문인데, ConcurrentHashMap은 읽기 작업에는 여러 쓰레드가 동시에 읽을 수 있지만, 쓰기 작업에는 특정 세그먼트 or 버킷에 대한 Lock을 사용한다.

그렇기에, 실제 저장 로직부분에서는 일부분적으로는 단일 스레드처럼 작용하게 된다.


```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
        implements ConcurrentMap<K,V>, Serializable {

    private static final int DEFAULT_CAPACITY = 16;

    // 동시에 업데이트를 수행하는 쓰레드 수
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
}
```
ConcurrentHashMap 클래스의 경우  `DEFAULT_CAPACITY`, `DEFAULT_CONCURRENCY_LEVEL`가 각각 ConcurrentHashMap에서 작업이 가능한 버킷의 수, 동시에 작업이 가능한 쓰레드의 수(세그먼트 수)를 나타낸다.

**버킷의 수 == 동시작업 가능한 쓰레드 수**인 이유는 위에서 말했던 것처럼 ConcurrentHashMap은 버킷 단위로 lock을 사용하기 때문에 같은 버킷만 아니라면 Lock을 기다릴 필요가 없다.

즉, 버킷당 하나의 Lock이 부여되는 셈이다.

여러 쓰레드에서 ConcurrentHashMap 객체에 동시에 데이터를 삽입, 참조하더라도 그 데이터가 다른 세그먼트에 위치하면 서로 락을 얻기위해 경쟁이 일어나지 않아, 데드락같은 상태도 회피할 수 있다.


## ConcurrentHashMap 동기화
![](https://user-images.githubusercontent.com/45676906/107613945-af778900-6c8c-11eb-8f1b-5b0f8bce4bcb.png)

ConcurrentHashMap `Compare and Swap` 방식을 이용하여 빈 버킷에 노드를 삽입한다.
이는 synchronized 키워드를 사용하지않고 Java에서 동기화하는 방법 중 하나이다.

![](https://user-images.githubusercontent.com/45676906/107615378-670d9a80-6c8f-11eb-8a76-766e7595c0fc.png)

버킷에 이미 Node가 있을 때, 즉 하나의 스레드에서 이미 세그먼트를 사용하여 Lock이 걸려있을때를 판단하기 위해선 `synchronized` 키워드를 이용한다.

## ConcurrrentHashMap 생성자
![](https://user-images.githubusercontent.com/45676906/107616159-d932af00-6c90-11eb-81b6-fb32864bdb81.png)

ConcurrentHashMap의 생성자에는 기본적으로 `conrrencyLevel`를 지정할 수 있다. 이를 효율적으로 설정한다면 동시성에서 Lock획득 시간 단축과 같은 이슈를
해결할 수 있을 것이다.

데이터를 너무 적은 수의 조각으로 나누면 경쟁을 줄이는 효과가 적을 것이고 너무 많은 수의 조각으로 나누면 이 세그먼트를 관리하는 비용이 커지기 때문에 서비스 특성에 맞는 값을 설정해야한다.

## ConcurrentHashMap 사용시기
단순 조회(Select)작업 보다는 쓰기 작업에 성능이 중요한 상황에서 쓰면 적합하다고 보인다.

### 🧐Select에서의 동기화는 지원이 안되잖아!
검색(get())에는 동기화가 적용되지 않으므로 업데이트 작업(put() 또는 remove())과 겹칠 수 있어 항상 최신 결과값을 반환한다.

## 가변 배열 리사이징

HashMap에서 버킷안에 노드가 로드팩터 값에 도달하게 되면 단순히 resize() 메소드를 통해 새로운 배열을 만들어 copy 하는 방식을 사용하지만,
ConcurrentHashMap 에서는 기존 테이블을 새로운 테이블로 버킷을 하나씩 전송(transfer) 하는 방식을 사용한다.

