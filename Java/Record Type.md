## Record
- Java 진영에서 JDK 16부터 정식스펙으로 새롭게 추가 클래스 선언 방법이다.
- 보일러 플레이트 코드(Bolierplate Code)를 줄이는 것을 목표로 도입되었다

> 💡 Bolierplate Code
> - 코드의 재사용성이 높고 변경이 적은 코드
> - Java 진영에서는 getter, setter, equals, hashCode, toString 이나 접근제어자가 이에 해당
> - 프로젝트의 진행을 위해선 반드시 필요하지만 반복적으로 작성되어야 하기 때문에 개발 효율성을 저하시키는 요소로 판단된다.
> - 물론, lombok이나 IDE의 도움을 받아 코드를 간결하게 만들 수 있지만 이는 언어 자체의 해결방법이라기 보다는 라이브러리의 이용을 통해 해결된 부분

### Record의 핵심 목표
- 객체 지향의 사상에 맞게 데이터를 간결하게 표현하기 위한 방법을 제공한다.
- 개발자가 동작을 확장하는 것보다 불변 데이터를 모델링하는데 집중하도록 함
- 데이터 지향 메소드를 자동으로 구현

## Record 사용
```java
public record Ticket(UUID uuid, Event event) { 
    
}
```
- Record 타입의 경우 필드를 클래스 내부에 작성하는 것이 아닌 선언과 동시에 나열한다.
- 이러한 필드를 컴포넌트라고 하며 기본적으로 `레코드명(헤더), {바디}`의 구조를 가지게됨

## Record의 특성
- 상속 불가능 (final class)
    - 암시적으로 final로 선언되어, abstract로 선언이 불가능하다. (추상화 불가)
- 필드 private final 및 Setter 지양을 통한 불변성 보장
    - record 내 각 필드(헤더에 나열한 컴포넌트)는 private final로 정의
    - 다른 클래스를 상속(extends)는 불가하지만, 인터페이스로는 구현(implements)이 가능
- 레코드 내부에 멤버 변수(인스턴스 필드)를 선언할 수 없다. (static 변수는 생성이 가능)

## Record - class
Record 타입의 경우 기본적으로 Intellij에서 생성하게 된다면 class로 변환하여주는 리팩터링 기능을 제공한다.

이때 Class형태로 변경하여 준다면 Record 특성 그대로 구현하여준다.

```java
public final class Ticket {
    private final UUID uuid;
    private final Event event;

    public Ticket(UUID uuid, Event event) {
        this.uuid = uuid;
        this.event = event;
    }

    public UUID uuid() {
        return uuid;
    }

    public Event event() {
        return event;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj == this) return true;
        if (obj == null || obj.getClass() != this.getClass()) return false;
        var that = (Ticket) obj;
        return Objects.equals(this.uuid, that.event) &&
                this.uuid == that.uuid;
    }

    @Override
    public int hashCode() {
        return Objects.hash(uuid, event);
    }

    @Override
    public String toString() {
        return "Ticket[" +
                "uuid=" + uuid + ", " +
                "event=" + event + ']';
    }
}
```


### 일반 클래스와 equals 구현시 차이점
일반 클래스는 기본적으로 객체의 주소값을 기준으로 equals() 가 구현되어 있는 반면, Record는 이를 Override하여 필드의 값을 모두 비교하도록 구현되어 있다.

그렇기에 기본적으로 `new` 키워드를 통해 생성된 객체를 equals()로 비교할 경우 **false**가 반환되지만, record의 경우 **true**가 반환된다.

### Getter의 차이점
기본적으로 스프링부트 진영에서 Getter를 Lombok 라이브러리를 통해 사용하는데 이경우 `get`이 Prefix로 붙고 그 뒤에 필드명이 붙게된다.
하지만 Record 타입의 경우 필드명을 메서드로 두어 호출한다.

```java
Ticket ticket = new Ticket(UUID.randomUuid(), event);
System.out.println(ticket.uuid());
System.out.println(record.event());
```

### 컴팩트 생성자 (Compact Constructor)
Record의 표준 생성자를 호출하면 초기화 작업처럼 실행되는 코드를 작성할 수 있다.
validation을 위한 용도로 사용할 수 있다.

```java
public record TestRecord(UUID uuid, Event event) {
    public TestRecord {
        if (event == null) {
            throw new IllegalArgumentException("Event given is null");
        }
    }
}
```

### 리플렉션으로 조작 불가능
Java 17 부터는 언어 차원에서 Reflection을 사용한 필드 조작을 방지하고 있다.
이는 Record의 불변성을 더욱 강화시켜주는 특징이다.

Record를 Reflection을 통해 값을 조작하려고할 경우 `llegalAccessException`가 발생한다.

### Collection Framework 사용시 주의점
```java
public record Parent(String name, int age, List<String> children) {}
```
List는 참조한후 `add` 또는 `remove` 메서드를 통해 값의 일관성을 보장하지 못한다.

그렇기에 `Collection.unmodifiableList`를 이용하여 불변성을 보장해주어야한다.

## Spring에서 Record 사용하기

**Entity**
우선 Entity의 사용에 대해 고려하기위해 Record 타입을 공부하였기에 Entity에 대한 얘기가 빠질 수는 없을것이다.

Record 타입으로 변경한 후 Entity를 쓰면 `User declared non-static fields id are not permitted in a record` 에러가 발생한다.

엔티티를 Record 타입으로 사용할 수 없는 이유는 몇가지가 있지만 아래와 같은 이유가 가장 적합합니다.

1. jpa는 프록시 생성을 위해 인수 생성자, non-final 필드, setter 및 non-final 클래스가 없는 엔티티에 의존한다.
즉, 프록시를 생성하기 위해서 entity는 불변이면 안됩니다.(jpa의 프록시는 일대일 매핑 시 지연 로딩 제공 등 다양하게 쓰입니다.)
2. 쿼리 결과를 매핑할 때 객체를 인스턴스화 할 수 있도록 매개변수가 없는 생성자가 필요
3. 접근자 메소드인 getter가 필수 명명 규칙을 따르지 않습니다.

**Req/Res DTO**
- API의 경우 요구사항의 변화에 따라 추가해주면 되기에 DTO단에서 사용할 수 있다.

**Config Class**
- DTO와 마찬가지로 설정값의 추가를 불변하게 만들어 사용하기에 유용하다.

### REF
- https://velog.io/@power0080/java%EC%9E%90%EB%B0%94-record%EB%A5%BC-entity%EB%A1%9C
- https://velog.io/@gongmeda/Java-Record-%ED%86%BA%EC%95%84%EB%B3%B4%EA%B8%B0-Spring-%EC%97%90%EC%84%9C%EC%9D%98-Record-%EC%82%AC%EC%9A%A9-%EC%82%AC%EB%A1%80%EC%99%80-%ED%95%A8%EA%BB%98