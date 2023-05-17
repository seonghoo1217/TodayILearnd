## 소개 배경
```markdown
2023-05-11 21:04:02.549 ERROR 1692 --- [nio-8080-exec-1] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is org.springframework.http.converter.HttpMessageConversionException: Type definition error: [simple type, class project.mogakco.global.dto.init.InitDTO$BasicSetting]; nested exception is com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `project.mogakco.global.dto.init.InitDTO$BasicSetting` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)
at [Source: (PushbackInputStream); line: 2, column: 5]] with root cause

com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `project.mogakco.global.dto.init.InitDTO$BasicSetting` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)
```

현재 단순 API에서 사용하는 DTO 클래스에서 `Jackson` 클래스의 오류가 발생하였다.

맨 처음에는 이전에 DTO에서 `@Getter`가 없어 발생하는 406에러인줄 알고, 이전 Trouble Shooting 경험을 양분삼아, DTO쪽을 확인해봤으나 문제가 될만한 코드는 없었다.

하지만, 최종적으로 이 에러를 해결하며 문제가 되었던 부분은 DTO가 맞았고, 상당히 새로운 이슈였기에 포스팅하게되었다.

## Trouble(문제 원인)
결론을 먼저 말하자면 해당 이슈는 `boot-starter` dependency에 기본적으로 포함되어 있는 `jackson library`가 빈 생성자가 존재하지 않는 모델을 생성하는 방법을 몰라 발생하는 에러였다.

**에러를 유발하던 상태**
```java
    @Getter
    @Builder
	public static class BasicSetting{
		private String oauthId;
		private String fcmToken;
	}
```

**API Controller**
```java
@PostMapping("/basicSet")
	public void initUserInfoBasicSetting(@RequestBody InitDTO.BasicSetting basicSetting) throws IOException {
		initService.basicSetting(basicSetting);
	}
```

그렇지만, 이상했던 점은 해당 API에서만 `@RequestBody`를 사용하는 형식이 아니라 모든 API 구조가 같은데 해당 구조에서만 에러가 발생한다는 점이 이해가 되지않았다.

그래서 왜 해당 API에만 이러한 에러가 발생하는지를 알아보고자한다.

## 원인 파악
이유는 간단했다. `@RequestBody`로 클라이언트에게 데이터를 전달받을 때 DTO를 통해 데이터를 전달받는다.

그런데 이때 DTO가 `@Builder` 어노테이션을 사용하는데 자기생성자가 존재하지 않기 때문에 발생하는 이슈였다.

자기생성자가 필요한 이유는 이 경우 **Jackson 라이브러리**는 클라이언트로 부터 전달 받은 JSON 데이터를,
해당 DTO 클래스로 변환 즉, 역직렬화하는 과정에서 생성자 대신 빌더 패턴을 이용하려하지만, 생성자가 없기에 에러가 발생한다.

**🤔 엥 DTO에서 @Builder사용해서 사용하는 데 문제 없었는데요?**

**DTO 예시**
```java
    @Getter
	@Builder
	public static class RankingDTO{
		private Long member_seq;

		private String nickname;

		private String member_imgUrl;
	}
```

**DTO Builder 패턴 사용 예시**
```java
public MemberResponseDTO.RankingDTO toRankingDTO(){
		return MemberResponseDTO
				.RankingDTO.builder()
				.member_seq(member_seq)
				.nickname(nickname)
				.member_imgUrl(member_imgUrl)
				.build();
	}
```

보통 DTO에서 Builder를 사용하는 경우는 Builder를 이용하여 Entity를 DTO화 시킬 때 사용하거나 전달받은 값을 DTO로 구성하는 용도로 사용한다.

이 경우에는 Builder 패턴에 자체적으로 매개변수를 전달받아 생성하기에 Builder 패턴을 이용하더라도 자기생성자가 필요하지않고, Jackson 라이브러리가 역직렬화하는데도 문제가 발생하지 않았다.

