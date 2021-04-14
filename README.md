# 라인 VOIP 사랑해요 
# WebRTC_Mesh
구글의 VOIP 회사 인수를 통해 만들어진 기술로써 기본적으로 VOIP가 이용하는 프로토콜을 사용한다.

## 목적

 이전에 제작한 CatchMind 프로젝트에서 어려웠던점을 해결하기 위해
 
 **전공지식 공부 및 오픈소스를 사용하지 않고 직접 구현하여 완벽한 이해를 목표로 한다**
 
+ 어려웠던점, 힘들었던점
  + LTE가 Offer, Wifi(Full Cone Nat)가 Answer일시 연결 가능하나 반대일때 불가능한 이유 이해불가
  + 위의 문제를 Turn 서버를 통해 해결할 수 있음을 확인해 Symmetric Nat가 원인임을 확인했으나 정확한 이유 이해불가
  + Nat의 네가지 종류에 대한 명확한 설명이 부족해 RFC를 하나하나 번역해 이해하는 어려움

+ 해결방법 및 이해하기 위한 노력
  +  모든 문제해결의 키는 전공지식이므로 관련 전공지식 전부 학습
  +  RTP, RTCP, NAT 4종류, ICE FrameWork, SDP, Turn/Stun 작동원리, Nat Traversal, Offer/Answer 구조 등 모든 이론 공부
  +  오픈소스가 아닌 실제 API를 통해 직접 구현
  +  연결 실패 재전송 로직 구성

---
## 프로젝트 수행 내용

### 1. WebRTC오픈소스 걷어내고 리펙토링

기존 NodeJs기반 WebRTC 웹어플리케이션  "CatchMind" 에서 WebRTC 구현을 위해 사용한 PeerJS와 시그널링 서버인 PeerServerJS를 걷어내고 
Native WebRTC API를 직접 사용해 Mesh 구조의 P2P 프로젝트 제작.

### 2. Offer/Answer 구조로 ICE Framework, SDP를 이용한 시그널링 서버 제작(SOCKET IO)

단말간의 통신경로 확보를 위한 방법 중 하나인 ICE Framework의 이해(Turn/Stun)와 Nat TraverSal 를 바탕으로 Offer/Answer 구조로 SDP를 교환하는 코드를 작성하여 학습한 내용을 확인한다.

### 3. Nat의 이해 + 실패시 재전송

Nat 는 단순히 외부IP/Port 와 내부 IP/Port를 변환해주는게 아닌
+ Nat 종류  
  + Full Cone NAT
  + Restricted Cone NAT
  + Port Restricted Cone NAT
  + Symmetric NAT
  

가 있음을 이해하고 이를 바탕으로 **양쪽 피어가 모두 Symmetric NAT가 아닐때** Offer와 Answer를 바꿔 전송하도록 코드를 작성한다.

#### 양쪽 피어가 모두 Symmetric NAT 일때는 Turn 서버가 필수.

---

# 프로그램 구현을 위한 배경지식(WebRTC 연결성 및 NAT 통과 기법) 상세설명 


### 0. WebRTC는 Use Of RTP 이다
실시간 전송 프로토콜 (RTP) 는 IP 네트워크 상에서 오디오와 비디오를 전달하기 위한 통신 프로토콜.
WebRTC API를 사용해 구현한 시그널링 서버로 두 피어가 시그널링을 한 후 P2P 통신을 한다

### 1. 시그널링 서버란?
p2p 연결을 위해 상호 간 ip/port 교환, SDP교환 등의 연결 설정을 교환하고 동의하기 위해 필요한 서버.
ICE 프레임워크를 이용한다.

### 2. SDP란?

IETF의 RFC 4566 으로 스트리밍 미디어의 초기화 인수를 기술하기 위한 포맷.
SDP(Session Description Protocol)는 WebRTC에서 스트리밍 미디어의 해상도나 형식, 코덱 등의 멀티미디어 컨텐츠의 초기 인수를 설명하기 위해 채택한 프로토콜. 
(ex: 웹캠 비디오의 해상도를 보낼 수 있고, 오디오 전송 또는 수신 여부)

### 3. ICE Framework란?
NAT 통과 기법을 위한 네트워크 표준.
ICE는 두 개의 단말이 P2P 연결을 가능하게 하도록 최적의 경로를 찾아주는 프레임워크.

** ICE 프레임워크가 STUN, 또는 TURN 서버를 이용해 상대방과 연결 가능한 후보(Candidate) 를 찾는다. **

### 4. Hole Puncing

P2P 통신을 목적으로 Routing Table 을 작성하기 위해 사전에 상대방과 패킷을 주고받게 하여 각자의 공유기에 Routing Table을 작성하는 것을 홀 펀칭이라고 합니다. Stun 서버를 이용한다.

### 5. ICE Candidate 란?

ICE 즉, STUN, TURN 서버를 이용해서 획득했던 IP 주소와 프로토콜, 포트의 조합으로 구성된 연결 가능한 네트워크 주소들을 후보(Candidate)로
만든 것. 이 과정을 Finding Candidate or Gathering Candidate라고 한다.

이렇게 후보들을 수집하면 일반적으로 3개의 주소를 얻게 됩니다.

자신의 사설 IP와 포트 넘버
자신의 공인 IP와 포트 넘버 (STUN, TURN 서버로부터 획득 가능)
TURN 서버의 IP와 포트 넘버 (TURN 서버로부터 획득 가능)


### 6. WebRTC오픈소스 걷어내고 리펙토링

PeerJS/ PeerServerJS를 걷어내고 WebRTC API를 사용한다.
+ 대표 WebRTC API
  + new RTCPeerConnection(config);
  + new RTCSessionDescription(SDP);
  + new RTCIceCandidate(candidate)


