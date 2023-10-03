DB 충돌 상황을 개선할 수 있는 방법 중 DB에 Lock을 사용하는 전략을 취할 수 있다.

이러한 Lock을 사용하는 이유는 데이터베이스의 중요한 부분인 일관성과 데이터 무결성을 지키기 위해 사용된다.

대표적으로 Shared Lock과 Exclusive Lock으로 분리되고, 그 바운더리 안에서 대표적으로 사용되는 Lock의 종류를 보자

## 공유 락(Shared Lock)
- 공유 락(Shared Lock)은 읽기 락(Read Lock)이라고도 불린다
- 공유 락이 걸린 데이터에 대해서는 조회(Select) 연산만 실행이 가능하고, 데이터 삽입 시 트랜잭션이 동작하지 않는다
- 공유 락이 걸린 데이터에 대해서 다른 트랜잭션도 똑같이 공유 락을 획득할 수 있으나, **배타 락은 획득할 수 없다**
- 공유 락을 사용하면, 조회한 데이터가 트랜잭션 내내 변경되지 않음을 보장한다

## 배타 락(Exclusive Lock)
- 배타 락은 쓰기 락(Write Lock)이라고도 불린다
- 데이터에 대해 배타 락을 획득한 트랜잭션은, 읽기 연산과 쓰기 연산을 모두 실행할 수 있다
- 다른 트랜잭션은 배타 락이 걸린 데이터에 대해 읽기 작업도, 쓰기 작업도 수행할 수 없다
- 즉, 배타 락이 걸려있다면 다른 트랜잭션은 공유 락, 배타 락 둘 다 획득 할 수 없다 (해당 트랜잭션이 DB 독점)

## Lock 획득을 못한 경우

트랜잭션이 Lock을 획득하지 못하는 경우 Dead Lock 발생 가능성이 생긴다.

예를 들어, A 트랜잭션이 공유 락을 걸어 데이터 조회 작업을 수행중일 때 B 트랜잭션이 공유 락을 요청 후 수행한다면, 조회작업만을 수행하기에 허용된다.
하지만 배타락을 사용한다면 **대기**상태가 되어, 데드락을 유발할 수 있게된다.

## 공유 락, 배타 락 주의사항
- 공유 락, 배타 락의 잠금 옵션은 Auto Commit 이 비활성화 되거나, BEGIN 혹은 START TRANSACTION 명령을 통해 트랜잭션이 시작된 상태에서만 잠금이 유지된다.
- 공유 락 배타 락을 사용할 땐 각각 `SELECT FOR SHARE`,`SELECT FOR UPDATE`을 획득해야한다.
- InnoDB 스토리지 엔진을 사용하는 테이블일 경우, 락을 획득하지 않고 select 쿼리를 날렸을 때 지원이된다.

## 교착상태(Dead Lock)

![](https://hudi.blog/static/d8f59d5b092ef60cfbafc06f11da94c9/610aa/deadlock.png)

- 데드락은 서로가 점유하고 있는 자원에 대해 무한정 대기하고 있는 상황을 의미한다.
- 락은 기본적으로 특정 데이터를 점유하기에 다른 트랜잭션에서 해당 점유중인 데이터에 작업을 수행하려는 경우 대기상태에 돌입해 데드락 발생가능성이 생기는 것이다.
- 예시 트랜잭션 A에서 데이터 1의 조회 작업 후 데이터2의 데이터를 수정을 해야되는 요구사항이 존재하고 트랜잭션 B는 데이터 2의 값을 조회하고 데이터 1의 데이터를 수정해야한다고 가정해보자.
  - 이 경우 트랜잭션 B는 데이터 2에 대한 공유 락을 가지고 있어 트랜잭션 A는 배타 락을 얻기 위해 대기상태에 이른다.
  - 마찬가지로 트랜잭션 A는 데이터 1에 대한 공유 락을 가지고 있기에 트랜잭션 B는 배타 락을 얻기 위해 대기상태에 이른다.

  
## 비관적 락(Pessimistic Lock)
- 비관적 락은 `Repeatable Read` 나 `Serializable` 정도의 격리성 수준을 제공한다.
- 비관적 락이란 트랜잭션이 시작될 때 **Shared Lock** 또는 **Exclusive Lock**을 걸고 시작하는 방법이다.
- 공유 락을 걸게 되면 write를 하기위해서는 배타 락을 얻어야하는데, 공유 락이 다른 트랜잭션에 의해서 걸려 있으면 배타 락을 얻지 못해서 업데이트를 할 수 없다.
- 그렇기에 먼저 선점한 트랜잭션이 종료될 때까지 대기해야 한다.

![] (https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F5wxW6%2Fbtq9hyUsNAt%2FKtepF9HSKgm19t3G9RRoVK%2Fimg.png)

최종적으로 정리하자면, Transaction을 이용하여 충돌을 가정하고 Lock을 걸어 충돌을 예방하는 것이 바로 비관적 락(Pessimistic Lock)이다.
서버 상에서 비관적 락을 이용할 경우 트랜잭션안에서 서비스 로직이 진행되어야한다.


## 낙관적 락(Optimistic Lock)
- 낙관적 락은 값을 수정할 때, 해당 값을 수정했다고 명시하여 다른 사람이 동일한 조건으로 값을 수정할 수 없게한다
  - 이는 타임스탬프, 버전이나 해쉬코드를 사용하여 데이터 버전을 부여하고 식별
- 이러한 낙관적 락의 특징은 DB Layer에서 제공하는 특징이 아닌, Application Level에서 제공한다

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FINtDF%2Fbtq9hzFPQQU%2FIvsEoEH3yYXqL1irPPs4W1%2Fimg.png)

