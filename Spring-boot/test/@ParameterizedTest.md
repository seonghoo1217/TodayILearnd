## Parameterized의 사용목적
- 테스트 코드에서 반복되는 구문을 최소화하기 위해
- 반복되는 구문 가독성을 저하시키고, 테스트 속도를 저하시킨다.

### 예시
```java
public class Shop {
    private String name;
    
    public Shop(String name) {
        validateName(name);
        this.name = name;
    }
    
    public String getName() {
        return name;
    }
    
    private void validateName(String name) {
        validateNotNull(name);
        validateLength(name);
    }
    
    private void validateNotNull(String name) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("이름은 공백일 수 없습니다.");
        }
    }
    
    private void validateLength(String name) {
        if (name.length > 10) {
            throw new IllegalArgumentException("이름은 10자 이하여야 합니다.");
        }
    }
}
```
다음과 같은 **Shop Domain** Class가 존재한다고 가정하여보자.

Shop 클래스는 코드에서 보이는 것 처럼 가게이름에 공백을 허용하지않고, 10자이하의 이름만을 수용하고, 아니라면 **IllegalArgumentException()**을 발생시킨다.

이를 Junit에서 테스트하려고한다면, 아래와 같이 총 3가지의 테스트 케이스가 생성이 될것이다.

```java
@Test
@DisplayName("이름으로 null 값 예외처리")
void createShopFromNullName() {
        assertThatThrownBy(() -> new Shop(null))
        .isInstanceOf(IllegalArgumentException.class);
        }

@Test
@DisplayName("공백 이름 예외처리")
void createShopFromBlankName() {
        assertThatThrownBy(() -> new Shop(""))
        .isInstanceOf(IllegalArgumentException.class);
        }

@Test
@DisplayName("10자를 초과하는 이름 예외처리")
void createShopNameOver10Length() {
        assertThatThrownBy(() -> new Shop("This Name is Over 10"))
        .isInstanceOf(IllegalArgumentException.class);
        }
```

위의 테스트코드 모두 각각의 메서드로 구성되어있지만, 결국 하나의 값을 검증하기 위해 분리되어 있어 불필요 코드가 증가된 꼴이 된다.

이러한 테스트코드의 증가는 Shop 클래스의 유효성 검사항목이 증가함에 따라 비례적으로 증가한다.

그렇기에 우리는 `@ParameterizedTest`를 이용하게된다.

## @ParameterizedTest
- Junit에 여러개의 테스트를 동시 테스트하기위한 목적으로 설계된 어노테이션
- 기본적으로 `@Test` 어노테이션 대신 사용된다.
- 이 때 파라미터로 넘겨줄 값들을 지정해주어야 하는데, 이 또한 어노테이션을 사용해서 테스트에 주입해줄 수 있다.

### @ValueSource
```java
@ParameterizedTest
@DisplayName("ValueSource, Shop Class 생성")
@ValueSource(strings = {"", " "})
void createUserException(String name) {
    assertThatThrownBy(() -> new Shop(name))
            .isInstanceOf(IllegalArgumentException.class);
}
```
- 테스트에 주입할 값을 배열형태 지정하여, 순차적으로 배열을 순회하며 테스트 메소드에 인수에 값을 넣어 테스트한다.
- 보통 기본 자료형 넣어, 예시와 같이 문자열을 테스트하는 용도로 많이 사용된다.

### @CsvSource
- `@CsvSource`의 경우에는 같은 메서드를 이용하지만 검증 값이 다를 때 사용한다.
- 위의 예제 같이 같은 값으로 여러 값을 검증할때 사용할 수 있다.

```java
void multiplyNumBy2(int num){
    return num*2;
}
```

```java
@ParameterzedTest
@CsvSource(value={"1:2","2:4","3:6"})
void multiplyBy2Test(int input,int excepted){
    assertThat(multiplyNumBy2(input)).isEqualTo(excepted);
}
```

### @NullSource, @EmptySource, @NullAndEmptySource
- @NullSource : TestCase의 인자값에 **null값**을 대입
- @EmptySource : TestCase의 인자값에 **빈값**을 대입
- @NullAndEmptySource : **null값과 빈값을 모두** 대입하여, 예외 검증
*************
- 해당 어노테이션들은 기본적으로 @ValueSource와 함께 사용하여 다른 값과 함께 검증이 가능하다.
- 테스트하고자하는 메서드의 인자값 boolean,byte 형태라면 @NullSource와,@NullAndEmptySource가 사용불가(인자값에 들어가지 않는다.)

```java
//위 예제 도메인 Shop 클래스의 이름 검증 테스트 케이스
@ParameterizedTest
@DisplayName("null, 공백, empty, 10글자이상 값으로 Shop 생성")
@NullAndEmptySource
@ValueSource(strings = {" ","This Message is Over 10 Length"})
void createShopExceptionNameValidate(String name) {
    assertThatThrownBy(() -> new Shop(name))
            .isInstanceOf(IllegalArgumentException.class);
}
```

### @EnumSource
```java
@ParameterizedTest
@DisplayName("6, 7월이 31일까지 있는지 테스트")
@EnumSource(value = Month.class, names = {"JUNE", "JULY"})
void isJuneAndJuly31(Month month) {
    assertThat(month.minLength()).isEqualTo(31);
}
```

- Enum타입을 검사하는데 있어 사용되는 어노테이션으로 **JPA에서 Embedded 값**을 검사하는데 유용하다.
- @EnumSource의 세번째 인자값에는 mode 값을 지정해주어야하며 **default는 INCLUDE**로 설정되어있다.
  - **mode**
    - **INCLUDE**: names.contains(name) - (name과 일치하는 모든 Enum 값) // default
    - **EXCLUDE**: !names.contains(name) - (name을 제외한 모든 Enum 값)
    - **MATCH_ANY**: patterns.stream().anyMatch(name::matches) - (조건을 하나라도 만족하는 Enum 값)
    - **MATCH_ALL**:patterns.stream().allMatch(name::matches) - (조건을 모두 만족하는 Enum 값)


### @MehodSource
- 이전의 어노테이션은 값을 입력받아 해당 값에 대한 검증을 수행하는 용도이지만, 해당어노테이션을 사용하면 메서드 자체를 검증할 수 있다.

```java
@ParameterizedTest
@MethodSource("provideTestData")
void testSomething(int input1, String input2) {
    // Your test logic here
}

static Stream<Arguments> provideTestData() {
    return Stream.of(
        Arguments.of(1, "One"),
        Arguments.of(2, "Two"),
        // Additional test data
    );
}

```
- @MethodSource에 작성하는 **메소드 이름은 인수로 제공하려는 메소드 이름**과 같아야 한다.
- 인수로 제공하려는 메소드는 **static**이어야 한다.
