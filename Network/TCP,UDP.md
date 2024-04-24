## 들어가기전에 짧게

**원래는 어떻게 통신을 했는가?**

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/db74ba2d-24e4-4d4b-ae5c-07d99cd33ca1/9357797b-cb56-4ba3-bba4-d7c2343423bc/Untitled.png)

위 사진과 같이 경로가 정해져있었음

그럼 어떤 문제가 발생하는가?

- 일단 유연하게 전달될 수 없고 혼잡돈가 높아질 가능성이 상당히 높 음
- 회선 교환 방식을 사용한다면 **이미 설정된 이동 경로는 할당 해제될 때까지 다른 호스트들이 사용할 수 없게 된다**
    - 말 그대로 독점하는 것이며, 실제로 데이터를 전송하고 있지 않아도 연결 설정이 끊어지지 않는 이상 다른 호스트들은 사용할 수 없다.
- **단절에 매우 취약**하다. 두 호스트간의 **연결이 끊어지면 이동 경로를 다시 연결**해줘야 하고, 그 사이 다른 호스트가 **차지해버리면 해당 경로가 릴리즈 될 때까지 기다려야**

이를 해결하기위해 생긴 개념이?

- **패킷 통신**

## 인터넷 프로토콜

- 인터넷을 통해 컴퓨터들은 정보를 통신하게 된다.
- 이때 서로 정보를 송/수신 하는 규약이 다르면 문제가 발생할 수 있다.
- 이를 표준 규격화한 것이 **프로토콜**
- 이 중 가장 유명하게 사용되는 것이 TCP/IP 프로토콜 슈트이며, 주의해야할점은 TCP와 IP는 별도의 레이어

## TCP / IP

- 패킷 통신 방식의 인터넷 프로토콜인 IP
    - 패킷의 전달 여부를 **보장하지 않고**, **전송 순서도 다를** 수 있음
- 전송 조절 프로토콜인 TCP
- TCP가 IP위에서 동작

### TCP/IP를 이용하겠다?

⇒ IP주소 체계를 이용하며, IP Routing을 이용하여 TCP의 특성을 이용하겠다.

즉, 송신자가 수신자에게 IP주소를 사용하여 데이터를 전달하고 해당 데이터의 신뢰성(전달, 순서보장)을 보장하는 프토콜을 이용하겠다.

## Transport Layer(OSI 4계층)

- 송신자와 수신자간의 논리적 연결을 지향
- 신뢰성 있는 연결을 유지할 수 있도록 도와준다.
- 데이터 수신양과 수신여부를 체크할 수 있다.
- TCP와 UDP가 대표

## Network Layer(OSI 3계층)

- IP(Internet Protocol)을 사용하여 경로와 목적지를 찾는 역할
    - 이를 Routing이라고한다.
- 대역이 다른 IP 또한 찾아 갈 수 있다.

## TCP 개요

- 근거리 통신망이나 인트라넷 상관없이 일련의 옥텟(8비트 단위)을 순서보장과 에러 보장없이 보내주는 것을 보장해주는 프로토콜이다.
- 전송계층에 위치
- Routing에 초점을 두기보단 다음 네가지에 대해 검증을 하기위한 프로토콜
    - 양쪽 단말의 통신준비를 체크
    - 데이터가 옳바르게 전달되었는지 체크
    - 데이터의 위변조를 검사
    - 오류 검증
- TCP가 실제로 데이터를 담는 구간을 `Segment(세그먼트)` 라고 정의한다.
- 신뢰 지향성 연결방법이다.

# TCP의 특징

### 1. Connection Oriented(연결지향)

- 엔드 포인트 2개 사이에 연결을 맺고(3-way-handshake) 데이터를 주고받음
- TCP 연결식별자는 두 EndPoint의 주소를 합친것
    - Local IP Address(송신자)
    - Local Port
    - Remote IP Address(수신자)
    - Remote Port

### 2. Bidirectional Byte System

- 양방향 통신과 함께 바이트 스트림을 사용

### 3. In-Order Delivery

