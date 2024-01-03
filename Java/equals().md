# equals와 == 의 차이
Java에서의 동등성의 개념을 아는가?

쉽게 말해 두 객체를 비교할 때 온전히 같다면 동일하고, 객체는 다르지만 내부 메시지(필드 값)가 같다면 동등하다고 볼 수 있다.

## == (등가 비교 연산자)
비교 연산자는 두 피연산자를 비교하는 데 사용되는 연산자로서, 피연산자의 값이 같은지 또는 다른지를 비교하는 연산자가 등가 비교 연산자로, Java는 !=와 ==를 제공하고 있다. 

사실 `==`은 기본형은 물론, 참조형에도 사용할 수 있다.
보통, 주소값을 비교하기 위해 사용된다.

## equals()
Object 클래스에 포함된 메서드로, 매개변수로 객체의 참조변수를 받아 비교하여 그 결과를 boolean으로 알려주는 메서드이다.

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

기본적으로 해당 구조로 구성되어있으나, 하위 클래스 수준에 도달할 수록 재정의하여, 사용한다.
대표적인 예시가 String의 equals() 재정의 메서드이다.

## == 사용시 주의 사항
- Integer의 경우 == 비교 연산 시 값이 같더라도 주소가 다르다면 ==으로 비교하면 false
- String의 경우 같은 Case로 생성한 경우 주소값이 같게 표기
  - new String으로 생성한 결과 다른 주소값을 가짐
    - 그 이유는  new 연산자에 의해 메모리 할당이 이루어지고, 따라서 항상 새로운 String 인스턴스가 생성됨
    - 문자열 리터럴을 직접 지정할 경우는 Java 소스파일에 포함된 모든 문자열 리터럴은 컴파일할 때 자동적으로 미리 생성
      - 생성 요청이 들어왔을 때 이미 같은 내용의 문자열 리터럴이 있다면, 새로 만드는게 아니라 기존의 것을 재사용하는 식으로 동작
    

**예시 코드**
```groovy
@Test
@DisplayName("문자열 참조 비교")
void compareString() {
    String test1 = "testString";
    String test2 = "testString";
    String test3 = new String("testString");
    String test4 = new String("testString");

    assertEquals(System.identityHashCode(test1), System.identityHashCode(test2));
    assertNotEquals(System.identityHashCode(test3), System.identityHashCode(test4));
}
```

## Java의 문자열 리터럴 관리
1. 문자열 리터럴을 선언한다.
2. .java 소스파일이 .class 클래스 파일로 컴파일되는 과정에서 문자열 리터럴이 클래스 파일에 저장된다.
3. 클래스 파일이 클래스 로더에 의해 메모리에 올라갈 때, 클래스 파일에 있는 문자열 리터럴 목록의 모든 리터럴들이 JVM 내에 있는 **상수 저장소 (constant pool)**에 저장된다.
4. 문자열 리터럴을 지정하여 생성된 String 변수에 상수 저장소의 주소를 할당한다.

**그림으로 보자면 이런느낌**
<img width="695" alt="스크린샷 2024-01-03 오후 9 13 37" src="https://github.com/TODO-List-Study/Todo-List-Study/assets/39437170/f7cdf4b1-8b3e-4581-ac89-357125ea18be">

**🤔 그래서 == 이 왜 된다고?**
결국 주소 값이 같기 때문이다. JVM에 의해 같은 주소에 있는 값을 재활용한다고 생각하면된다.

**그럼 equals()는?**
```java
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String aString = (String)anObject;
            if (coder() == aString.coder()) {
                return isLatin1() ? StringLatin1.equals(value, aString.value)
                                  : StringUTF16.equals(value, aString.value);
            }
        }
        return false;
    }
```

구현체를 보면, 같은 객체라면 바로 true를 반환하고 같은 주소 값을 가진 객체가 아니더라도, 내용 값이 같은지 한번 더 검증한다.
그렇기에 String의 경우는 equal()로 비교하는게 확실함과 동시에, **null safe**하다고 생각한다.

