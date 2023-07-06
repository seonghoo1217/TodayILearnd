# Spring Batch Basic

Spring Batch란 엔터프라이즈 시스템을 운영할 때 대용량 일괄처리에 있어 편의성을 주고 가볍고 포괄적이게 사용가능한 배치 프레임워크이다.
또한 Spring Framework의 특성을 가져온 것이기에 `DI`,`AOP`,`Service Layer 추상화`등의 기능 또한 사용가능하다.

Spring 배치는 대표적으로 몇몇 특수 상황에서 사용할 수 있다.

> 💡 배치를 사용하기 적절한 상황
> 
> 1. 대용량의 비즈니스 로직을 다루어야 할 때
> 
> 2. 스케쥴링을 통해 자동화 된 작업이 필요한 경우
> 
> 3. 대용량의 데이터를 포맷팅 또는 유효성 검사 일괄처리 등이 트랜잭션 안에서 처리 후 기록되어야할 때

Spring Batch는 로깅/추적, 트랜잭션 관리, 작업 처리 통계, 작업 재시작, 건너뛰기, 리소스 관리 등 대용량 레코드 처리 에필 수적인 재사용 가능한 기능을 제공한다.
또한 최적화 및 파티셔닝 기술을 통해 대용량 및 고성능 일괄 작업을 가능하게 하는 고급 기술 서비스 및 기능을 제공한다

## 배치 어플리케이션의 조건
- 대용량 데이터 : 대량의 데이터를 가져오거나, 전달하거나, 계산하는 등의 처리를 할 수 있어야 한다.
- 자동화 : 심각한 문제 해결을 제외하고는 사용자 개입 없이 실행되어야 한다.
- 견고성 : 잘못된 데이터를 충돌/중단 없이 처리할 수 있어야 한다.
- 신뢰성 : 무엇이 잘못되었는지를 추적할 수 있어야 한다. (로깅, 알림)
- 성능 : 지정한 시간 안에 처리를 완료하거나 동시에 실행되는 다른 애플리케이션을 방해하지 않도록 수행되어야 한다.

### Spring Batch vs Spring Quartz
Spring Quartz는 스프링 프레임워크에서 사용되는 자바 기반의 스케줄링 라이브러리이다.

그렇기 때문에 단순 스케줄러 역할을 하는 Quartz는 Batch 처럼 대용량 데이터에 대한 처리는 지원하지 않는다.
반대로 Batch 역시 Quartz의 다양한 스케줄 기능을 지원하지 않아서 보통은 Quartz + Batch를 조합해서 사용한다.

## 스프링 배치 아키텍처

**계층화 아키텍처**

![Spring_Batch_architecture](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbnFDbn%2Fbtrt1J2IjLo%2FGFyPOicHWmXhiTW2Ymgcck%2Fimg.png)

- **Application** : Spring Batch를 사용하여, 개발자가 작성한 모든 배치 작업과 사용자 정의 코드
- **Batch Core** : 배치 작업을 시작하고 제어하는 런타임 클래스를 포함
- **Batch Infrastructure**: 개발자와 애플리케이션에서 사용하는 일반적인 Reader와 Writer 그리고 RetryTemplate과 같은 서비스를 포함

위와 같은 계층화 구조 덕에 개발자는 **비즈니스 로직** 작성에 몰두할 수 있고, 배치 동작을 Batch Core를 통해 손쉽게 제어가 가능하다.

### Job Repository
![Job_Repo](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FT1ug9%2FbtrtRDW0Rtm%2Fkn0txJZU27MSmo5qDKQjZ1%2Fimg.png)

다양한 배치 수행과 관련된 수치 데이터와 잡의 상태를 유지 및 관리한다.

일반적으로 관계형 데이터베이스를 사용하며 스프링 배치 내의 대부분의 주요 컴포넌트가 공유한다.

실행된 Step, 현재 상태, 읽은 아이템 및 처리된 아이템 수 등이 모두 JobRepository에 저장된다

### Job
Job은 배치 처리 과정을 하나의 단위로 만들어 표현한 객체이며, 여러 Step 인스턴스를 포함하는 컨테이너이다.

Job이 실행될 때 스프링 배치의 많은 컴포넌트는 탄력성(resiliency)을 제공하기 위해 서로 상호작용을 한다.

### Step
스프링 배치에서 가장 일반적으로 상태를 보여주는 단위이다. 각 Step은 잡을 구성하는 독립된 작업의 단위이다.

Step에는 Tasklet, Chunk 기반으로 2가지가 있다.

### Tasklet
Step이 중지될 때까지 execute 메서드가 계속 반복해서 수행하고 수행할 때마다 독립적인 트랜잭션이 얻어진다. 초기화, 저장 프로시저 실행, 알림 전송과 같은 잡에서 일반적으로 사용된다.

### Chunk
한 번에 하나씩 데이터(row)를 읽어 Chunk라는 덩어리를 만든 뒤, Chunk 단위로 트랜잭션을 다루는 것

Chunk 단위로 트랜잭션을 수행하기 때문에 실패할 경우엔 해당 Chunk 만큼만 롤백이 되고, 이전에 커밋된 트랜잭션 범위까지는 반영이 된다.

Chunk 기반 Step은 ItemReader, ItemProcessor, ItemWriter라는 3개의 주요 부분으로 구성될 수 있다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FtNi1O%2FbtrtZ7v5HrM%2F4eLtLQJIhopD4xgbpkhJw0%2Fimg.png)

ItemReader와 ItemProcessor에서 데이터는 1건씩 다뤄지고, Writer에선 Chunk 단위로 처리된다.

보통적으로는 일반적으로 스프링 배치는 대용량 데이터를 다루는 경우가 많기 때문에 Tasklet보다 상대적으로 트랜잭션의 단위를 짧게 하여 처리할 수 있는 ItemReader, ItemProcessor, ItemWriter를 이용한 Chunk 지향 프로세싱을 이용한다.

## 배치 시나리오
![](https://deeplify.dev/assets/images/spring-batch-senario.jpg)

보통의 배치를 사용하는 시나리오는 아래와 같다.

1. 데이터베이스, 파일 또는 큐에서 데이터 읽기
2. 데이터를 정의한 방식으로 처리
3. 처리된 데이터를 데이터 쓰기