- Sender와 Receiver가 데이터를 주고받는 구조
- 데이터의 순서가 보장되어야 함으로, 32bit 정수 자료형을 사용

### 4. Reliability through ACK

데이터를 송신하고 수신자로부터 ACK(데이터 받았음)를 받지 않으면, 송신자 TCP가 데이터를 재전송한다. 따라서 송신자 TCP는 수신자로부터 ACK를 받지 않은 데이터를 보관한다(buffer unacknowledged data).

### 5. Flow control

송신자는 수신자가 받을 수 있는 만큼 데이터를 전송한다. 수신자가 자신이 받을 수 있는 바이트 수 (사용하지 않은 버퍼 크기, receive window)를 송신자에게 전달한다. 송신자는 수신자 receive window가 허용하는 바이트 수만큼 데이터를 전송한다.

### 6. Congestion control

네트워크 정체를 방지하기 위해 receive window와 별도로 congestion window를 사용하는데 이는 네트워크에 유입되는 데이터양을 제한하기 위해서이다

## 3-way-handshake

총 3단계로 구성된 통신방법

`SYN` , `SYN/ACK` ,`ACK Flag` 로 구성되어있다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/db74ba2d-24e4-4d4b-ae5c-07d99cd33ca1/142d19d0-2f7d-48f9-8e16-76e8e810cb4c/Untitled.png)

TCP로 이루어지는 모든 통신은 반드시 3-way handshake를 통해 시작한다.

송신자가 보낼 준비가 되어 있는지를 미리 확인한 후 통신을 시작하여 데이터를 안전하게 보내기 위함

데이터를 받았을 때 잘 받았음을 알리는 'ACK'를 송신자에게 전달.

송신자는 이 'ACK'를 보고 수신자가 데이터를 잘 받았음을 확인하고 다음 데이터를 전달할 준비

# TCP의 오류 검출

## 검사합(체크섬)

TCP는 모든 세그먼트에 필수사항으로 16비트 검사합 필드를 기록해야 한다. 수신자는 검사합을 이용해 세그먼트가 손상되었는지 판단하고, 세그먼트가 손상되었다면 손상으로 판명된 세그먼트를 버린다.

검사합 계산을 위하여 의사헤더를 만들어서 추가한다. 왜 검사합을 계산할 때 의사헤더를 포함하는 것일까? **TCP 헤더만으로는 IP 헤더의 오류를 검출할 수 없기 때문이다.**

만약 IP주소가 잘못 되었거나 프로토콜이 잘못된 경우(예를 들어, 패킷이 TCP이 아니라 UDP로 전송되는 경우)에는 의사헤더의 값으로 오류를 검출할 수 있다

## 재전송

### 재전송 시간초과

세그먼트 발신자는 전송하는 각 세그먼트마다 재전송 시간초과(RTO; retransmission time-out) 타이머를 구동한다. 타이머가 만료되면 해당 세그먼트는 훼손되거나 손실된 것으로 간주하고 세그먼트를 재전송한다.

확인응답만을 포함하는 세그먼트에는 시간초과 타이머가 설정되지 않으며, 이러한 세그먼트는 재전송되지 않는다.

TCP에서 RTO값은 가변적이며 세그먼트의 왕복시간(RTT; round trip time)을 기반으로 업데이트된다.

### 손실 세그먼트 재전송

세그먼트는 전송 중 버려지거나, 오류 검출 등의 이유로 수신자에 의해서 버려질 수 있다. 이렇게 세그먼트가 손실되면 재전송하여야 한다.

수신자는 입력 버퍼에 손실된 세그먼트의 자리를 비워놓는다. 빈 공간이 다 채워지기 전까지는 버퍼의 내용을 어플리케이션 계층으로 전달하지 않는다.

세 번째 세그먼트에 대해서 확인응답을 전송하지 않기 때문에 송신 측에서 설정된 타이머는 만료가 될 것이다. 타이머가 만료되면 송신 TCP는 세 번째 세그먼트를 재전송할 것이다.

!https://velog.velcdn.com/images%2Fshroad1802%2Fpost%2F9a9d652d-cbbe-4752-835c-5e3c04dd4fc6%2Fimage.png

