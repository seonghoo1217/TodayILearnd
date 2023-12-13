# JPARepository vs Repository vs CRUDRepository

> 최근 팀원과 협업 프로젝트를 진행하는 과정에서 Repository Layer를 구성할 때 일반적으로 사용하는 JPARepository 외에 `Repository`라는 인터페이스를 확장 사용하여 구현하였기에 궁금하여 이에 대해 조사 후 포스팅하게되었다.

## JPARepository
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fdv23uQ%2FbtrHxVCLZgU%2FuhBdS1RKIGv4A5nChFagWK%2Fimg.png)

위의 그림은 Spring Data JPA 진영에서 제공하는 인터페이스의 상속관계를 도식화한것으로 아래로 갈 수록 저수준 모듈이며 기능 구현이 많음을 알 수 있다.

JPARepository의 최대 장점은 인터페이스에 별다른 정의 없이 다양한 메서드를 사용할 수 있다는 것이 장점이다.

하지만 이 또한 JPARepository의 최대 단점으로 다가온다.

이는 객체 지향의 관점과도 연관되는데 불필요한 메시지 노출된다는 점이다.

### 불필요한 메시지의 노출 (단점)
객체지향의 관점에서 메시지는 객체가 서로 상호작용하는 수단이다.

어떤 객체가 어떠한 메시지를 받을 수 있는지에 대한 정의이기도하며 책임이자 역할이다.

어디까지나 객체지향 관점에서의 이야기이지만, 실제로도 정의하지 않은 메서드를 전부 제공받기에 정말 필요한 메서드만 선언해서 사용하면 메서드의 재사용성이 높아진다.

**JPARepository를 확장 구현한 Case**
![](https://github.com/ddip-team/ddip-be/assets/39437170/ec34db6d-1508-454b-8f1b-65bcc6dd04c0)
- 실질적으로 delete나 exists등의 메서드를 구현하지 않았음에도 불구하고 수 많은 메서드들이 노출된다.

## Repository
Repository를 상속하게 되면 기본으로 구현되는 메서드가 전혀 없다. 직접 선언한 메서드에 대해서만 외부에 개방된다.

이는 Application 계층에서 Repository(Domain 계층)에 어떤 메시지를 보낼 수 있는 지 한눈에 파악할 수 있게 된다.

Repository를 확장 구현하면, JPARepository와 반대로 선언한 메서드만 외부로 개방한다.

### 명세적인 측면
명세적인 측면이란건 조건을 충족한다면, 선언한 타입이라고 인정을 해준다거나, 내부 구현은 자유롭게 하되 조건에 대해서는 필수적으로 구현해야하는 제약해야하는 명세이자 규약으로써 역할을 수행한다

**JPARepository를 상속 구현한 경우**
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcnmsBi%2FbtrGzPcYWGs%2FtyBOy7YKcjg7zXLJQRGdvK%2Fimg.png)
- JpaRepository를 상속할 경우, 총 4개의 인터페이스를 상속하는 결과가 만들어지고, 이로 인해 수많은 명세들이 함께 포함된다.
- 그러나 이들은 모두 구현 Repository Layer에 명시되어 사용되지는 않는다. 구현체를 만들 때 명시되지 않았더라도 모든 메서드를 구현해야 한다는 점은 여전히 문제
- 하지만, Repository를 상속할 경우, 완전히 독립적인 명세로서 역할을 할 수 있게 됨
- Repository를 상속구현한 인터페이스의 경우 인터페이스에 선언되어 있는 메서드들만이 명세의 전부입니다.
따라서 해당 구현체의 메서드들이 모두 인터페이스의 책임이고 개방해야 할 메시지들이며, 이를 구현하려는 구현체들이 어떤 것을 구현해야 하는지 명세로서 온전한 역할을 할 수 있게 된다.


### 테스트 더블 용이성
> 💡Test Double?
> 실제 객체를 대신하여 테스팅에 사용되는 모든 방법을 호칭하는 용어이다.
> 
> 대표적으로 Dummy, Fake, Stub, Spy, Mock 등이 존재한다.

JPARepository를 상속받는 경우 테스트 더블을 위 대역을 만들기 위한 비용에서 Repository 상속과 차이가 크게 발생한다.

가장 핵심은 **불필요한 코드라인의 증가**와 테스트에 사용되지 않는 메서드의 외부 노출이 발생한다는 점에서 `Repository` 상속이 가지는 의미가 크다.

## JPARepository 상속 계층도
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdDpLva%2FbtrGvTNYP0o%2FMU553oBcEQ6oXOKDBLPGJ1%2Fimg.png)

JpaRepository는 CrudRepository, PagingAndSortingRepository 등을 상속한다.
CrudRepository에는 CRUD 관련 메서드들이 선언되어 있고, PagingAndSortingRepository에는 Sort 객체를 매개변수로 받는 페이징 처리 메서드가 선언되어 있고,
JpaRepository에는 추가적인 편의 메서드들이 많이 선언되어 있어 개발환경에 편의성을 높여준 것은 사실이다.

하지만, 필요 기능만을 구현하여 사용하는 것이 객체지향적이나 구조 상의 이점이 있다는 것이 자명한 사실이다.


## 결론
- 객체의 책임과 메시지를 명확히
- 필요한 메서드만 선언했기에 테스트 더블 생성 시 비용이 절감됨
- JpaRepository 보다는 Repository를 상속해서 사용