사진과 같이 2번의 수정 요청이 있어도 같은 값으로 수정이 일어난 경우에는 별도의 버전 컬럼을 통해 식별하여 1번의 수정요청만 발생하도록 만든다.

## Rollback
트랜잭션끼리 수정 쿼리를 전송하다 충돌이 발생할 경우, 하나 이상의 수정요청에는 롤백이라는 작업이 반드시 필요로해진다.

**비관적 락**의 경우 하나의 트랜잭션으로 묶여있기 때문에 수정이 하나 실패하면 database 단에서 전체 Rollback이 발생한다.
그렇기에, 비관적 락을 이용할 때는 트랜잭션의 분리가 중요하게 작용된다.

**낙관적 락**의 경우 말했다시피 Application Level에서 동작하기에 충돌이 발생하여 수정을 못한 부분에 대해서는 롤백에 대한 책임을 Application 단에서지며 Application에서 롤백을 수동으로 처리해야한다.

**비관적락 수도코드**
```sql
 - SELECT id, `name`
       FROM theTable
       WHERE id = 2;
 - {새로운 값으로 연산하는 코드}
 - BEGIN TRANSACTION;
 - UPDATE anotherTable
       SET col1 = @newCol1,
           col2 = @newCol2
       WHERE id = 2;
 - UPDATE theTable
       SET `name` = 'Karol2',
       WHERE id = 2;
 - {if AffectedRows == 1 }
 -     COMMIT TRANSACTION;
 -     {정상 처리}
 - {else}
 -     ROLLBACK TRANSACTION;
 -     {DB 롤백 이후 처리}
 - {endif}
```

**낙관적락 수도코드**
```sql
 - SELECT id, `name`, `version`
       FROM theTable
       WHERE iD = 2;
 - {새로운 값으로 연산하는 코드}
 - UPDATE theTable
       SET val1 = @newVal1,
           `version` = `version` + 1
       WHERE iD = 2
           AND version = @oldversion;
 - {if AffectedRows == 1 }
 -     {정상 처리}
 - {else}
 -     {롤백 처리}
 - {endif}
```

## 비관적 락 vs 낙관적 락
- 낙관적 락은 트랜잭션을 필요로하지 않는다. 따라서 Application Level에서 동작하는 낙관적 락이 성능적으로 비관적 락보다 더 좋다
- 트랜잭션이 필요하지 않기에 충돌감지 또한 가능하다
  1. 클라이언트가 서버에 정보를 요청
  2. 서버에서는 정보를 반환
  3. 클라이언트에서 이 정보를 이용하여 수정 요청
  4. 서버에서는 수정 적용 ( 충돌 감지 가능 )

- 그렇기에 충돌이 예상되지 않는 곳에 사용하면, 좋은 성능을 보장받을 수 있다.
- 하지만 롤백에서의 성능이 좋지못하다. 충돌이 났다고 한다면 이를 해결하려면 개발자가 수동으로 롤백처리를 해야하기 때문이다.
  - DB에서 지원되는 비관적 락의 경우 충돌시 그냥 롤백해버리면 되지만, 낙관적 락은 직접 이전의 코드로 수정쿼리를 날려야한다.
- 결과적으로 낙관적 락은 충돌이 많이 예상되거나 충돌이 발생했을 때 비용이 많이 들것이라고 판단되는 곳에서는 사용하지 않는 것이 효율이 가장좋다.
