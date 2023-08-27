![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcVd3H0%2FbtssgKJjkle%2FkYY11VSXE4XQQVcjvBE8Y1%2Fimg.png)

현재 진행중인 프로젝트에서 STOMP 프로토콜을 도입해야하는 일이 생겼다.

그렇기에 STOMP 관련 Config를 작성하고 Front(Client)와 Connection을 테스트 해보았는데 CORS이슈가 발생하였다

프로젝트 REST 구조의 첫 테스트가 아닌 기존 API가 존재하였고, 동작에 이상이 없었기에 뜬금없다고 생각하여 트러블 슈팅 결과 나와같은 이슈를 겪고 있는 사람이 존재하지 않을까 싶어 포스팅하게 되었다.



## Trouble(문제 원인)
결론적으로 이슈가 되었던 부분은 현재 금전적인 리소스 비용을 축소하고자 ngrok이라는 미들웨어 프로그램을 사용 중이었다.



또한, JWT 토큰의 유효성을 검증하기 위한 Filter를 Custom하여, Request와 Response에 대한 로직이 존재하였고 ngrok에서 미들웨어 역할을 수행하기에 CORS에러를 이전에 유발하여, Client에게 값을 전달하는 시점 즉, Servlet이 Response되는 시점에 setHeader등을 통한 메서드를 이용해 직접적으로 수정하는 로직을 구현했었는데 바로 이부분이 문제가 되었다.



## 원인 코드
현재 STOMP 프로토콜을 구성하기위해 두 가지 클래스들을 수정하였었다.

첫 번째는 STOMP 관련 Config 클래스로 MessageBroker를 상속받은 형태의 클래스이다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fd3VmAD%2FbtsshntC8Ff%2FKVTamMxfrX3ClMY3oM8pg0%2Fimg.png)

위의 코드에선 기본적으로 지정해주어야할 STOMP Connect의 엔드포인트를 지정해주고 해당 엔드포인트에 대한 CORS 설정으로 모든 접근에 대해 허용을 해놓은 상태이다.



두 번째로는 기존의 CORS에 관한 Config Class로서 Client(Next13환경)와 통신시에 발생하는 CORS 이슈를 해결하기 위해 작성한 클래스이다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FD57eF%2FbtssfRhYZHT%2FEKy5V61KQYrIj8gU410sS0%2Fimg.png)

여기서 봐야할 부분은 addCorsMappings 메서드로 Front의 URL에대해 접근을 허용하고 소켓 사용시 서버의 URL이 CORS를 유발할 수 있는 요소가 될 수 있기에 허용하였다.



마지막으로 문제가 되었던 부분이 CustomFilter를 작성한 JWT관련 Filter 부분이었다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbhdVm8%2FbtssfXCsDu1%2FVEoH8yAWWw7qEktJkwcXek%2Fimg.png)

filter의 일부분을 가져온 코드인데, 잘살펴보면 어떤 요청이던 상관없이 Ngrok의 미들웨어 파싱에러를 해결하기위해 작성한 Header 설정이 적용되는 모습을 볼 수 있다.


여기서 에러가 나게된 이유는 쉽게 표현하자면 설정이 덮였기 때문에 에러가 발생한것이다.

코드를 보면 Client에게 전달되는 Header를 set을 통해 강제로 수정하기에 기존 STOMP CORS와 기존 CORS Config가 사라지게 된것이다.



예시를 들어 Bean이 등록되는 순서가 WebConfig,STOMP 순서대로 일어난다고 가정을해보자. 그렇다면 밑의 그림과 같이 Header에는 Web CORS와 STOMP CORS가 정상적으로 등록되었다가 Client에게 전달을 위해 Filter를 거치면서 설정이 사라지기에 CORS에러가 발생한 것이었다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbMDcvg%2FbtssgKilZbI%2F4INh63AQxbqzadmZZEDC9k%2Fimg.png)

## Trouble Shooting(문제 해결)
정말 간단하게 Filter에서 해당 코드를 제거하고 WebConfig Class에 Ngrok관련 CORS 에러를 해결할 수 있는 설정을 추가하였다.



## 결론
직접 Filter나 Interceptor와 Handler를 구현하여 Request와 Response에 추가적인 로직을 구성할 수 있는 구조를 선호하여 항상 이렇게 작업을 해오다가 처음으로 만난 당황한 에러가 아닌가 싶다.



Error Debug와 Logging의 중요성을 다시 깨닫게된다.x