# 엔티티 생명주기와 변경감지 @Transactional 이슈

최근 프로젝트를 진행하는 과정에서 겪었던 사소한 이슈인데 JPA의 기본이기도 하면서 절대 실수를 범하면 안되는 부분이기에 소개하려한다.

## 엔티티의 생명주기
우선 JPA에서는 엔티티의 생명주기를 크게 4가지로 관리한다.

- 비영속(new/transient)
- 영속(managed)
- 준영속(detached)
- 삭제(remove)

### **비영속**

비영속 상태는 객체를 새로 생성하였을 때 시점으로 영속성 컨텍스트에 의해 관리되지 않는 상태를 뜻한다.
JPA와는 관계없이 객체만 생성한 상태를 말한다.

```java
Timer timer=new Timer(1L,"11:06:48");
```

영속상태가 아니므로 1차캐시에 포함되지 않으며, 변경감지등의 기능이 적용되지 않는다.

### **영속**

영속성 컨테스트에 의해 관리되는 상태.

생성한 객체를 **EntityManager의 persist()** 메소드를 통해 **영속성 컨테스트에 저장**하거나,

혹은 **find()** 메소드를 통해 **DB에서 엔티티를 조회**하면 영속성 컨텍스트에 의해 저장되어 관리된다.

```java
Timer timer=new Timer(1L,"11:06:48"); // 해당 시점에서 비영속 상태이다.
EntityManager em;

em.getTransaction().begin(); //엔티티 매니저에서 수행하는 모든 로직은 트랜잭션 안에서 수행되어야 함

em.persist(timer); // 해당시점에 영속상태가 되어 영속성 컨텍스트에 의해 관리됨

em.getTransaction.commit();
```


### **준영속**

영속성 컨텍스트에 의해 관리되었다가 현재는 분리되어 영속상태가 아닌 상태를 뜻한다.

따라서 영속성 컨텍스트가 제공하는 기능을 사용하지는 못한다.

```java
Timer timer=new Timer(1L,"11:06:48"); // 비영속
EntityManager em;

em.getTransaction().begin(); 

em.persist(timer); // 영속
        
em.detach(timer); //timer 엔티티를 준영속으로 변환
        
em.getTransaction.commit(); 

```

### 준영속과 비영속의 차이

이 둘의 가장 큰 차이는 영속성 컨텍스트에 의해 관리되었는지에 대한 관리 유무이다.

영속성 컨텍스트에 의해 관리되기 위해선 엔티티로서 고유 식별자가 필요하고 이에 따라 준영속 상태의 엔티티의 경우 식별자가 있다는 것을 보장하지만,
비영속 상태의 엔티티의 경우 식별자의 존재 유무가 확실치 않다.

비영속 상태에서는 지연 로딩이나 캐싱 기능을 이용할 수 없으며, 관계형 매핑 매커니즘의 이점을 활용할 수 없다 

따라서, 관련된 데이터를 조회하기 위해 매번 데이터베이스와의 통신이 필요하다.

반면에, 준영속 상태에서는 영속성 컨텍스트의 기능을 일부 이용할 수 있다.

예를 들어, 1차 캐시에 저장된 엔티티를 다시 조회하거나, 지연 로딩을 사용하는 연관 엔티티를 초기화하는 등의 작업이 가능하다.

### 삭제

1차 캐시와 데이터베이스에서 모두 삭제된 상태를 뜻한다.

```java
em.remove(entity);
```

### 생명주기 흐름도

![EM](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc6RPuA%2FbtrnE98qQSG%2FKkM5X199j8iAtW5TLNflA0%2Fimg.png)


## 그럼 이게 변경감지와 무슨 상관이 있을까?

Entity의 필드 값을 수정시킬 때는 보통 준영속 상태 시점에서 사용된다.

하지만 내 코드는 멤버를 Spring Data JPA로 조회 후 그 조회한 Member Entity를 바로 변경감지를 통해 필드값을 수정한다.

**그래서 혹시 조회 후에 바로 준영속상태로 내려가는걸까?** 라는 의문이 생겨났다.

만약 그렇다면 준영속상태에서는 DB에 접근하여 값을 바꾸는 것이 불가능하니까

**하지만...**

우리의 ChatGPT 선생님께서는 계속 영속상태를 유지한다고 말하셨다.

