임계영역에 대해 동기화문제를 해결하기 위해선 총 세가지의 조건이 만족되어야한다.

1. mutual exclution 상호배제
2. bounded waiting 한정된 대기
3. prograss 진행의 융통성

임계영역에 대한 동기화 방법으로 항상 거론되는 것이 Mutex와 Semaphore이다.

**들어가기전 , 다른 방법도 분명히 존재하지만**

하드웨어적 해결방법, 피터슨, 데커 알고리즘 3개는 **busy wating**을 사용하기 때문에 자원 낭비도 심하고 알고리즘도 복잡하다.

# 세마포어: Semaphore

세마포어는 critical section에 진입하기 전에 스위치를 사용 중으로 놓고 임계구역으로 들어간다.

이후에 도착하는 프로세스는 앞의 프로세스가 작업을 마칠 때까지 기다린다. 프로세스가 작업을 마치면 세마포어는 다음 프로세스에 임계구역을 사용하라는 동기화 신호를 보낸다.

세마포어는 다른 알고리즘과 달리 임계구역이 잠겼는지 직접 점검하거나 busy waiting을 하거나, 다른 프로세스에 동기화 메시지를 보낼 필요가 없다.

### 세마포어의 프로세스

1. `Semaphore(N)` 으로 공유 가능한 자원의 수를 초기설정한다,
2. `P()` 함수를 수행함으로서 임계구역에 들어가기 전에 사용 중이라는 표시를 한다
3. `V()`으로 임계구역에 나올 때 비었다고 표시

```c
Semaphore(n);
P()
//임계구역
V()
```

### 코드적인 접근

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/db74ba2d-24e4-4d4b-ae5c-07d99cd33ca1/1ce2aa2d-7d28-4646-ae4f-d081cedb7a0f/Untitled.png)

세마포어는 `sleep - wakeup` 구조로 이루어져 있다.

그렇기에 실제로 대기에는 `sleep(block)` 함수를 사용하며 사용가능 신호를 보낼때는 `wakeup()` 함수를 사용한다.

1. Semaphore(n)
    - 자원의 수를 저장하는 전역변수 RS를 n값으로 초기화한다.
2. P()
    - 잠금을 수행하는 코드로 RS가 0보다 크면(사용 가능한 자원이 있다면) 1만큼 감소시키고 임계구역에 진입
    - RS가 0보다 작으면(사용할 수 있는 자원이 없으면) 0보다 커질 때까지 기다린다.
3. V()
    - 잠금 해제와 동기화를 같이 수행하는 코드로, RS 값을 1 증가시키고 세마포어에서 기다리는 프로세스에 임계구역에 진입해도 좋다는 wake_up 신호를 보낸다.

세마포어에서 잠금이 해제되기를 기다리는 프로세스(or 스레드)는 세마포어 큐에 저장되어 있다가, **V()를 통해 wakeup 상태가 되면,** 큐에 나와 임계구역에 진입한다.

## 세마포어 단점

P(),V()가 실행 중일 때, `context switching`이 발생하면 **mutual exclution과 bounded waiting 조건을 보장하지 못한다.**

이를 위해 원자성을 보장하여, 분리실행이아닌 하나의 프로세스내에서 수행되어야한다.

# 뮤텍스: Mutex

뮤텍스는 사실 세마포어의 공유자원(RS)가 1일 때, 즉 하나의 스레드만이 공유자원을 점유하는 방법

뮤텍스와 세마포어는 구동 방법, 원리가 모두 같다.

```c
wait(mutex);
// critical section
signal(mutex);
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/db74ba2d-24e4-4d4b-ae5c-07d99cd33ca1/794d9f20-b2e9-4abe-a9dc-61cd47ee35ab/Untitled.png)

뮤텍스는 `wait(mutex)`, `signal(mutex)`로 이루어져 있지만, 실제 세마포어의 P(),V()연산과 동일한 연산을 수행한다.

`wait`을 하면 공유자원 mutex(이전에는 RS)를 하나 감소시키고, `signal` 하면 mutex를 하나 증가시키고 block 상태에 빠져있는 프로세스를 뮤텍스 큐에서 깨워 ready queue로 보내는 역할을 한다.

# 모니터: Monitor

세마포어의 가장 큰 문제는 잘못된 사용으로 인해 임계구역이 보호받지 못한다는 것이다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/db74ba2d-24e4-4d4b-ae5c-07d99cd33ca1/9f9866d7-50e0-4d4f-a36e-b71be0115b0f/Untitled.png)

다음은 세마포어의 잘못된 사용으로 문제가 발생한 경우이다.

1. 세마포어를 사용하지 않고 임계구역에 들어갈 때, 임계구역이 보호받지 못한다. 즉 mutual exclusion이 보장되지 않아 `race condition`이 발생한다.
2. `P()`만 두 번 사용하고 `V()`를 사용하지 않아서 wake_up 신호가 발생하지 않은 경우 무한대기상태가 발생한다.
3. `P()`와 `V()`를 반대로 사용하여 mutual exclusion이 보장되지 않는다.

만약, 공유자원을 사용할 때 모든 프로세스가 세마포어 알고리즘을 따른다면 굳이 `P()`, `V()`를 사용할 필요 없이 자동으로 처리하도록 하면 된다. 이를 실제로 구현한 것이 모니터이다.

모니터는 공유 자원을 내부적으로 숨기고 공유 자원에 접근하기 위한 인터페이스만 제공함으로써 자원을 보호하고 프로세스 간에 동기화를 수행한다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/db74ba2d-24e4-4d4b-ae5c-07d99cd33ca1/0f791cdb-e92f-48ba-a131-a0f7bc306ac5/Untitled.png)

모니터는 system call과 같은 개념으로 생각하면 된다.

운영체제의 자원을 일반 사용자가 접근하는 것을 허용하지 않기 위해서 시스템 자원을 숨기고 인터페이스만 제공하는 **system call**처럼, 모니터도 보호할 자원을 임계 구역으로 숨기고 임계구역에서 작업할 수 있는 인터페이스만 제공하여 자원을 보호한다.

모니터의 특징은 다음과 같다.

1. 임계구역으로 지정된 변수나 자원에 접근하고자 하는 프로세스는 직접 `P()`, `V()`를 사용하지 않고 모니터에 작업을 요청한다.
2. 모니터는 요청받은 작업을 모니터 큐에 저장한 후 순서대로 처리하고 그 결과만 해당 프로세스에 알려준다.