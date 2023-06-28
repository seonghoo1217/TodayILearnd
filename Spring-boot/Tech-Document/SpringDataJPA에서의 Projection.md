# What is Projection?

Projection이란 DB에서 필요한 속성(Table의 원하는 컬럼)만을 조회하는 것을 의미한다.

SpringDataJPA에서 Projection을 사용하기 위해선 여러방법이 존재한다.

예시와 함께 알아보자

## 예시 주문 테이블
```java
@Entity
public class Order{
	@Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Long orderSeq;
	
	@OneToOne
	private Shop shop;
	
	private OrderType orderType;
	
	private OrderStatus orderStatus;
	
	private DeliveryStatus deliveryStatus;
	
	private ReciverInfo reciverInfo;
}
```

## NativeQuery
첫 번째로는 `Native Query`를 이용하여,원하는 데이터를 **Select**하여, 직접적으로 반환 받을 데이터를 명시적으로 작성하는 것이다.

<span style="color:gray;font-size:12px;">아래 방법의 경우 결국 Entity를 사용하기에 NativeQuery를 수행하여도 모든 컬럼에 대한 조회가 이루어짐 </span>

```java
@Query(value="select o.orderStatus, o.orderType, o.deliveryStatus from Order as o Where o.orderSeq = ?",nativeQuery=true)
Order findByNativeQuery(Long orderSeq);
```

<span style="color:gray;font-size:12px;">DTO를 사용한 NativeQuery</span>

```java
@Query(value="SELECT o.orderStatus, o.orderType, o.deliveryStatus" +
        "FROM Order as o " +
        "WHERE o.orderSeq = ?"
        ,nativeQuery=true)
OrderResponseDTO findByNativeQuery(Long orderSeq);
```
하지만, 해당 방법의 경우 **Projections**의 등장으로 인하여, 사용되지 않을 뿐더러, 협업 및 코드관리 측면에서 이점을 얻을 수 없음

> 💡 Native Query
> 
> NativeQuery의 경우 실무에서 사용할 이유도 없을 뿐더러 협업 관계에서 비효율적임
> 
> NativeQuery를 만약 직접 구현해야하는 JDBCTemplate과 같은 외부 라이브러리를 사용하는 것을 권장
> 
> 또한, 최근 queryDsl의 등장으로 복잡한 쿼리를 풀어낼 수 있다는 점이 장점으로 다가온다.
> JPA Criteria를 기반으로 도메인 주도 설계(DDD)의 명세(Specification)를 지원하고
> JPA Criteria를 기반으로 코드 구성이 되기 때문에 진짜 실무에서 절대로 쓸일이 없다

## Projection - 인터페이스 기반 Close Projection
조회를 원하는 속성들을 집합으로 인터페이스를 생성한다.

```java
// 주문의 정보, 배달정보만을 조회하고 싶을 떄
public interface ODStatusOnly{
	OrderStatus getOrderStatus();
	DeliveryStatus getDeliveryStatus();
}
```

```java
public interface OrderRepository extends JPARepository<Order,Long>{
	List<ODStatusOnly> getInfoOrderStatus(@Param("orderStatus") OrderStatus orderStatus);
}
```

기존의 Data JPA를 사용할 때 조회 반환값에 Generic을 넣어 반환시키는 것이 아닌, Custom한 `Interface`를 넣어주면된다.

**예제코드**
```java
    @Test
	public void 클라이언트에게_BODY_전달받은_경우(){
		//given
		String email="seonghoo1217@naver.com";

		//when 주어진 LocalDate 기준이상의 값 컬럼 조회
		List<MemberProjection> memberCommitInfo = memberRepository.findByEmail(email);
		memberCommitInfo.forEach(
				r-> {
					System.out.println("Name="+r.getName());
					System.out.println("Last Commit Date="+r.getCommitTime());
				}
		);
	}
```

이때 Fetch된 Result는 **Entity Object**가 아닌 JPA가 생성한 **Proxy 객체**이다.

즉, 개발자가 인터페이스에 정의를 해놓는다면 Projection을 통해 간단하게 JPA가 구현체를 만들어 전달한다.

