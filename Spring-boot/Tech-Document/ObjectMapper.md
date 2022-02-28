# ObjectMapper란?
JSON으로 이루어진 컨텐츠를 Java로 *deserialization 하거나 Java 객체를 JSON으로 *serialization 할 때 사용하는 Jackson라이브러리의 클래스이다.
ObjectMapper는 생성비용이 비싸므로 보통 bean으로 등록하거나 static으로 공유하여 사용한다.

* serialization,deserialization: 객체 직렬화/객체 역직렬화: 직렬화는 객체의 내용을 바이트 단위로 변환하여 파일 또는 네트워크를 통해서  스트림(송수신)이 가능해지게 해주는것. 역직렬화는 데이터를 객체로 조립한다고 생각하면 편하다.

## 예시를 통하여 알아보기

```java
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

@Getter
@ToString
@AllArgsConstructor
@NoArgsConstructor
public class Laptop {

    private String Cpu;
    private String GTX;
    private String Ram;
}
```

라는 클래스가 존재할경우 이것은 Java객체이기 때문에 JSON으로 serialization하고 싶은 경우 ObjectMapper의 writeValue()메서드를 이용하면된다.


* Java => JSON
```java
import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.File;
import java.io.IOException;

public class ObjectMapperEx {
    public static void main(String[] args) {
        ObjectMapper objectMapper = new ObjectMapper();

	// Java Object ->  JSON
        Laptop laptop = new Laptop("i7-9k","1080ti", "16GB");
        try {
            objectMapper.writeValue(new File("src/Laptop.json"), laptop);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

위와 같이 writeValue() 메서드를 이용하여 JSON파일로 바꿔주면 되는데 이때 파라메터로는 JsonGenerator와 직렬화할 자바객체 클래스가 필요하다.
JsonGenerator에는 스트림파일로 변환가능한 양식의 파일을 경로와 함께 지정해주면된다.
여기서 주의점으로는 직렬화시킬 객체에 필드접근이 가능하도록 @Getter가 있어야한다.
Jackson 라이브러리가 Getter,Setter를 통해 prefix를 제거하고 맨앞을 소문자로 만들어 필드를 식별하기 때문인데 직렬화 시킬 객체에 Getter가 없을경우 필드에 식별과 접근이 불가하여 값을 가져오지 못해 오류가 발생하게된다.

* JSON => Java

```java
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;

public class ObjectMapperEx {
    public static void main(String[] args) {
        ObjectMapper objectMapper = new ObjectMapper();

        // JSON -> Java Object
        String json = "{\"Cpu\":\"i7-9k\",\"GTX\":1080ti,\"Ram\":\"16GB\"}";
        try {
            Laptop deserializedPerson = objectMapper.readValue(json, Laptop.class);
            System.out.println(deserializedPerson);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
    }
}
```

위와 같이 역직렬화를 통해 JSON파일을 Java객체로 바꿀 떄는 readValue() 메서드를 이용한다. JSON형태의 문자열을  매개변수로 넣어주고 객체로 만들어줄 클래스의 이름을 넣어준다.
주의할 점으로는 JSON을 파싱한 결과를 전달할 생성자가 있어야한다.

