# J2EE 디자인패턴?
- Sun Microsystems에서 만들어진 반복되는 설계 문제를 해결하기 위한 모범 사례 모음이다.
  - 패턴은 다양한 문제에 적용할 수 있고, J2EE 개발자들의 성공적인 경험을 활용할 수 있는 솔루션
- 웹 기반의 엔터프라이즈 어플리케이션을  개발하기 위해 만들어졌다.
- J2EE: Java 2 Platform Enterprise Edition, JAVA SE를 확장한 엔터프라이즈 사양

## J2EE 디자인 패턴 종류
대표적인 3가지 레어어로 `Presentation`,`Business`,`Integration` Layer로 나누어진다.

## Presentation Layer Design Pattern

### Intercepting Filter Pattern
- 사용자 요청의 전처리와 후처리를 쉽게 처리

### View Helper Pattern
- Helper 컴포넌트의 Presentation formatting에 관계되지 않은 로직
- 즉, 화면과 Business Delegate에 접근하는 Helper를 분리한다.

> 💡 View Helper 
> 패턴은 사용자 인터페이스(UI)와 관련된 로직을 캡슐화하고 재사용 가능한 도우미 함수나 메서드로 제공하는 방법을 구성한다.
> 
> 이로써 개발자는 뷰(View)에서 로직을 분리하여 코드를 관리하는 것에서 이점을 얻음.
> 
> View Helper 컴포넌트는 View Helper 패턴의 일부로서, 사용자 인터페이스를 생성하고 조작하는 데 필요한 도우미 함수나 메서드를 포함하는 컴포넌트로서
> 주로 특정 뷰(View)나 템플릿에서 사용되며, 이러한 도우미 함수를 통해 뷰의 표현과 동작을 조정한다.


### Composite View Pattern
- 독립적인 View를 종합하여 새로운 View를 생성한다.

> Composite View
> 
> 소프트웨어 개발에서 사용되는 디자인패턴으로, 해당 패턴은 UI를 구성하는 요소들을 계층 구조로 관리한다.
> 
> Composite View 패턴은 주로 MVC 패턴에서 사용된다.
> 
> 일반적으로 MVC 패턴에서는 뷰(View) 역할이 UI를 표현하고 사용자 입력을 처리하는 역할을 수행한다.
> 
> 이때 Composite View 패턴을 적용하면 뷰가 여러 개의 작은 요소로 구성되는 경우에도 일관된 방식으로 관리할 수 있다.
> 
> Composite View 패턴에서는 다음과 같은 주요 구성 요소들이 사용됩니다:

**Composite View 구성요소**

- Component: 이는 계층 구조에서의 모든 요소의 공통 인터페이스를 정의한다. 이 인터페이스는 요소들의 추가, 제거, 탐색 등을 수행할 수 있는 메서드를 포함.

- Leaf: 이는 계층 구조에서의 최하위 요소로, 더 이상 하위 요소를 가지지 않는다. 주로 기본 UI 요소들을 나타낸다.

- Composite: 이는 계층 구조에서의 중간 또는 최상위 요소로, 다른 요소들을 하위로 가질 수 있다. 이를 통해 계층 구조를 구성하고 여러 개의 작은 요소들을 하나의 큰 UI 요소로 조합할 수 있다.

즉, Composite View 패턴은 UI를 계층 구조로 표현함으로써 UI의 복잡성을 다루기 쉽게 만들어준다.

### Front Controller Pattern
- 사용자 요청제어를 한곳에서 관리하기 위한 컨트롤러를 제공한다.

## Business Layer Desing Pattern

### Business Delegate Pattern
- presentation 계층과 Business 계층간의 분리를 통해 결합도를 낮추는 방법

### Session Facade Pattern
- 클라이언트와 비즈니스 로직 사이에 중간 계층을 제공하여 비즈니스 로직의 복잡성을 감소시키고 클라이언트와의 상호작용을 단순화
- 클라이언트는 분산된 비즈니스 컴포넌트들과 직접 상호작용하지 않고, 중간 계층에 위치한 Session Facade를 통해 비즈니스 로직을 호출

### Transfer Object Pattern
- 클라이언트에서 서버로 한 번에 여러 속성을 가진 데이터를 전달 할 때 사용한다.
- 한 객체(Class)안에 `filed` 값을 여러개 묶어 전달한다.
- 데이터 송수신을 위한 **round trip** 숫자를 줄여 성능적으로 이점이 크다.
- 기본적으로 DTO(Data Transfer Object)을 사용한다.

![] (https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb7IPba%2FbtrmQ3n86G4%2FcbTx7KKKu9PD5aqDUuYRd1%2Fimg.jpg)

- VO(Value Object)와는 별도의 개념으로 분리된다.