NativeQuery를 작성할 필요성이 사라진 것이다.

## Projection - 인터페이스 기반 Open Projection

**예시 코드**
```java
//ODStatusOnly

public interface ODStatusOnly{
	@Value("#{target.orderTime+' '+target.orderUser}")
	OrderStatus getOrderStatus();
}
```

위와같이 **Open Projection** 방법을 사용할 경우 반환 객체의 필드값을 하나의 문자열(SpEL과 유사)로 리턴받는다.

하지만, 해당 방식의 경우 결국 `Entity`의 모든 컬럼 값을 조회한 후 지정한 데이터를 문자열로 뿌려주는 것이다.

즉, JPQL Select **최적화를 목표**로 하고 사용하는 **Projection의 의의**에서 이점을 얻을 수 없다.

## Projection - 클래스 기반의 Projection

**예시코드**
```java
@Getter
public class ODStatusOnly{
	private OrderStatus orderStatus;
	private DeliveryStatus deliveryStatus;
	
	public ODStatusOnly odStatusOnly(OrderStatus orderStatus,DeliveryStatus deliveryStatus){
		this.orderStatus=orderStatus;
		this.deliveryStatus=deliveryStatus;
    }
}
```

클래스 기반으로 Projection을 사용할 때 중점적으로 봐야할 것은 **`Constructor(생성자)`**이다.

클래스 기반으로 Projection을 구성할 경우 JPA는 Constructor의 Parameter 명을 기반으로 최적화 시도를 한다. 

이 때, 필드값과 다른 이름일경우 `IllegalStateException`을 throw 하게된다.(ContextLoader가 이를 찾을 수 없기 때문)

```java
public interface OrderRepository extends JPARepository<Order,Long>{
	List<ODStatusOnly> findClassProjectionByOrderStatus(@Param("orderStatus") OrderStatus orderStatus);
}
```

## 동적 Projection

```java
public interface OrderRepository extends JPARepository<Order,Long>{
	<T>List<T> findGenericProjectionByOrderStatus(@Param("orderStatus") OrderStatus orderStatus,Class <T> type);
}
```

제네릭을 사용하면, 동적으로 Projection의 데이터를 변경할 수 있다.

```java
orderRepository.findGenericProjectionByOrderStatus(orderStatus,OSOnlyDTO.class );
```

## 중첩 구조를 이용한 Projection 처리

```java
public interface exampleProjection{
	Long deliverySeq();
	OrderResponse getOrderStatus();
	
	interface OrderResponse{
		Integer deliveryTime;
    }
}
```

인터페이스 내부에 하나의 인터페이스를 추가적으로 생성하여 가져올 필드값과 객체를 명시적으로 선언하는 방법이다.

해당 방법의 경우, 명시적으로 선언을 하기에 유지보수 관점에서 이점을 얻을 순 있으나, 확장에 폐쇄적인 구조이다.

또한, 쿼리 실행결과 또한 **`Delivery`** 도메인에선 개발자의 의도대로, Sequence값만 추출되지만, Order의 경우 모든 필드값을 조회한다.
그리고 중첩 구조의 가장 큰 단점으로는, 첫 메서드 즉 **deliverSeq()** 까지만 쿼리 최적화가 동작한다.

## 결론
- Projection 대상이 Entity이고, 모든 컬럼을 조회할 필요가 없을 경우 JPQL 최적화를 목적으로 사용한다면 성능개선 측면과 응답성 측면에서 이점을 얻을 수있다.
  - 단, Projection 대상이 root 엔티티가 아닐경우, SELECT 쿼리 최적화가 불가하다
    - LEFT OUTER JOIN이 발생한다.
    - 모든 필드를 SELECT 한 후 후처리 작업이 이루어지기에 사실상 **최적화라고 볼 수 없음**

이번 Data JPA를 활용한 Projection 코드를 작성하며 queryDsl의 중요성을 리마인드 할 수 있는 계기가 된것같다.

다음 포스팅은 queryDsl에서의 Projection에 대해 다뤄보고자한다.