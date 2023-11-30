# 레플리케이션(Replication)이란?
여러 개의 DB를 권한에 따라 수직적인 구조(Master-Slave)로 구축하는 방식
리플리케이션에서 Master Node는 쓰기 작업 만을 처리하며 Slave Node는 읽기 작업 만을 처리하는 구조를 많이사용한다.

## 레플리케이션 처리방식
![Untitled](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbHW2YF%2FbtqKRO16Oln%2FUrvZZeMCO20q9xY0XKuKSK%2Fimg.png)

1. Master 노드에 쓰기 트랜잭션이 수행
2. Master 노드는 데이터를 저장하고 트랜잭션에 대한 로그를 파일에 기록 (BIN LOG)
3. Slave 노드의 IO Thread는 Master 노드의 로그 파일(BIN LOG)를 파일(Replay Log)를 복사
4. Slave 노드의 SQL Thread는 파일(Replay Log)를 한 줄씩 읽으며 데이터를 저장한다.

**리플리케이션은 Master와 Slave간의 데이터 무결성 검사(데이터가 일치하는지)를 하지 않는 비동기방식으로 데이터를 동기화**하는 것이 주목적이다.

## 리플리케이션의 장점과 단점
**장점**
- DB 요청의 60~80% 정도가 읽기 작업이기 때문에 Replication만으로도 충분히 성능을 높일 수 있다.
- 비동기 방식으로 운영되어 지연 시간이 거의 없다.

**단점**
- 노드들 간의 데이터 동기화가 보장되지 않아 일관성있는 데이터를 얻지 못할 수 있다.
- Master 노드가 다운되면 복구 및 대처가 까다롭다.

# 클러스팅(Clustering)이란?
여러개의 DB를 수평적인 구조로 구축하는 방식
클러스터링은 분산 환경을 구성하여 Single point of failure와 같은 문제를 해결할 수 있는 Fail Over 시스템을 구축하기 위해서 사용

동기 방식으로 노드들 간의 데이터를 동기화하는 점에서 레플리케이션과의 차이점이 존재한다.

## 클러스터링(Clustering) 처리 방식
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FoaVae%2FbtqKOCg14ow%2FkkpZDYChulrTJvyqRVKLbk%2Fimg.png)
1. 1개의 노드에 쓰기 트랜잭션이 수행되고, COMMIT을 실행한다.
2. 실제 디스크에 내용을 쓰기 전에 다른 노드로 데이터의 복제를 요청한다.
3. 다른 노드에서 복제 요청을 수락했다는 신호(OK)를 보내고, 디스크에 쓰기를 시작한다.
4. 다른 노드로부터 신호(OK)를 받으면 실제 디스크에 데이터를 저장한다.

클러스터링은 DB들 간의 데이터 무결성 검사(데이터가 일치하는지)를 하는 동기방식으로 데이터를 동기화한다.

## 클러스터링(Clustering) 장점과 단점
**장점**
- 노드들 간의 데이터를 동기화하여 항상 일관성있는 데이터를 얻을 수 있다.
- 1개의 노드가 죽어도 다른 노드가 살아 있어 시스템을 계속 장애없이 운영할 수 있다.

**단점**
- 여러 노드들 간의 데이터를 동기화하는 시간이 필요하므로 Replication에 비해 쓰기 성능이 떨어진다.
- 장애가 전파된 경우 처리가 까다로우며, 데이터 동기화에 의해 스케일링에 한계가 있다.

클러스터링을 구현하는 방법으로는 또 Active-Active와 Active-Standby가 있다. Active-Active는 클러스터를 항상 가동하여 가용가능한 상태로 두는 구성 방식이고, Active-Standby는 일부 클러스터는 가동하고, 일부 클러스터는 대기 상태로 구성하는 방식
