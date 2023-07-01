# String Class
`String` Data Type의 경우 많은 개발자들이 int,long과 같이 간편하게 사용하지만 이는 엄연한 Class로서,
성능에 영향을 끼칠 수 있는 요인 중 하나이다.

String도 결국 Class(객체)인 만큼 메모리와 GC에 많은 영향을 끼친다.

## String Class Bad Practice
`Object`와 기본 자료형을 제외하고 가장 많이 사용되는 객체 중 하나가 **String**이다.

하지만 앞서 말했다시피 loop 또는 대용량 자료형을 처리하는데 String Class를 사용하게 된다면?

그만큼의 메모리 사용량이 증가되게 되고, 이는 규모와 서비스 환경에 따라 감당할 수 없는 규모로 커질 수 있다.

이를 해결하기 위해선 보통, `StringBuffer`나 `StringBuilder`를 활용하여 문자열을 처리하도록한다.

위의 두 클래스 모두 문자열 연산에 대해 효율적이며, 메모리 효율성과 속도마저 빠른 것을 보장한다.

## StringBuilder, StringBuffer
StringBuilder와 Buffer모두 JDK5.0 버전에서 추가된 클래스로 동일한 기능을 제공하지만,
차이점이 있다면 **Thread Safe의 유무**이다.

StringBuffer는 Thread Safe하기 때문에 멀티스레드 환경에서 객체를 처리해도 문제되지 않지만, Builder는 문제를 일으킨다.

## 성능비교

| Benchmark                     | (delay) | (steps) | Mode | Cnt          | Score         | Error | Units |
|-------------------------------|---------|---------|------|--------------|---------------|-------|-------|
| chapter.Case04.useStringBufferCase    | N/A     | N/A     | avgt | 993,289      | 476           | ns/op |
| chapter.Case04.useStringBuilderCase   | N/A     | N/A     | avgt | 1,699,145    | 880           | ns/op |
| chapter.Case04.useStringCase          | N/A     | N/A     | avgt | 411,855,515  | 385           | ns/op |

위의 성능비교 표를 보면 알 수 있듯 Buffer가 가장 빠른 속도를 보장한다.

결과적으로는 String의 지표가 압도적으로 낮은 것을 확인할 수 있고 이러한 원인이 발생하는 원인은 String Class의 동작원리에 있다.

### String Class의 원리
String 객체에 loop를 순회하며, 문자열의 연산을 수행할 때 `Heap Area`에 load된다.

다음순번 loop에 str += valueStr 코드가 수행되면 기존 주소, 기존 str 객체는 Garbage가 되고, 새 주소, 새 str 객체가 새로 생성된다.

이를 Loop 동안 반복하기에 성능이 좋을 수가 없다.

### String Buffer, Builder 원리
String Buffer나 StringBuilder는 String과 다르게 새로운 객체를 생성하지 않고, Heap Area에 이미 존재하는 객체의 크기를 증가시키면서 값을 더해간다.

그렇다고 해서, String을 사용하는것이 무조건 나쁜것이고, String Buffer나 String Builder를 반드시 사용해야 하는것은 아니다.

아래와 같은 상황에 유용하게 이용할 수 있다.

1. String은 짧은 문자열을 더할 경우 사용

2. StringBuffer는 Thread Safe 프로그램이 필요 할때, 개발 중 시스템 부분이 Thread Safe 인지 모를 경우 사용하면 좋다.

3. Class에 static으로 선언한 문자열을 변경, singleton으로 선언된 Class에 선언된 문자열일 경우 StringBuffer를 반드시 사용해야 한다.

4. StringBuilder는 Thread Safty 상관 없이 전혀 관계 없는 프로그램을 개발할때, method 내 변수 선언은 그 method안에 지역 변수로 존재하므로 StringBuilder를 사용하면 된다.

## Ref
[자바-성능-튜닝-이야기](https://www.yes24.com/Product/Goods/11261731)