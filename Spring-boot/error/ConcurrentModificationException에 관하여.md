#ConcurrentModificationException에 관하여

## 에러 발생이유

```java
        List<ChatMessage> allMessage = chatRepository.findAll();
        ChatRoom chatRoom = chatRoomRepository.findByRoomCode(roomCode).get();
-        for (ChatMessage chatMessage:allMessage){
            if (chatMessage.getChatRoom().getId()!=chatRoom.getId()){
                allMessage.remove(chatMessage);
            }
        }
```

현재 진행중인 웹 소켓을 이용한 채팅 프로그램 프로젝트에서 채팅방 입장시 모든 메시지를 불러와 해당 방에 해당되는 메시지만 추리는 과정에서
위의 코드처럼 findAll()로 찾은 후 해당방의 메시지가 아닌것을 List에서 remove하려했다. 그렇지만 ConcurrentModificationException에러가 나게 되었고 
그에 관하여 왜 이러한 에러가 발생하는지와 해결방법을 기술하도록 하겠다.


## 이유?

ConcurrentModificationException은 단순히 동시성 문제 일줄 알았다. 서로 다른 쓰레드들이 간섭하여서 일어나는 문제가 되는구나 싶었지만 로컬변수에도 문제가 일어나는것을 보니
동시성문제는 아니었다.그렇다면 뭐가 문제였을까하고 찾아보니 Collection을 상속받는 List,Set 등에서 Loop중 데이터 조작이 발생하는 경우 문제가 되어 일어나는 에러라고 했다.
만약 문제가 일어날경우 IndexOutOfBoundsException을 예상했지만 너무나 예상외의 에러였기에 구조를 파헤쳐보았고 단순 for 구조의 loop가아닌
Enhanced for loop는 Iterator 방식으로 List를 순회해야 하는 구조이기 때문에 에러가 났던것이다.

## 해결방법

```java
        List<ChatMessage> allMessage = chatRepository.findAll();
        ChatRoom chatRoom = chatRoomRepository.findByRoomCode(roomCode).get();
        for (Iterator<ChatMessage> itr=allMessage.iterator();itr.hasNext();){
            ChatMessage chatMessage=itr.next();
            if (chatMessage.getChatRoom().getId()!=chatRoom.getId()){
                itr.remove();
            }
        }
```

오브젝트 자체를 iterator 시키는게 아니라 Iterator를 이용하여 이런식으로 사용하여서 Iterator자체를 지워버리면 사용가능하다

이 방법이 아니면

```java
List<ChatMessage> allMessage = chatRepository.findAll();
        ChatRoom chatRoom = chatRoomRepository.findByRoomCode(roomCode).get();
        /*List<ChatMessage> chatMessages = allMessage.stream().filter(i ->
                i.getChatRoom().getId() == chatRoom.getId()).collect(Collectors.toList());
*/
        for (Iterator<ChatMessage> itr=allMessage.iterator();itr.hasNext();){
            ChatMessage chatMessage=itr.next();
            if (chatMessage.getChatRoom().getId()!=chatRoom.getId()){
                itr.remove();
            }
        }
        return allMessage;
    }
```
이렇게 stream을 통하여 filter 조건문에 걸린값들을 collect(toList)시켜서 List값으로 반환받은후 이값을 return하여도 무방하다.