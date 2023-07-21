# Isolation Level

> 💡 isolation level이란?
> 
> 동시에 여러 트랜잭션이 처리될 때 트랜잭션끼리 얼마나 서로 고립되어 있는지를 나타내는 것
> 
> 즉, 동시에 여러 트랜잭션이 처리될 때 특정 트랜잭션이 다른 트랜잭션에서 수정 / 조회의 작업의 허용을 결정하는 것

## 격리 수준의 4가지
- READ UNCOMMITTED
- READ COMMITTED
- REPEATABLE READ
- SERIALIZABLE

위에서 부터 아래로 갈수록 트랜잭션 간의 고립정도가 높아지며, 성능이 떨어지는 것이 일반적이다.

일반적으로 `Oracle DB`에서는 `READ UNCOMMITTED`를 `MySQL`에서는 `REPEATABLE READ`의 격리수준을 Default로 제공한다.

## READ UNCOMMITTED
Client 측에서 요청된 SELECT 문장이 수행되는 동안 해당 데이터에 `Shared Lock`이 발생하지 않는 계층이다.

즉, 현재 작업중인 트랜잭션에서의 내용 변경 작업이 `COMMIT`이나 `ROLLBACK` 여부에