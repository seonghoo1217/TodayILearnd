# Web-Socket을 공부하면서 필요한 용어와 이슈들에 대한 정리

## 기본용어정리

* payload: 전송되는 데이터를 의미합니다. 데이터를 전송할 때 Header와 META데이터,에러 체크 비트같은 다양한 요소들과 함께 보내 데이터의 전송 효율과 안정성을 높입니다. 데이터 자체를 의미하는것이 페이로드입니다.

```java
    {
    "status":
    "from":"localhost",
    "to":"http://abcd/ef",
    "methoad":"GET",
    "data":{"message:Hello World"}
    }
```

위와같은 JSON에서 payload는 data뿐만이고 나머지는 통신요소일뿐이다.

* CORS: Cross-Origin Resource Sharing는 추가 HTTP 헤더를 사용해 한출처에서 실행중인 웹 어플리케이션이 다른 출처의 선택한 자원에 접근할 수있는 권한을 부여하도록 브라우저에 알려주는 체제로 리소스가 자신의 출처와 다를 때 호출된다.

* EndPoint: API가 서버에서 자원에 접근할 수 있게해주는 URL로 같은 URL이어도 다른요청을 하게끔 구별해주는 항목이다.


## Web-Socket Session의 동시성
**WebSocketHandler**를 사용하여 핸들러 처리를 할경우 현재 표준상 동시전송을 지원하지않는다.
따라서 STOMP 메시징 프로토콜을 이용하여 메시지 전송을 초기화하거나 WebSocketSession을 ConcurrentWebSocketSessionDecorator로 Wrapping 해야한다.
ConcurrentWebSocketSessionDecorator은 오직하나의 스레드만 메시지를 전송하도록 보장되어있다.

## Web-Socket HandShake
WebSocketHandler 전/후로 필요한 작업이 있을경우 HandShakeInterceptor 인터페이스를 구현하여 등록하면된다.
이를 통해서 HandShake자체를 막거나 WebSocketSession속성을 사용할 수 있다.

## AllowedOrigin

WebSocket을 위해 스프링에선 기본적으로 Same-Origin 즉,동일한 출처의 도메인에 대해서만 커넥션을 수행하겠다는 뜻이다.
하지만 핸들러마다 도메인설정을 통해 다른 도메인에서도 접근이 가능하다.

## SockJS

* SockJS는 WebSocket API를 사용하도록 허용하지만 브라우저에서 WebSocket을 지원하지않는경우 대안으로 런타임중 코드를 고치지않는 대안으로 나온 기술이다.

클라이언트 - 서버 연결을 위한 웹소켓 연결의 안정성과 정확성이 무조건적으로 보장되지않는다.
여러가지 예외상황이 생길수 있다. 예시를 들어보자.

1. 모든 클라이언트의 브라우저에서 WebSocket을 지원한다는 보장이 없다.
2. 클라이언트/서버 중간에 위치한 프록시가 Upgrade 헤더를 못해 서버에 전달하지 못할 수도 있다.
3. 프록시가 도중에 커넥션을 종료시킬 수 있다.

이러한 문제는 SockJS가 지원하는 WebSocket Emulation을 통해서 해결이 가능하다.
WebSocket Emulation이란것은 WebSocket으로 첫번째 연결시도하고 WebSocket연결이 실패한경우 WebSocket 이전의 기술들을 사용한다.
Http Streaming스펙으로 다시 연결요청을보내고 이마저도 안된다면 HTTP Long Polling 으로 연결요청을 한다.

WebSocket Emulation에 과정을 간단히 말하자면 SockJS가 서버로부터 정보를 얻기위해 GET/info로 요청을 보낸다.
클라이언트가 서버에 요청을 보내므로써 서버의 WebSocket 지원여부,CORS를 위한 Origin 정보 등의 정보를 응답으로 받는다.