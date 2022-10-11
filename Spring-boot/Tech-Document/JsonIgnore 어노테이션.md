# @JsonIgnore

Json과 관련하여 많이 사용되는 패키지의 jackson 패키지의 원문글을 살펴보면 @JsonIgnore 어노테이션에 대해 다음과 같이 적혀있다.

```markdown
Jackson @JsonIgnore, @JsonIgnoreProperties and @JsonIgnoreType
=======

***

This page will walk through Jackson @JsonIgnore, @JsonIgnoreProperties and @JsonIgnoreType annotations example.
These annotations are used to ignore logical properties in JSON serialization and deserialization.
@JsonIgnore is annotated at a class property level to ignore it.
@JsonIgnoreProperties is annotated at class level and we need to specify the logical properties of that class to ignore
them.
@JsonIgnoreType is annotated at class level and it ignores the complete class.
@JsonIgnore and @JsonIgnoreType has an element value which accepts Boolean values to make it active and inactive.
@JsonIgnoreProperties has elements that are allowGetters, allowSetters, ignoreUnknown and value.
The element value in @JsonIgnoreProperties specifies the names of properties to ignore. On this page we will provide
complete example to use @JsonIgnore, @JsonIgnoreProperties and @JsonIgnoreType in JSON serialization and deserialization
step by step
```

해석해보자면 @JsonIgnore를 사용하게 되면 프론트엔드와 협업할때 대부분 RestAPI구조에서 JSON으로 데이터를 입출력하는 경우가 많은데
그 중 숨기고 싶은정보에 해당 어노테이션을 달면 숨길 수 있다. 회원데이터의 password 또는 id를 감출때 많이 사용된다.

좀 더 용어적으로 설명을 하자면 @JsonIgnore , @JsonIgnoreProperties, @JsonIgnoreType 등을 사용하면 JSON 타입의 데이터에 직렬화,
역직렬화에서 속성을 무시하는 용도로 사용할 수 있다.

* 직렬화 : 객체의 내용을 바이트 단위로 변환하여 스트림이 가능한 상태로 만드는것
* 역직렬화 : 역직렬화는 직렬화된 데이터를 받는것이다.

- @JsonIgnore : 클래스의 멤버 변수 수준에서 사용된다.
- @JsonIgnoreProperties : 클래스 수준에서 사용된다.
- @JsonIgnoreType : 전체 클래스를 모두 무시한다.


## 예시

```java
@jsonIgnore
public String myName;


또한 게터 세터 메서드에서도 사용 가능하다.

@JsonIgnore
public String getmyName(){
	return this.myName;
}

@JsonIgnore
public void setmyName(String myName){
	return this.myName = myName;
}
```

위의 예제에서 사용된 myName 필드가
역직렬화할때 서버는
Unrecognized field "myName"
와 같이 에러를 던질 것이다.
직렬화를 할땐 myName 필드가 Json에 담기지 않는다.

@JsonIgnore 어노테이션은 @JsonProperty와 함께 사용될 수 있다.(명시적 선언 그리고 JSON에서 선언된 키값을 맵핑시킴)

```java
@JsonIgnore
@JsonProperty("Name")
private String myName;
```

myName 프로퍼티에 @JsonProperty("myName")를 추가함으로서 에러를 던지지 않으면서
JSON의 직렬, 역직렬화 할때 myName필드를 무시할 수 있다.

또한 @JsonIgnore의 속성값으로(false)를 줄 수 있다

```java
@JsonIgnore(false)
private String myName;
```

@JsonIgnore를 false 속성으로 비활성화 시켰고,
논리적 프로퍼티인 myName을
JSON 직렬화, 역직렬화에 사용할 수 있다.

