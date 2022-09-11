# Redis란?

* 레디스는 고성능 키-값 저장소로서 문자열, 리스트, 해시, 셋, 정렬된 셋 형식의 데이터를 지원하는 NoSQL이다.

사실 이 한줄로 정리되는 인메모리 데이터베이스이지만 좀더 자세히 풀어보도록하자.

Redis는 Memcached와 비슷한 캐시 시스템으로서 동일한 기능을 제공하며 영속성, 다양한 데이터 구조와 같은 부가적인 기능을 지원한다.
레디스는 모든 데이터를 메모리에 저장하고 조회할 수 있으면서 캐시의 성질을 가지고 있는 즉 , 인메모리 데이터베이스이다.
이 말만 들으면 Redis는 모든 데이터를 메모리에 저장하는 빠른 DB라고 생각되지만 빠른 성능은 Redis의 특징 중 일부일 뿐 Redis를 사용하였을때 
얻는 이점은 바로 다른 인메모리 디비에는 없는 다양한 자료구조를 지원해준다는 점이다.

![](https://miro.medium.com/max/700/1*tMiZs3RCrmxLGiFZgWRP6g.png)

이렇게 다양한 자료구조를 지원하여 주면 개발의 편의성이 높아지고 난이도가 낮아진다는 장점이 있다.
예를 들어 어떤 데이터를 정렬 해야하는 상황이 있을떄 DBMS를 사용한다면 DB에 데이터를 저장하고 저장된 데이터를
다시 읽어오는 과정에서 쿼리가 나가기 떄문에 디스크에 직접접근해야하는 시간적 손해가 크다.
또한 DB에 무리가 크게 가기도 한다.

하지만 Redis를 이용하면 레디스에서 제공하는 Sorted-Set이라는 자료구조를 사용하면 더빠르고 간편히 정렬할 수 있다.

이를 위해 Redis를 캐시서버로 사용하며 캐시의 경우 한번 읽어온 데이터를 임의의 공간에 저장하여 다음에 읽을 땐 빠르게 결과값을
받을 수 있도록 해주기 때문에 DB의 부하를 줄여주고 서비스의 속도 성능도 보장하여준다.

또한 Redis는 NoSql로서 Key-Value 형태로 비정형의 값을 저장하고 관리한다.
Redis는 다음과 같은 특징 다섯 가지를 가지게 된다.

1. 영속성을 지원하는 인메모리 데이터 저장소이다.
2. 읽기 성능 증대를 위한 서버 측 복제를 지원한다.
3. 쓰기 성능 증대를 위해 클라이언트 측에 *샤딩(sharding)을 지원한다.
4. 다양한 서비스에서 사용되며 검증된 기술이다.
5. 문자열, 리스트, 해쉬, 셋, 정렬된 셋과 같은 다양한 데이터형을 지원해준다. 메모리 저장소임에도 불구하고 많은 데이터형을 지원하므로 다양한 기능을 구현할 수 있다.

* 샤딩(Sharding) : 같은 테이블 스키마를 가진 데이터를 다수의 데이터베이스에 분산하여 저장하는 방법을 의미한다.

## Redis의 영속성

레디스는 지속성을 보장하기 위해 데이터를 Disk에 저장하여 서버가 중단되더라도 Disk에 저장된 데이터를 읽어서 메모리에 로딩을한다.

데이터를 Disk에 저장하는 방식은 총 2가지 방식이 있다.

1. RDB(Snapshotting) 방식

-> 순간적으로 메모리에 있는 내용을 Disk에 전체로 옮겨 담는 방식

2. AOF(Append On File) 방식

-> Redis의 모든 write/update 연산 자체를 모두 log 파일에 기록하는 형태


## Redis 명령어

레디스를 실행 하는 명령어는 다음과 같다. Window라면 직접 Redis를 설치하여야하고 맥의 경우 brew 명령어를 통해 설치한다.

brew install redis (Mac Redis 설치)
redis-server (Redis Server 실행)
redis-cli (Redis Client 접속)

brew services start redis   (Redis 서비스 실행)
brew services stop redis    (Redis 서비스 중지)
brew services restart redis (Redis 서비스 재시작)

flushAll (Redis 모든 Key 삭제)

Redis의 데이터 입력은 set key value의 문법으로 작성하고 데이터 출력은 get key로 key에 해당하는 value값을 출력한다.
ex) set key1 "Kim" ,get key1을하면 "kim"이 출력

문자열 추가 명령어 append이다
ex) append key1 " Bomin" get key1을하면 "Kim Bomin"이 출력된다.

sadd 명령어를 통해서 Set 자료구조를 사용할 수 있고 smembers 명령어를 통해 Set 자료구조를 출력한다.
ex) sadd set:test my , sadd set:test name , sadd set:test is Lee ,smembers set:test

zadd 명령어로 sortedSet 명령어를 사용할 수 있다.

hset 명령어로 HashMap 자료구조를 사용할 수 있다.