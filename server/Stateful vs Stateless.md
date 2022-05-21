# Stateful /Stateless 란?

* Stateful: server side에 client와 server의 동작 및 상태 정보를 저장하는 형태로 세션에 의존하여 server의 응답이 달라지게된다.

예시) 클라이언트 쪽에서 TCP의 형태로 데이터를 전송할 경우 server side에 클라이언트의 세션정보를 저장시킨다.

* Stateless: server side에 client와 server의 동작 및 상태정보를 저장하지않고 server와 client의 세션이 독립적인 상태이다.

예시) UDP 통신