### 빠른 재전송

세 번째 세그먼트가 손실된 경우를 가정해보자. 수신측은 4,5,6번째 세그먼트를 각각 수신할 때마다 확인응답을 전송한다. 그러면 송신 측은 동일한 값을 갖는 세 개의 확인응답 즉, 3개의 중복응답을 수신한다. 중복응답을 3개 수신하면 비록 세 번째 세그먼트에 대한 타이머가 만료되지는 않았지만 빠른 재전송을 위한 규칙에 의거하여 송신 측은 모든 확인응답이 기대하는 세그먼트인 세 번째 세그먼트를 즉시 재전송한다.

!https://velog.velcdn.com/images%2Fshroad1802%2Fpost%2F6107549d-ed57-482b-8458-e2676f0fc906%2Fimage.png

### 지연 세그먼트

TCP는 비연결 지향 프로토콜인 IP 위에서 전송되기 때문에 TCP 세그먼트들은 서로 다른 지연을 가진 서로 다른 경로를 거쳐서 목적지에 도달한다. 따라서 TCP 세그먼트도 지연될 수 있다.

수신 측은 지연 세그먼트를 손실이나 훼손 세그먼트와 동일한 방법으로 처리한다. 또한 지연 세그먼트는 중복 세그먼트로서 재전송된 이후에 도착할 수도 있다.

### 중복 세그먼트

위의 지연 세그먼트의 경우 수신 측에 의해서 손실로 처리되는 경우에 늦게 도착한 세그먼트는 중복 세그먼트가 된다. 중복 세그먼트가 도착한 경우 중복 세그먼트는 수신자가 기대하는 순서 번호 값보다 적은 순서 번호 값을 가지기 때문에 수신자 TCP는 중복 세그먼트를 버린다.

### ACK의 손실 자동 교정

확인응답이 손실되더라도 다음 확인응답에 의해서 자동으로 대치될 수 있다. ACK는 이전 확인응답을 포함하는 의미이기 때문에 이전의 세그먼트들이 모두 정상적으로 전송되었다는 의미가 된다.

!https://velog.velcdn.com/images%2Fshroad1802%2Fpost%2F8fcf9592-74eb-4c1e-b41f-17d113b9219e%2Fimage.png

### 확인응답의 손실로 인해 발생하는 교착상태(deadlock)

수신자가 rwnd값이 0으로 설정된 확인응답을 전송함으로써 송신자로 하여금 일시적으로 윈도우를 폐쇄한 경우 발생할 수 있다.

윈도우를 폐쇄한 이후 수신 측이 이러한 제한을 없애고자 하지만 전송할 데이터가 없는 경우, ACK 세그먼트를 전송한다. 하지만 이 ACK가 손실된 경우에는 서로 기다리는 교착상태가 된다. 이와 같은 교착상태를 막기 위해 영속 타이머(persistence timer)가 설계되었다.

# **UDP(User Datagram Protocol)**

- *데이터를 데이터그램 단위로 처리하는 프로토콜*
- TCP와 달리 UDP는 비연결형 프로토콜
- 즉, 연결을 위해 할당되는 논리적인 경로가 없는데,그렇기 때문에 각각의 패킷은 다른 경로로 전송되고, 각각의 패킷은 독립적인 관계를 지니게 되는데 이렇게 데이터를 서로다른 경로로 독립적으로 처리하게 되고, 이러한 프로토콜을 UDP라고함

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/db74ba2d-24e4-4d4b-ae5c-07d99cd33ca1/f93f7ff4-e74d-4351-9788-3ec1ad38c221/Untitled.png)

## UDP 특징

- 비연결형 서비스로 데이터그램 방식을 제공
- 정보를 주고 받을 때 신호 절차가 없음
- Checksum을 활용한 최소한의 에러만 검출
- 신뢰성이 낮다
- TCP보다 속도가 빠르다

## 차이

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/db74ba2d-24e4-4d4b-ae5c-07d99cd33ca1/6ef6e149-14be-4630-9b71-ce9d2edf7382/Untitled.png)