![image](https://user-images.githubusercontent.com/39437170/221391983-28bac164-f082-4972-b96b-619ede463c1c.png)

### 그렇다면 발생원인은..?

자, 우리는 이쯤에서 내가 앞서 말한 엔티티의 생명주기에서 엔티티의 모든 작업은 트랜잭션 내부에서 동작해야한다고 했다.

엔티티의 필드값을 변경하기 위해선 크게 5가지 절차가 있다.

1. 트랜잭션시작
2. 변경하고자 하는 필드값이 포함된 엔티티의 식별자를 통한 where 조회
3. 조회한 데이터에 수정할 내용들로 교체
4. update 처리
5. 트랜잭션 처리

필자의 경우 Member 정보에 대한 update 처리를 **Service Layer**에서 진행하였고
해당 Service Layer에 **`@Transactional`** 처리가 되어있지 않아 작업이 원활히 수행되지 못했다.

좀 더, 자세히 말하자면 JPA에서의 변경감지는 총 4단계를 거친다.

1. 트랜잭션이 시작된다.
2. 조회하고자하는 Entity를 영속화 시킨다.
3. Entity의 정보를 수정
4. 이후 트랜잭션이 커밋되어 작업을 완료한다.

위에서 볼 수 있듯 변경감지는 명시적으로 update 쿼리를 작성하지 않는다.

## 변경감지 자세히 알아보자

변경감지는 트랜잭션이 시작되어 정보를 조회한 시점에 스냅샷을 생성한다. 

쉽게 생각하면 원본 정도로 얘기할 수 있다.
그리고 트랜잭션이 커밋되면 바뀐 부분을 찾아 update 해주는 것이 변경감지이다.

그럼 이를 자세히 뜯어보자

### Transaction Commit 시점

JPA는 트랜잭션이 커밋될때 EM(EntityManger)에서 자동으로 flush를 호출해준다.

트랜잭션 커밋시 개발자가 직접 DB에 변경내용을 update하지 않는다면 변경이 되지 않기 때문에 JPA에서 이를 자동으로 수행해준다.

> 💡 **flush** : 영속성 컨텍스트의 변경사항을 DB에 동기화 시켜 반영하는 것

### 스냅샷과 비교

flush를 호출하게 되면 스냅샷과 Entity의 바뀐 정보를 서로 비교한다.

스냅샷은 DB에서 데이터를 가져와 영속성 컨텍스트에 저장한 후 Entity를 영속화하는 시점의 최초정보들이다.

### update 처리

스냅샷 정보와 변경된 Entity 정보를 비교해 바뀐 부분을 기준으로 update 쿼리를 작성한다. 
이는 JPA가 상태 변화를 감지하고 자동으로 내부에서 update 쿼리를 발생시키므로 개발자가 명시적으로 update 쿼리를 적을 필요가 사라진다.

update 쿼리는 JPA EM 내부에 존재하는 쓰기 지연 저장소에 저장해뒀다가 트랜잭션이 커밋 되는 시점에 DB로 요청을 보내 상태 변화를 일으킨다.

이때 쓰기 지연 저장소를 사용하는 이유는 여러 개의 엔티티의 변경 내용을 한 번에 처리하여 성능을 개선하고, 
트랜잭션 롤백이 발생할 경우 쓰기 지연 저장소에 저장된 변경 내용도 롤백될 수 있도록 하기 위함이다.

Mysql의 undo log와 비슷한 개념이라고 생각하면 편하다.

### 변경감지 조건

위에서 말한 것처럼 변경감지가 발생하려면 아래의 조건을 충족해야한다.

1. 우선 Entity가 영속화되어 있어야한다.
2. 작업이 Transaction에 포함되어 있어야한다.
3. 트랜잭션이 제대로 반영되어 flush가 호출되어야한다.

### 사용 예시

```java
@Transactional(readOnly=true)
@Service
@RequiredArgsConstrutor
public class MemberService {
	
	private final MemberRepository memberRepository;
	
	public void userNicknameUpdate(String nickname){
		memberRepository.findByNickname(nickname).get().updateNickname(nickname);
    }
}
```

```java
@Entity
public class Member{
    ...
    ...
	...
	...
    
    public static void updateNickname(String nickname){
    	this.nickname=nickname;
    }
}
```

만약 위의 예시코드에서 Service Layer의 @Transactional 어노테이션이 빠지게되면 정상적인 변경감지가 이루어지지 못한다.


## 결론

정말 흔하고 편하게 써왔던 변경감지이고, 공부를 갓 시작했을 때나 이론적으로 접했기 때문에

늘 코드로 작성하면서도 이런 내부적인 작동원리를 모르고 썼다는것이 부끄러웠다. 무엇이든 동작원리를 깨닫고 사용해야 효율성이 극대화 된다는걸 다시 깨닫게 되는 경험이었다.