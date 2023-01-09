## String vs StringBuffer/StringBuilder

이 세객체에서 String이 가지는 차이점은 불변성(immutable)을 갖는다는 점이다.

```java
String str = "hello";   // String str = new String("hello");
str = str + " world";  // [ hello world ]
```
위의 예제에서 "hello" 값을 가지고 있던 String 클래스의 참조변수 str이 가리키는 곳에 저장된 "hello"에 "world" 문자열을 더해 "hello world"로 변경한 것으로 알고있었다.

하지만, 기존의 **"hello"** 값이 들어가있던 String클래스의 참조변수는 str이 "hello world"라는 값을 가지고있는 새로운 메모리영역을 가리키게 변경되며,
처음 선언했던 "hello"로 값이 할당되어있던 메모리영역은 Garbage로 남아있다가 GC가 수거해간다.
요약하자면 문자열에 수정이 가해진 순간 새로운 인스턴스를 생성하여 대체된 것이다.

![String](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99948B355E2F13350F)

이렇듯 String은 불변성을 보장받기에 변하지 않는 문자열을 자주 읽어들이는 경우 String을 사용하는 것이 효율성이 높다.
그러나 수정,추가,삭제등의 작업이 빈번할 경우 Heap 영역에 수많은 Garbage가 생성되어 힙 메모리 부족으로 인한 치명적 영향이 발생한다.

이와 반대로 StringBuffer/StringBuilder의 경우 가변성을 가지며, 수정/삭제 등의 작업이 API로 존재하기에
동일 객체 내에서 문자열의 변경작업이 가능해진다.

```java
StringBuffer sb= new StringBuffer("hello");
sb.append(" world");
```

![StringBUffer](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F9923A9505E2F133608)

## StringBuffer vs StringBuilder

동일 API를 지닌 이 두객체의 차이로는 동기화의 유무차이가 존재한다.
StringBuffer는 동기화를 지원하기에 멀티쓰레드 환경에서 안전하다(thread-safe)
이는 String도 불변성을 가져 thread-safe의 성질을 가진다.

반대로 StringBuilder의 경우 동기화를 지원하지 않기에 멀티스레드 환경에 적합하지 않고 동기화를 고려하지 않는 상황(싱글스레드)에서의 성능은
StringBuffer보다 뛰어나다.


### 최종
String : 문자열 연산이 적고 멀티쓰레드 환경일 경우
StringBuffer : 문자열 연산이 많고 멀티쓰레드 환경일 경우
StringBuilder :  문자열 연산이 많고 단일쓰레드이거나 동기화를 고려하지 않아도 되는 경우  

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99BE23375E2F133722)