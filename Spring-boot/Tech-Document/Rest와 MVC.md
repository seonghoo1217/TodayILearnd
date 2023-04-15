# Rest와 MVC
Rest와 MVC는 상반되는 개념일까? 실재로는 그렇지않다.

Rest는 Http 프로토콜에서 동작되는 하나의 아키텍처 스타일이다.

하지만, MVC는 아키텍처 스타일이 아닌 소프트웨어의 디자인 패턴 중 하나이며, 가장 큰 핵심을
소프트웨어의 계층 분리를 통해 세가지 역할로 나누어, 유지보수성과 확장성을 높일 수 있다.

## Spring에서의 MVC와 Rest

Spring에서는 반환값이 달라지게 된다.

기본적으로 MVC에서는 DispatcherServlet을 통해 View를 클라이언트에게 전달하는 반면,
Rest의 경우 MessageConveter를 이용해 Json 또는 Text 형태를 리턴하게 된다.

