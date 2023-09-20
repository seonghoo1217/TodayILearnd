Java에서 예외를 다루는법은 try/catch 이외에는 잘 알고있지 못했다. 
하지만, 테스트코드에서 예외를 다루기위해 try/catch를 쓴다는 것은 정말이지 부적합 문법이라고 생각을 했고, 이를 역시 Junit측에서도 인지를 하였기에 별도의 예외처리를 위한 메서드를 구현해두었다.

## Junit에서 Assertions를 통한 예외를 다루는법
*************

우선적으로 예외를 다루기 위해서 예외를 발생시키 메서드를 하나 생성하자.

```java
public class OnlyUseException(){
    public static void error() {
        throw new RuntimeException("Something Error!");
    }
}
```

이를 실제 테스트코드에 적용시킨다면, 마냥 에러를 발생시키는 것이 아닌 **테스트 조건(condition)에 부합하지 않을경우 에러를 발생**시키면 될것이다.

다음으로는 `Assertions`의 메서드를 이용하여 해당코드가 **에러를 발생시키는지 검증**하면된다.

## 방법 1.assertThrows
*************
```java
@Test
public void junit5_assertThrows() {
    Assertions.assertThrows(RuntimeException.class, () -> {
        OnlyUseException.error();
    });
}
```
첫 번째 인자값에 예외타입을 넣어, 해당 예외가 발생하는지 검증한다.

예를 들어, 문자열의 길이를 검증하는 메서드에서 길이가 5이상일때 `IllegalArgumentException()`를 발생시키는 메서드가 있다면, 해당 문자열의 길이가 4이하일 때는
테스트코드가 정상동작하지 않지만, 반대의 경우 테스트 코드가 **pass 상태**가 된다.

## 방법 2.assertj의 assertThatThrownBy
*************
````java
@Test
public void junit5_assertThatThrownBy() {
    assertThatThrownBy(() -> OnlyUseException.error())
            .isInstanceOf(RuntimeException.class)
                .hasMessageContaining("Error!");
}
````

첫번째 인자값의 에러를 검증할 메서드를 넣고, isInstanceOf에 발생하는 예외타입을 넣는 방법으로 개인적으로 assertThrows 방법보다 가독성이 좋다.
또한 해당 메서드 이용시 `hasMessageContaining` 메서드를 통해 에러메시지의 검증 또한 가능하다.

## 방법 3. try ~ catch
*************
Java에서 예외를 처리하는 가장 흔한방법으로, 메모리 누수나 loop에서의 효율성등의 이유로 권장하지 않는다.

```java
@Test
public void junit5_tryCatch() {
    try {
        OnlyUseException.error();
    } catch (RuntimeException e) {
        Assertions.assertEquals("Something Error!", e.getMessage());
    }
}
```
위 코드의 경우에도 예외를 처리하는 용도가 아닌, 에러를 검증하기 위해 사용되기에 적합하지 못하다.
왜냐하면, 예외 메시지의 경우 변하기 쉬운값이며 리팩터링에 대한 내성이 없는 테스트코드가 되버린다.

또한 코드의 명확성이 떨어져 결국 어떤것을 테스트하기 위함인지가 불분명해진다.

## 방법 4. 예외메시지 테스트 - assertThrows 반환값 사용
*************
try ~ catch 구조에서 발전한 형태로 예외메시지만을 테스트하기 좋다.

```java
@Test
public void junit5_exceptionMessage() {
    Throwable exception = assertThrows(RuntimeException.class, () -> {
        OnlyUseException;
    });
    assertEquals("Something Error!", exception.getMessage());
}
```

### 종합적으로
개인적으로 가장 선호하는 방식은 `assertThatThrownBy()`를 사용하는 방식이다.

코드의 명시성과 가독성을 챙기면서, 예외메시지나 부가적인 테스트또한 수행할 수 있기에 선호한다.

## 여담
Assetions의 assertThorws의 경 기본적으 **3가지 메서드로 오버로딩** 되어있다.

```markdown
static <T extends Throwable>T assertThrows(Class<T> expectedType, Executable executable)

static <T extends Throwable>T assertThrows(Class<T> expectedType, Executable executable, String message)

static <T extends Throwable>T assertThrows(Class<T> expectedType, Executable executable, Supplier<String> messageSupplier)
```

기본적으로 첫인자에 예외타입지정하고 두 번째 인자에 Lambda를 통해 예외를 테스트할 메서드를 작성한다.

또한 내부적으로 부모 타입을 상속한 예외인지 검사하도록 **Assertions.assertThrows**가 구성되어있다.

```java
private static <T extends Throwable> T assertThrows(
    Class<T> expectedType, Executable executable, Object messageOrSupplier) {
 
    try {
        executable.execute();
    }
    catch (Throwable actualException) {
        if (expectedType.isInstance(actualException)) {
            return (T) actualException;
        }
```

그렇기에 `NullPointException`을 유발하는 메서드를 `RuntimeException`으로 검증하여도 테스트는 통과된다.