# HTTP와 TCP/IP의 차이점

HTTP는 최상위 계층인 Application 계층(응용 프로그램 계층)에 동작한다.
개념적으로만 봤을 땐 HTTP는 TCP/IP 계층 위에서 동작한다.

HTTP를 까보면 TCP가 나오게 된다.

즉 TCP 기반으로 만들어진 프로토콜이 HTTP이다.

![OSI7](https://velog.velcdn.com/images%2Fcorone_hi%2Fpost%2F57b94a1c-4842-4e4b-b46a-f8fe732d8a48%2Fimage.png)


TCP는 byte Array 형태로 정보를 통신한다.
Http는 String 형태로 정보를 통신한다.

Http에는 헤더정보가 붙으며 TCP는 패킷단위이다.