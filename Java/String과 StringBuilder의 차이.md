- [목차]
- [String과 StringBuilder?](#String과-StringBuilder?)
- [String](#String) 
- [StringBuilder](#StringBuilder)
- [String,StringBuilder 무엇이 더좋은가?](#String,StringBuilder-무엇이-더좋은가?)
  - [코드적으로 비교해보자](#코드적으로-비교해보자)
- [StringBuffer?](#StringBuffer?)
  -[StringBuffer와 StringBuilder의 차이점](#StringBuffer와-StringBuilder의-차이점) 
- [최종정리](#최종정리)
# String과 StringBuilder?

* String과 StringBuilder 둘 모두 자바에서의 문자열을 사용하기위한 자료형으로 많이 사용된다.
* **하지만** 이 둘의 한계점과 이용용도는 차이가 있다. 지금부터 그에 대해 기술하려한다. 

## String 
* 문자열을 사용하기 위해 보편적으로 가장 많이 사용되는 객체로서 가장 큰 특징으로는 3가지정도가 있다.

1. 불변(immutable)객체이다.
2. String형의 문자열을 합치기 위해 연산시에는 새로운 객체가 생성된다.
3. 루틴에서 같은 작업반복시 성능이 저하된다.

* String 객체에는 이러한 명확한 단점이있다. 이를 극복하고자 나온것이 StringBuilder이다.
* String 클래스의 최대 단점은 **힙 메모리에 수많은 가비지가 생성**되어 힙메모리가 부족하여 어플리케이션 성능에 치명적인 영향을 끼칠 수 있다.

## StringBuilder
* 앞서 말한것처럼 StringBuilder 객체 또한 문자열을 사용하기 위한 객체로서 기존의 String 객체와 대조되는 특징이 3개정도 된다.

1. 가변(variable)객체이다.
2. 연산시에 새로운 객체를 생성하지않고 기존에 생성된 데이터에 더해주는 방식으로 사용된다.
3. 다양한 메서드가 지원된다.

* StringBuilder는 append(),deleteCharAt()같은 메서드가 지원되기 때문에 문자열의 추가,수정,삭제가 편하다.

## String,StringBuilder 무엇이 더좋은가?
이전까지 설명한 것만으로는 당연히 StringBuilder가 좋아 String을 이용할 이유가 없는것 처럼 설명되었지만
개발자로서 가장 좋은 선택지로는 **각 객체의 특징을 이해하고 상황에 따른 객체사용을 적절히 이용**하는 것입니다.

예를 들어 회원이 들어올 때 마다 안녕하세요 + 회원이름 + 님 이라는 문자열을 String형으로 보여주는 경우
계속해서 **불필요한 객체를 생성**하여서 사용을 해야합니다.

**하지만** StringBuilder 사용시에는 불필요한 객체생성없이 최초 생성한 객체에 
덧붙이거나 수정하면 되기 때문에 성능면에서 이득이라고 볼 수 있습니다.

또한 고객의 정보를 받는 프로그램에서 고객의 주민번호를 받게된다면 주민번호는 불변해야하기에 String형태로 받는것이 좋다고 불 수 있습니다.

* 결국 개발자가 상황에 따라 알맞는 객체를 이용해야한다고 할 수 있습니다.

### 코드적으로 비교해보자

```java
public class Test(){
  public static void main(String[]args){
    Scanner scanner=new Scanner();
    String username=scanner.next();
    for(int i;i<1000;i++){
      System.out.println("username = " + username + i.toString());
    }
  }
}
```

=> 이를 String 형태로 출력할경우 1000번의 for문 루틴동안 String 객체를 새로 생성하게된다. 

그렇게 되면 전체적인 프로그램의 성능저하를 초래할 수도 있기 때문에 StringBuilder를 이용할경우를 보자

```java
public class Test(){
  public static void main(String[]args){
    Scanner scanner=new Scanner();
    StringBuilder username= new StringBuilder(scanner.next());
    for(int i;i<1000;i++){
      System.out.println("username = " + username.append(i.toString()));
    }
  }
}
```

=> StringBuilder의 경우 객체를 재생성하는게 아닌 최초의 데이터에서 더해가기 때문에 효율성이 좋아진다.

## StringBuffer?

* StringBuffer는 StringBuilder와 동일한 Api를 가지고있으며 StringBuilder와 유사한 성격을 가지고 있습니다.

1. 가변객체이다.
2. 추가 수정 삭제등의 메서드가 존재한다.
3. 어플리케이션 부담이 적다.

### StringBuffer와 StringBuilder의 차이점

그렇다면 동일 Api를 가진 StringBuffer와 StringBuilder의 차이점은 무엇이 있을까?
가장 큰 차이점으로는 역시 **동기화의 유무**로서 StringBuffer는 멀티쓰레드환경에서 동기화 키워드가 지원되기에
멀티쓰레드 환경에서 안전하게 이용(thread-safe)하기엔 StringBuffer가 더좋고 
동기화가 지원되지않는 StringBuilder는 동기화를 고려하지않는 단일 쓰레드에서 성능을 더 효율적으로 발휘할 수 있습니다.
(참고.String도 불변성을 가지기 때문에 멀티쓰레드 환경에서는 안정성(thread-safe)을 가지고 있습니다.)

## 최종정리

* String : 문자열 연산이 적고 멀티쓰레드 환경, 값이 불변해야하는경우
* StringBuffer: 문자열 연산이 많고 멀티쓰레드 환경일 경우
* StringBuilder: 문자열 연산이 많고 단일쓰레드거나 동기화를 고려하지않는경우


![출처](https://docs.microsoft.com/ko-kr/dotnet/api/system.text.stringbuilder?view=net-6.0#StringAndSB)