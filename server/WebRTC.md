# WebRTC

- 엔드 포인트 간의 실시간 스트리밍을 가능하게 해주는 통신 프로토콜이다.
- 초기 시그널링 서버에서 두 엔드 포인트의 정보가 교환되고, 교환된 정보를 바탕으로 P2P 연결을 맺게 된다.

# WebRTC에는 어떤 통신 종류가 있는가?

- P2P(Peer-to-Peer)
- SFU(Selective Forwarding Unit)
- MCU(Multipoint Control Unit)

# P2P

## ICE(Interactive Connectivity Establishment)

- 두 peer간 데이터 송수신시 최적의 경로를 찾아주는 프레임워크
- 두 Peer간 연결 테스트를 위해 SDP를 이용해 미디어 패킷을 보내 연결 가능한지 확인 함

## **ICE Candidate**

- STUN, TURN 서버를 이용해 얻어낸 IP주소, 프로토콜, 포트의 조합으로 구성된 네트워크 주소들
    - private IP , 포트번호
    - public IP, 포트번호 : STUN, TURN에서 가져온다.
    - TURN 서버의 IP, 포트번호 : TURN에서 가져온다.

ICE를 이용해 P2P 통신할수 있는 주소 후보(ICE Candidate)들을 찾고, 정보들을 통신가능케 하는 기술이 **SDP**

### **요약**

NAT를 하게 되면 라우터에만 공인 IP가 부여되고, 내부에 있는 컴퓨터에는 사설 IP가 부여된다. 이처럼 컴퓨터에 공인 IP가 할당되지 않은 경우도 있고, 라우터의 방화벽에서 외부 접근을 차단할 수도 있다.

따라서 두 피어가 연결될 방법을 찾아야 한다. candidate는 두 피어가 연결될 수 있는 IP, port, protocol의 후보를 의미한다. ICE 프레임워크는 두 피어의 candidate 중에서 최적의 연결 경로를 찾아준다.

## STUN / TURN Server

### **STUN**

- STUN 서버는 단말의 공인 IP/port를 찾고, 라우터가 연결을 제한하는지 확인하고, 제한하지 않는다면 STUN 서버로부터 받은 공인 주소를 교환하면 P2P 연결이 이루어진다.
- 라우터의 방화벽 정책에 의해 STUN 서버로부터의 주소 연결이 불가능한 경우가 있다. 이 경우 TURN 서버 사용

### TURN

- 피어는 TURN 서버와 연결을 맺고, 이 중계 서버를 통해 데이터를 주고받는다. 중간에 서버를 거치기 때문에 P2P 연결이라고 할 수 없고 오버헤드가 크다. 따라서 STUN 서버를 통해 직접 연결을 맺을 수 없는 불가피한 경우에만 사용한다.
- 우회하는 방법이다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/db74ba2d-24e4-4d4b-ae5c-07d99cd33ca1/b4a6bc7f-b7d9-4474-b1c4-be58328a712f/Untitled.png)

# 서버의 종류

- signaling
- SFU(**Selective Forwarding Unit)**
- MCU**(Multi-point Control Unit)**

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/db74ba2d-24e4-4d4b-ae5c-07d99cd33ca1/60d5f569-e6b2-47db-ab6e-fc629e58af75/Untitled.png)

## Signalling

- **특징**
    - **peer 간의 offer, answer라는 session 정보 signal만을 중계**한다.
    - **처음 WebRTC가 peer간의 정보를 중계할 때만 서버에 부하가 발생**한다.
    - peer간 연결이 **완료된 후에는 서버에 별도의 부하가 없다.**
    - **1:1 연결에 적합**하다.
- **장점**
    - 서버의 부하가 적게 든다
    - peer간의 통신이기에 속도가 제일 빠르고 , 실시간 성이 보장된다.
- **단점**
    - N:N 혹은 N:M 연결에서 **클라이언트의 과부하가 급격하게 증가**

## SFU

- 특징
    - **종단 간 미디어 트래픽을 중계하는 중앙 서버 방식**
    - **서버와 클라이언트 간의 peer를 연결**
    - 1:1, 1:N, N:N 혹은 N:M 등 모든 연결 형식에서 클라이언트는 연결된 **모든 사용자에게 데이터를 보낼 필요없이 서버에게만 자신의 영상 데이터를 보내면 된다.**
    - 모든 커넥션은 종료전까지 보장되어야한다.
- 장점
    - 클라이언트의 부하를 줄일 수 있다.
- 단점
    - Signaling 서버보다 **서버 비용이 증가**
    - **대규모 N:M 구조에서는 여전히 클라이언트가 많은 부하를 감당**

## MCU

- **특징**
    - **다수의 송출 미디어를 중앙 서버에서 혼합(muxing) 또는 가공(transcoding)하여 수신측으로 전달하는 중앙 서버 방식**
    - 클라이언트 peer간 연결이 아닌, **서버와 클라이언트 간의 peer를 연결**
    - **서버에게서 하나의 peer로 데이터를 받으면 된다.**
    - **중앙 서버의 높은 컴퓨팅 파워가 요구**
- **장점**
    - **클라이언트의 부하가 현저히 줄어든다.**
    - **N:M 구조에 사용 가능**
- **단점**
    - 실시간성이 저해된다.