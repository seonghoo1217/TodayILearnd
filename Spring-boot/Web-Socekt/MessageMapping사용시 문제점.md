# MessageMapping 어노테이션 사용시 문제점

* STOMP를 이용한 웹소켓 프로젝트 사용시 문제점에 관하여 적어놓았다.

## 문제점 발생원인
기존의 DTO를 통해서만 사용해왔던 채팅방의 구조를 Entity화 시켜 DB를 구축시킬 때 구조 리팩터링을 거치던 도중 StompChatController 라는 
채팅의 MessageMapping을 관련하는 Controller에서 방의 번호를 받기위해 @PathVariable 어노테이션을 사용함.

## 문제 부분 코드

* StompChatController 부분

```java
@MessageMapping("/chat/enter/{roomId}")
    public void enter(@PathVariable("roomId") String roomId, ChatMessage message) {
        message.setMessage(message.getWriter() + "님이 채팅방에 참여하였습니다.");
        chatService.saveChat(message);
        log.info("stompRoomId={}",roomId);
        template.convertAndSend("/sub/chat/room/" + roomId,message);
    }
```

* 클라이언트 코드 부분

```javascript
          stomp.send('/pub/chat/enter/'+roomId, {}, JSON.stringify({roomId: roomId, writer: username}))
```

위 코드는 유저가 채팅방에 입장하였을 때 환영문구를 적어주기 위한 코드였다. 클라이언트 코드에 있는 roomId에는 roomId=[[${room.roomCode}]]라는 값이 들어가있다.
그렇기 때문에 나는 평소 컨트롤러에서 사용하듯이 PathVariable 어노테이션을 이용하면 값이 받아질 줄 알았다.

## 문제

위의 MessageMapping부분을 보면 알 수 있듯 roomId를 콘솔에 출력해보았다. 하지만 값은 /chat/enter/roomId로 보낸 값인 
JSON형식의 데이터가 String 형태로 변환되어 roomId에 들어가게 되었다.@PathVariable 사용시 roomId는 123 writer는 abc라고 가정하였을 때
MessageMapping부분의 log에는 ({roomId:123,writer:abc})가 들어가버리게 되었다.

## 문제 해결 방법

```java
@MessageMapping("/chat/enter/{roomId}")
public void enter(@DestinationVariable String roomId, ChatMessage message) {
        message.setMessage(message.getWriter() + "님이 채팅방에 참여하였습니다.");
        chatService.saveChat(message);
        log.info("stomproomId={}",roomId);
        template.convertAndSend("/sub/chat/room/" + roomId,message);
        }
```

=> 기존@PathVariable부분을 @DestinationVariable으로 바꾸어주면된다. 그럼 기존 컨트롤러에서 PathVariable 어노테이션을 사용한거처럼 이용가능하다.

## @PathVariable VS @DestinationVariable 차이점

둘다@Target(ElementType.PARAMETER),@Retention(RetentionPolicy.RUNTIME),@Documented를 상속받고 형식도 value 값을 String으로 선언하여 받는것은 똑같지만
@PathVariable은 RequestMapping URI에서 넘어오는 모든 값들을 받는것이고 @DestinationVariable은 오직 MessageMapping에서 만 사용된다.