implements와 extends 모두 자바의 **상속 개념**에서 사용되는 키워드이다.

두 메서드 모두 부모 객체 상속하여 정의한다는 점에서 공통점을 갖는다. 차이점이 생기는 부분은 **부모 객체를 그대로 사용하는지에 대한 유무**이다.

`extends`의 경우 부모에서 선언 / 정의를 모두하며 자식은 메소드 / 변수를 그대로 사용하여 구현한다.

`impliments`는 부모를 선언만하며, 정의(내용)은 자식에서 오버라이딩 (재정의) 해서 사용한다.

별도로 `abstract` extends와 interface 혼합. extends하되 몇 개는 추상 메소드로 구현한다.

## extends
- 단일 상속의 대표적인 형태
- 부모의 메소드를 그대로 사용할 수 있으며 오버라이딩 할 필요 없이 부모에 구현되있는 것을 직접 사용가능

## implements
- implements의 가장 큰 특징은 이렇게 부모의 메소드를 반드시 오버라이딩(재정의)해야한다.
- 또한, 다중 상속을 지원한다.

## 종합정리
- extends는 일반 클래스와 abstract 클래스 상속에 사용되고, implement는 interface 상속에 사용된다.
- class가 class를 상속받을 땐 **extends**를 사용하고, interface가 interface를 상속 받을 땐 **implements**를 사용한다.
- class가 interface를 사용할 땐 implements를 써야하고
- interface가 class를 사용할 땐 implements를 쓸수 없다.
- extends는 클래스 한 개만 상속 받을 수 있다.
- extends 자신 클래스는 부모 클래스의 기능을 사용한다.
- implements는 여러개 사용 가능하다.
- implements는 설계 목적으로 구현 가능하다.
- implements한 클래스는 implements의 내용을 다 사용해야 한다
- extends는 클래스를 확장하는 거고 implements는 인터페이스를 구현하는 것을 목적