이를 좀더 명확하게 이해하기 위해선 Jackson 라이브러리의 동작원리에 대해 알아야한다.

### 원인파악 - 1
Json 데이터를 Java의 `Object` 형태로 반환할 때 `PostMapping`의 경우 `Jackson2HttpMessageConverter`에서 변환이 이루어진다.

Jackson2HttpMessageConverter의 **동작원리**는 아래와 같다.

1. 요청 처리 : 클라이언트가 JSON 형식의 데이터를 요청 본문에 포함하여 서버에 전송할 때, Jackson2HttpMessageConverter는 요청 본문의 JSON 데이터를 읽어 Java 객체로 변환한다 
이 과정을 **역직렬화(deserialization)** 라고 칭한다.
2. 변환 처리 : Jackson2HttpMessageConverter는 ObjectMapper라는 Jackson 라이브러리의 핵심 클래스를 사용하여 JSON과 Java 객체 간의 변환을 수행한다.

이러한 경로로 Java Class 즉, 개발자가 지정한 DTO Class로 변경된다. 

이 과정에서 @Builder를 사용한다고 명시 된 경우 빌더패턴을 이용해 변환하려하지만, 자기 생성자가 존재하지 않기 때문에 에러를 발생시킨다.

**🤔 다른 DTO의 경우에는 역직렬화할때 문제가 발생안하나요?**

**DTO 예시**
```java
@Getter
	public static class timerRecodeInfoToday{
		private String hours;
		private String minute;
		private String second;
		private String fcmToken;
		private LocalDate timerCreDay;
		private String oauthId;

	}
```

일반 타 DTO의 경우 @Getter 어노테이션이 적용되어 있으며, Lombok의 기능을 사용하여 Getter 메서드가 자동으로 생성된다.

이 경우에는 Builder 패턴이나 기본 생성자의 유무와는 별개로, Getter 메서드가 필요한 맴버 변수들에 대한 접근자를 자동으로 생성하므로 역직렬화 이슈가 발생하지않게되는 것이다.

Jackson 라이브러리가 Getter 메서드를 사용하여 객체의 속성에 접근하므로, Getter 메서드가 존재하면 객체를 역직렬화할 수 있다.


## Trouble Shooting(문제 해결)
이를 해결하기 위해선 두가지 방법이 존재한다.

1. Builder 패턴을 사용하지 않아도 된다면 제거하는 방법

2. 자기생성자를 명시적으로 사용해주는 방법

당연히도, 필자는 자기생성자를 사용하는 방식을 이용했다.

**변경된 코드**
```java
	@Getter
	@NoArgsConstructor(access = AccessLevel.PROTECTED)
	public static class BasicSetting{
		private String oauthId;
		private String fcmToken;

		@Builder
		public BasicSetting(String oauthId, String fcmToken) {
			this.oauthId = oauthId;
			this.fcmToken = fcmToken;
		}
	}
```

이번 Trouble Shooting을 진행하며 알게 된 사실은 빌더 패턴을 사용할 때 뿐만아니라 좋은 코드 관행을 위해, 빌더 패턴을 사용하는 클래스에는 명시적으로 기본 생성자를 추가하는
것이 권장되며, Lombok의 @NoArgsConstructor 어노테이션을 사용하여 명시적으로 자기 생성자를 생성하는 방법이 클린 코드에 가깝다는 것이다.

결과적으로 이를 통해 코드의 가독성과 유지보수성을 높일 수 있는 방법이다.

## 결론
자기생성자를 명시적으로 생성하는 것이 가독성과 클린코드에 이점이 있다는 부분을 새롭게 알았지만, 한편으로는 과연 모든 코드에 존재한다면 괜히 불필요한
 레거시 코드를 늘리는 것은 아닌가에 대한 고민이 또 생겨나게 되었다.

클린코드와 효율성에서 중심을 잡기란 항상 어려운 부분이라고 생각이들며, 이를 중점적으로 공부하는 방향으로 나아가야한다.