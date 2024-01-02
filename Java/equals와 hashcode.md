# equals(), hashcode

Object Class에 정의된 equals의 설명을 읽어보자

<img width="1420" alt="스크린샷 2024-01-02 오후 8 59 02" src="https://github.com/TODO-List-Study/Todo-List-Study/assets/39437170/8c2fedae-17cc-4fdd-befc-0a94264e82aa">

기본적으로 a.eqeuals(a)의 경우, a.equals(b)가 true일때 b.eqauls(a) 또한 true를 반환해야한다는 조건이 있다.

또한 동등한 비교에 사용되는 정보가 수정되지 않는 한, null이 아닌 참조 값 x 및 y에 대해 x.equals(y)의 여러 호출이 일관되게 true를 반환하거나 false를 일관되게 반환해야하는 일관성을 강조한다.

여기서 우리가 눈여겨 봐야할 부분은 아래부분이다.

> Note that it is generally necessary to override the hashCode method whenever this method is overridden,
> so as to maintain the general contract for the hashCode method,
> which states that equal objects must have equal hash codes.

equals()를 오버라이딩 할 때마다 hashCode()도 함께 재정의하는 것을 권장하여, 동일한 개체가 동일한 해시 코드를 가지는 것을 권장한다.

이에 대해 자세히 살펴보자

## equals()
```java
public boolean equals(Object obj) {
        return (this == obj);
    }
```

equals()는 말 그대로 특정 객체와 다른 객체가 **'같은' 객체**인지를 판별해주는 메서드이다.

기본적으로 객체에서 제공하는 equals() 메서드 자체는 비교연산자와 크게 다르지않다.

하지만 원시타입의 Wrapper 클래스들의 equals는 좀 다르다.

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

위 코드는 String 클래스의 equals() 메서드로 단순 주소값을 비교하는 것과 다르게 주소가 다를 경우 String의 value를 직접 비교하여 두 객체가 같은지 다른지를 판단한다.

이렇게 구현되어있는 이유는 참조형이 실제 메모리에 들고 있는 값과, 해당 메모리 주소를 나타내는 주소값 두 가지를 가지고 있기 때문인데 실제로 "a"와"a"라는 문자열 2개가 존재할 때,
주소값을 비교하는 equals를 그대로 사용하게 된다면, 실제로는 같은 문자열이지만 메서드가 개발자의 의도 외로 동작하게된다.

그렇기에 대부분의 참조형 클래스는 equals()를 재정의하여 자신의 특성에 맞게 객체를 비교하고 있다.

## 객체 동등성과 동일성
위의 설명을 읽어서 알겠지만, 객체를 비교하는 2가지 방식이 존재한다.
1. 객체의 주소값을 직접 비교하는 방식 (==)
   - 해당 방법은 동일성을 판단하는 방식이다.
     -  객체의 주소값을 직접 비교하는 경우 동일성을 판단한다고 한다.
   - 서로 다른 변수는 서로 다른 메모리 주소값을 갖기에 가능
2. 주소가 아닌 다른 어떤 값을 이용해 비교하는 방식 (재정의 : override)
    - 위에 String 케이스가 이에 해당한다.
    - 실제로는 서로 다른 값이라도, 의미상으로 같다고 판별되는 경우를 동등성을 보장한다고 한다.

## hashCode()
hashCode()는 객체의 해시 코드 값을 반환해주는 메서드이지만, Object 클래스에만 이렇게 정의되어있다.

일반적인 메서드 구현체는 native 메서드이기에 확인할 수 없다.

즉, 정리하면 hashCode()는 native 메서드로 JDK마다 구현이 다르기에 어떤 알고리즘으로 구현되었는지는 딱 정의할 수 없다.

어차피 용도는 해시코드의 반환이니 신경쓰지 않아도될듯하다.

그럼 이제 우리에게는 가장 큰 질문이 남아있다.

## 그래서 왜 equals()를 재정의할 때 hashCode()도 재정의해야 할까?
초반에 말했던 공식문서의 이야기이다.

equals()와의 관계를 알아보자. 이는 hashCode()의 java document를 보면 알 수 있다.

> Returns a hash code value for the object. This method is supported for the benefit of hash tables such as those provided by java.util.HashMap.

hashCode는 자체적인 사용목적보다는 HashMap와 같은 hash table을 지원하기 위해 만들어진 메서드이다.(Hash알고리즘을 사용하는 자료구조체)

여기서 말하는 구조체들은 일반적으로 개발자가 Key-Value를 지정하는 것이아닌 해시함수를 통해 얻은 해시값을 Key값으로 사용한다.

또한, Java에서 해시테이블 자료구조의 key는 모두 hashCode()를 통해 만들어진다
