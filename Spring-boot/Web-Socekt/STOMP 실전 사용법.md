# STOMP 실전 사용법

* 이 글을 정리하게 된 이유는 너무나도 많은 STOMP 및 웹 소켓 정리글들이 있지만 나처럼 코드를 치며 이해를 원하는 사람들이 원할만한 블로그가 별로 없었기에 TIL에 정리 후 블로그에 서술 예정이다.

STOMP 사용은 크게 3가지 구조를 요한다.
1. WebSocketMessageBrokerConfigurer 를 구현하는 Config 파일
2. @MessageMapping을 관리하는 Controller 
3. 클라이언트 JS 코드

이를 순차적으로 살펴보자

## WebSocketMessageBrokerConfigurer 를 구현하는 Config 파일

STOMP를 사용시에 필수적으로 Override하여 구현해야 하는 메서드가 두개 있다. 바로 configureMessageBroker 와 registerStompEndpoints 이다.

우선 configureMessageBroker를 코드적으로 먼저 살펴보자

```java
@Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/sub");
        config.setApplicationDestinationPrefixes("/pub");
    }
```
configureMessageBroker 에는 기본적으로 MessageBrokerRegistry 타입의 파라메터가 들어와야하고 이를 사용하여 enableSimpleBroker와 setApplicationDestinationPrefixes를 정의할 수 있다.
이 두 메서드를 알기전 Stomp의 구독이라는 개념과 퍼블리싱의 개념을 이해해야 한다.
이해를 위해 채팅 시스템을 예시로 들어보자 우리가 카카오톡의 이용할 때는 각 방의 대화 내용이 공유되지 않는다. 이를 기억하고 설명을 들어보자.
만약 당신은 채팅의 1번 방에 있다. 그럼 당신은 채팅방에 입장하였을시에 /sub/chat/room/1 에 구독이 되었을 것이다. 그리고 그 안에서 이루어지는 메세지 전송등은 1번방으로만 전송이 된다.
그렇기에 각방의 대화 내용이 공유되지 않을 수 있는 것이다.

그럼이제 enableSimpleBroker에는 구독의 헤더를 적어주고 setApplicationDestinationPrefixes 에는 전송(퍼블리싱) 경로의 헤더를 적어주면 된다.

두번째로 Override 하여 구현하는 registerStompEndpoints는 메서드 이름처럼 엔드포인트 즉 소켓연결 경로를 적어주면된다.
코드로 함께 살펴보자.

```java
@Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/stomp/chat").setAllowedOrigins("http://localhost:8080").withSockJS();
    }
```
이 메서드는 기본적으로 StompEndpointRegistry 타입의 파라메터를 가지고 addEndpoint,setAllowedOrigins,withSockJS() 등의 메서드를 가진다.
addEndpoint는 말그대로 웹소켓에 연결할 URI를 지정해주는 것으로 창구 역할을 해준다고 생각하면 이해가편하다.(이는 클라이언트 코드와 이어진다.)
setAllowedOrigins는 해당경로로의 소켓 통신만 허용하겠다는 것이다.
withSockJS는 웹소켓을 지원해주지않는 브라우저로 접속하였을 때 HTTP Polling이라는 과거의 기술을 자동으로 지원해주는 메서드이다.

## @MessageMapping을 관리하는 Controller

@MessageMapping은 스프링이 지원해주는 어노테이션으로 STOMP에서 아까 위에서 말한 구독과 퍼블리싱을 도와주는 기능이다.
이 단계는 코드로 보면 이해가 편하니 바로 코드를 살펴보자

```java
    @MessageMapping("/chat/message/{roomId}")
    public void message(@DestinationVariable String roomId, ChatMessage chatMessage){
        ChatRoom chatRoom = chatRoomRepository.findByRoomCode(roomId).get();
        chatMessage.setChatRoom(chatRoom);
        chatMessage.setChatTime(LocalDateTime.now());
        chatService.saveChat(chatMessage);
        template.convertAndSend("/sub/chat/room/" + roomId, chatMessage.toDto());
        }
```

당신이 클라이언트 코드에서 채팅을 보낼때의 경로가 만약 /pub/chat/message/roomID라면 해당 경로 컨트롤러에서 이를 받게되고
이를 다시 sub가 받을 수 있게 SimpMessagingTemplate를 사용하여 convertAndSend라는 메서드를 이용해 보내게된다. 
이때 파라메터로는 경로,payload 순이다.

### 클라이언트 JS 코드

우선 가장먼저해야할 자바스크립트 코드는 당연히 웹소켓을 연결 시켜주는 부분이다.
코드를 보며 진행해보자

```javascript
var sockJs = new SockJS("/stomp/chat", null, {transports: ["websocket", "xhr-streaming", "xhr-polling"]});
let stomp = Stomp.over(sockJs);
```

위에서 지정한 EndPoint를 파라메터에 넣어주고 이를 다시 Stomp에 연결시켜준다. 그러면 간편하게 stomp에 연결이 가능해진다.
뒤에 있는 파라메터들은 아까 위에서 언급하였듯이 SockJS가 웹 소켓을 지원해주지않는 브라우저라면 가장먼저 websocket으로 연결시도를 하고 안되면 streaming이라는 기술로 연결을 시도한다. 이것마저 안되면 polling 기술을 사용한다.

그후 저 stomp를 connect()해주면 이제 본격적인 stomp사용이 시작된다.

```javascript
stomp.connect()
```

그리고 connect()되어 있는동안 sub를 하여야 하기 때문에 connect function안에 subscribe를 지정해준다.

```javascript
stomp.subscribe("/sub/chat/room/" + roomId) //위에서 사용한 MessageMapping 예제에서 보내지는 값이 여기로 들어오게 된다.
```

데이터를 전송해야할 때는
```javascript
stomp.send('/pub/chat/room/'+roomId)
```

이렇게 보내주면된다. 위에서 보여준 MessageMapping예제의 컨트롤러로 값이 전달된다.

만약 더 이상 구독경로를 사용하지않는다면 아래코드를 실행시켜주면된다. 파라메터는 Object타입으로 메시지를 넣어주면된다.
```javascript
stomp.unsubscribe();
```

만약 Stomp연결을 끊어버리고 싶다면 아래 코드처럼 connect()함수의 반대인 disconnect()를 이용해주면된다.

```javascript
stomp.disconnect();
```
