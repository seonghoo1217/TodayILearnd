## 1. Serial GC

- 단일 스레드로 동작하며 모든 애플리케이션 스레드를 중단(stop-the-world) 한 스레드가 메모리를 정리
- 주로 메모리가 작거나 단일 코어 환경에서 사용한다.

## 2. Parallel GC

- 여러 개의 GC 스레드를 통해 `stop-the-world` 를 빠르게 처리
- 오래된 Java(8 이하)에서 **기본 GC**로 많이 쓰였으며, 현재도 고성능 환경에서 선호될 수 있음

## 3. CMS(Concurrent Mark-Sweep)

- 애플리케이션과 병렬적으로 안쓰는 메모리를 수거했다.
- Java 8버전에서 주로 사용했으며 이후 Java 9부터는 `G1`  버전 사용

### CMS의 단편화 이슈

CMS는 힙(old)을 `Mark & Sweep` 알고리즘 방식으로만 회수하기 때문에 사용이 끝난 객체 영역을 연속적으로 압축하진 못한다.

그 결과 메모리 단편화가 누적되면 새 객체 할당 시 큰 연속 공간을 찾기 아려워지고  결국 Full GC(Stop-the-world)로 이어져서 성능이 떨어진다.

## 4. G1(Garbage-First) GC

- Java 9부터 기본 GC
- 힙을 region(영역) 단위로 나누어, 이용률이 낮은 영역부터 우선적으로 수집한다.
- Stop-the-world 시간을 예측 가능하게 줄이는데 중점
- 범용 GC로서 Java 8 ~ 11까지 사용

### G1의 Pause 이슈

G1에서 Stop-the-world 시간을 짧게 유지하려고하는데 예상치 못하게 긴 **`Pause`**가 발생하는 경우

**1) Evacuation Pause에서의 병목**

- **Evacuation(객체 이동) 단계**는 대부분 **Stop-the-world(STW)** 형태로 진행됩니다.
- G1은 Marking은 상당 부분 **동시(Concurrent)로** 수행할 수 있지만, **정리(Evacuate) 단계**에서 객체를 새 Region으로 옮길 때 애플리케이션 스레드를 일시 중단
- 힙이 매우 커지거나 Old 영역에 객체가 많이 쌓인 경우, **이 Evacuation이 길어지면** Pause가 늘어날 수 있다.

**2) To-space Exhausted(대상 영역 부족)**

- Evacuation 중, 복사할 객체를 담을 “새 Region(‘to-space’)”이 충분치 않다면, G1은 **즉시 Full GC** 모드로 전환할 수 있다.
- **Full GC**는 단일 스레드 Serial 방식으로 전체 힙을 정리하므로, **Pause가 급격히 길어지는** 최악의 시나리오가 된다.

**3) Mixed GC에서 Old 영역 처리량 부족**

- G1은 Young 수집과 달리, Old 영역의 객체가 많을수록 Mixed GC 사이클에서 처리해야 할 Region이 늘어난다.
- **동시 마크** 단계와 STW로 진행되는 **처리 단계**(cleanup, selective evacuation)가 균형을 못 맞추면, “목표 Pause 시간을 초과”하는 상황이 발생할 수 있다.

**4) Marking이 지연되어 “Concurrent Cycle”이 밀릴 때**

- G1의 **Concurrent Mark**가 애플리케이션 스레드와 병행되지만, CPU가 부족하거나 객체 그래프가 복잡하면 마크가 늦어지고, 다음 Young GC가 트리거될 때 “아직 Old 영역 어떤 객체가 살아있는지 모름 → 예기치 않은 추가 작업”이 발생해 **Pause가 증가**할 수 있다.

## 5. ZGC

- 아주 낮은 지연시간을 목표로 설계된 GC로서 대용량 힙과 초저지연 연산을 지원한다.
- Java 11에서 처음 도입
- Java 15에서 정식 도입
- 동시(Concurrent)로 대부분을 처리하여 `Stop-the-world` 를 수 밀리초 수준으로 억제

## 6. Shenandoah(셰넌도어)

- 저지연 GC(Concurrent+ Region 기반)
- '큰 GC 작업을 적은 횟수로 수행하는 것보다 작은 GC 작업을 여러번 수행하는게 더 좋다'는 개념을 적용해 만들어진 GC
    - 작은 단위의 GC수행을 자주하기 때문에 `Concurrency`를 보장한다.
    - GC가 CPU를 더 사용하는 대신 pause 시간을 줄인다.

### Shenandoah의 특징

- 기존 CMS가 가진 단편화, G1이 가진 Pause 이슈를 해결했다.
- 강력한 Concurrency와 가벼운 GC 로직으로 Heap 사이즈에 영향을 받지 않고 일정한 Pause 시간이 소요된다.

### Single-Generational

Shenandoah GC는 G1 GC를 기반으로 만들어졌다.

heap을 region 별로 나누고 `mark-seep & evacuation`을 진행한다. 그러나 Shenandoah GC는 G1 GC와 다르게 Generation으로 영역을 나누지 않는다. Generation을 나누게 되면 영역을 나누고 evacuation과정에서 메모리 copy가 계속 발생한다. 그러나 Generation이 나누어지지 않으면 memory copy에 대한 비용이 없어지게 되어 더 효율적

### GC는 왜 메모리를 여러 영역으로 나눌까?

- 대부분의 객체는 생성 후 얼마 안되어 사용이 끝나지만, 반대로 한 번 오래 살아남은 객체는 상대적으로 계속살아남는다.
- 그렇기에 이특성을 활용해서 `young` 과 `old` 로 분리해 관리하면 전체 GC 성능을 크게 높일 수 있다.

### **Shenandoah GC의 동작 과정**

![image.png](attachment:44aa5801-914b-4430-bd30-a34e57722b55:image.png)

1. Initial Marking
2. Concurrent Marking
3. Final Marking
4. Concurrent Compaction

pause후 root set을 스캔하고 **`Java 스레드와 동시에 marking`**을 수행한다. 그 이후 한번 더 pause를 수행하여 final marking을 수행하고 압축 마지막 단계에서 marking된 객체들을 제거. 전체적인 흐름은 CMS GC와 비슷하다.ㅌ