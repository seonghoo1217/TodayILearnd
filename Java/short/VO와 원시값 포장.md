**둘의 가장 큰 차이는 캡슐화**

## 원시값 포장
- 원시타입(primitive type)을 객체로 Wrapping하여야 할때 사용한다.
- Wrapper Class가 제공하는 메서드를 사용하거나, 원시 자료형을 Collection(map,list ,,,)으로 다루고 싶은 경우 등에 사용
- null 가능성을 제공하고, 일반적으로 불변성을 보장

## VO(Value Object)
- VO는 비즈니스 로직에서 사용되는 서로 연관있는 값을 캡슐화하기 위해 사용
- Java에서 제공하는 String은 이에 해당
- 동등성이 보장되지않음, 불변성은 보장

### VO 동등성 보장 못하는 Case

```java
Person p1 = new Person("John");
Person p2 = new Person("John");

// 동등성 (false)
System.out.println(p1 == p2);

// 동등성 (true)
System.out.println(p1.equals(p2));
```
- 동일성은 깨지게 되어 VO는 `equals()`와 `hashCode()`를 적절히 재정의하여 동등성을 보장해야함.

## 결론
- VO와 원시값 포장을 사용하는 이유는 값의 생성, 유효성 검사 및 관련 로직을 객체 내부로 캡슐화하여 책임을 하나의 객체에 모을 수 있기 때문
