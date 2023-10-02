# Synchronized
- 멀티스레드 환경에서 하나의 공유자원을 여러 스레드에서 동시에 접근하여 사용하게 되면 Race Condition, Dead Lock이 발생할 수 있다.
- 이러한 문제를 해결하기 위해 **`스레드를 동기화`**하는 전략을 구성하여, 에러를 방지한다.

## 스레드 동기화
- 스레드 동기화는 멀티스레드 환경에서 여러 스레드가 하나의 공유자원에 동시에 접근하지 못하도록 막는것을 뜻한다.
- 공유데이터가 사용되어 동기화가 필요한 부분을 **임계영역(critical section)** 이라고 지칭한다.
  - 해당 임계영역에 synchronized 키워드를 사용하여 여러 스레드가 동시에 접근하는 것을 금지함으로써 동기화를 할 수 있습니다. 

## synchronized 사용
- synchronized 키워드는 동기화가 필요한 대상 (메소드, 코드블럭)앞에 사용하여 동기화할 수 있다.
- synchronized로 지정된 임계영역은 한 스레드가 이 영역에 접근하여 사용할때 lock이 걸림으로써 다른 스레드가 접근할 수 없게 된다.
- 해당 스레드가 이 임계영역의 코드를 다 실행 후 unlock 상태가 되면, 대기중이 다른 스레드가 이 임계영역에 접근하여 다시 lock을 걸던 사용