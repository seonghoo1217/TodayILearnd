## 중복 키 동시 insert로 인한 데드락
- 해당 Case의 경우 `Unique Index`로 인해 데드락이 발생할 수 있는 케이스에 대해 정리한다.

이러한 시나리오를 가정하여보자, 당신의 프로젝트에서 Entity의 PK를 노출시키고 싶지 않아 UUID Type의 Unique한 필드 값을 보조 키로 사용하고 있다.

이는 내가 현재 진행중인 띱이라는 프로젝트에서도 적용중인 상황이다.

그러던 도중 Unique Index에 중복 Key 컬럼에 대해 데드락이 발생하는 이슈에 대해 알게되어 테스트케이스로 정리하고자한다.

### 🤔우선 Unique 컬럼에 대해 DeadLock이 어떻게 발생하게 될까?
mySql의 공식문서를 보면 아래와 같은 내용이 기술되어있다.

> If a duplicate-key error occurs, a shared lock on the duplicate index record is set.
> This use of a shared lock can result in deadlock should there be multiple sessions trying to 
> insert the same row if another session already has an exclusive lock.

**//번역**
> 중복 키 에러가 발생 시에 shared lock(s-lock)이 중복된 인덱스 레코드에 걸립니다. 해당 s-lock은 여러 세션에서 같은 로우를 insert하려하고, 
> 다른 세션이 이미 exclusive lock(x-lock)을 가지고 있다면 데드락이 생길 수 있습니다.

예를 들어 Transaction t1,t2,t3가 존재하며 같은 데이터를 insert하려고 할때 아래의 시나리오로 데드락이 발생하는 원인을 알아볼 수 있다.

1. t1이 insert를 위해 x-lock을 획득
2. t2가 insert를 시도, duplicate error로 s-lock을 걸기 위해 대기
3. t3가 insert를 시도, duplicate error로 s-lock을 걸기 위해 대기
4. t1 커밋, t2, t3 s-lock 획득
5. t2, t3가 x-lock 걸기 위해 대기
6. 하지만 t2는 t3가, t3는 t2가 s-lock을 가지고 있어 데드락 발생

이를 해결하기 위해선 명시적으로 x-lock을 걸어 다른 트랜잭션이 끝나기 전까지 대기하도록 변경해야한다.