## 🤔 String은 알겠고 Integer는요?
Integer의 경우 Integer 클래스의 Integer Cache에 대해 고려해야한다.

Integer를 할당하는 예시로 `Integer number1 = 100;` 이렇게 할당할 경우 원시타입은 컴파일러에 의해 auto boxing 되어 `Integer number1 = Integer.valueOf(100);` 형태로 컴파일 된다.

### valueOf 내부 톺아보기
```java
    /**
     * Returns an {@code Integer} instance representing the specified
     * {@code int} value.  If a new {@code Integer} instance is not
     * required, this method should generally be used in preference to
     * the constructor {@link #Integer(int)}, as this method is likely
     * to yield significantly better space and time performance by
     * caching frequently requested values.
     *
     * This method will always cache values in the range -128 to 127,
     * inclusive, and may cache other values outside of this range.
     *
     * @param  i an {@code int} value.
     * @return an {@code Integer} instance representing {@code i}.
     * @since  1.5
     */
    @IntrinsicCandidate
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
       
```

내부 구현체는 입력 값이 IntegerCache의 log~high 범위에 있다면 IntegerCache의 캐시 주소값을 리턴하고, 
아닌 경우 new를 이용해 새로운 Integer 인스턴스를 생성해 리턴하도록 설계되어있다.

**주석 내용의 경우 아래와 같은 내용을 담고 있다.**
> 새로운 Integer 인스턴스가 필요하지 않은 경우, 이 메서드는 자주 요청되는 값을 캐싱하여 공간 및 시간 성능을 크게 향상시킬 수 있으므로 일반적으로
> Integer(int) 생성자보다 이 메서드를 우선적으로 사용해야 합니다.
> 이 메서드는 항상 -128~127 범위의 값을 캐시하며, 이 범위를 벗어난 다른 값은 캐시할 수 있습니다.

```java
private static class IntegerCache {
  static final int low = -128;
  static final int high;
  static final Integer[] cache;
  static Integer[] archivedCache;

  static {
    // high value may be configured by property
    int h = 127;
    String integerCacheHighPropValue =
            VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
    if (integerCacheHighPropValue != null) {
      try {
        h = Math.max(parseInt(integerCacheHighPropValue), 127);
        // Maximum array size is Integer.MAX_VALUE
        h = Math.min(h, Integer.MAX_VALUE - (-low) -1);
      } catch( NumberFormatException nfe) {
        // If the property cannot be parsed into an int, ignore it.
      }
    }
    high = h;

    // Load IntegerCache.archivedCache from archive, if possible
    CDS.initializeFromArchive(IntegerCache.class);
    int size = (high - low) + 1;

    // Use the archived cache if it exists and is large enough
    if (archivedCache == null || size > archivedCache.length) {
      Integer[] c = new Integer[size];
      int j = low;
      for(int i = 0; i < c.length; i++) {
        c[i] = new Integer(j++);
      }
      archivedCache = c;
    }
    cache = archivedCache;
    // range [-128, 127] must be interned (JLS7 5.1.7)
    assert IntegerCache.high >= 127;
  }

  private IntegerCache() {}
}
```

- low 값은 항상 -128로 고정이고, high는 디폴트는 127이고 상한 값을 `AutoBoxCacheMax` 설정을 건드리는 것으로 수정가능하다.
- 해당 캐시는 Integer 배열 형태로 관리된다. 
- 한 마디로, 캐싱 범위에 속한다면, 이미 존재하는 Integer 인스턴스의 주소를 참조하게 되었기에 서로 다른 두 변수가 같은 주소를 공유하게된다.
- 즉, == 연산이 true가 반환된다.

## 결론
Integer 같은 경우는 드물겠지만, String의 경우 웹 프로젝트 또는 Java Application을 구현하며, 종종 발생하는 이슈라고 생각한다.

그렇기에 필자는 equals를 사용한 통일성을 중요시한다. 하지만, 참조형은 equals사용! 이라고 암기하는 것과 동작원리를 알고 사용하는 것은 차이가 크다고 생각한다.

그렇기에 동작원리에 대해서 꼼꼼히 살펴보자