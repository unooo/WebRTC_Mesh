##### 라인 VOIP 직무를 향한 진심을 담다....! 

# WebRTC_Mesh 프로젝트 목적

### 1. 구글이 만든 VOIP 기반 WebRTC 기술을 사용해 Mesh 구조의 P2P를 구현/이해한다.
### 2. VOIP의 프로토콜을 재사용한 WebRTC를 통해 VOIP이해를 위한 초석을 다진다.
### 3. 이전에 제작한 CatchMind 프로젝트에서 어려웠던점/을 해결한다
 
+ 어려웠던점, 힘들었던점
  + ***LTE가 Offer, Wifi(Full Cone Nat)가 Answer일시 연결 가능하나 반대일때 불가능한 이유 이해불가***
  + 위의 문제를 Turn 해결해 Symmetric Nat가 원인임을 확인했으나 정확한 원리 이해불가
  + Nat의 네가지 종류에 대한 명확한 설명이 부족해 RFC를 하나하나 번역
---

 **이 3가지 목적을 위해 심화 전공지식 공부 및 오픈소스를 사용하지 않고 직접 구현하여 완벽한 이해를 목표로 한다**
 
+ 해결방법 및 이해하기 위한 노력
  +  **한글 자료가 매우 부족한 방면 영문 및 공식문서 자료가 매우 방대한 양이지만 정리가 잘되어 있어 집중적으로 공부**
  -https://hpbn.co/webrtc/
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

### 1. Web Application API (MediaSource) 플러그인 없이 브라우져로 음성과 영상사용 + 노이즈, 에코, 패킷로스 처리까지 해준다

![tp4](https://user-images.githubusercontent.com/30948477/114663265-ff2e0b80-9d34-11eb-90ec-d53f3bf30590.JPG)

```javascript
  navigator.getUserMedia (constraints, gotStream, logError); 
```

### 2. 코덱 표준은 오픈소스를 이용해 라이센스Fee를 내지 않는다
![tp5](https://user-images.githubusercontent.com/30948477/114663694-ad39b580-9d35-11eb-8932-0e81116695ad.JPG)

### 3. RTCPeerConnection API의 대략적인 구조
![tp6](https://user-images.githubusercontent.com/30948477/114663872-ec680680-9d35-11eb-84b8-4ec869ab531d.JPG)

### 4. WebRTC는 Use Of RTP 이다.(https가 강제되어 정확히는 SRTP)
실시간 전송 프로토콜 (RTP) 는 IP 네트워크 상에서 오디오와 비디오를 전달하기 위한 통신 프로토콜.
WebRTC API를 사용해 구현한 시그널링 서버로 두 피어가 시그널링을 한 후 P2P 통신을 한다
![tp1](https://user-images.githubusercontent.com/30948477/114661840-a2c9ec80-9d32-11eb-9384-c2c1294fded7.JPG)
![tp8](https://user-images.githubusercontent.com/30948477/114665008-7369ae80-9d37-11eb-908c-b08cb9d8ce31.JPG)

### 5. 시그널링 서버란?
p2p 연결을 위해 상호 간 ip/port 교환, SDP교환 등의 연결 설정을 교환하고 동의하기 위해 필요한 서버.
ICE 프레임워크를 이용한다.
![tp2](https://user-images.githubusercontent.com/30948477/114661957-d86ed580-9d32-11eb-8dfd-6eab7aecc068.png)

### 6. offer / Answer 구조란?

간단하게 전화를 걸면 Offer, 전화를 받으면 Answer 이다.
![oa](https://user-images.githubusercontent.com/30948477/114662307-6a76de00-9d33-11eb-816a-ea56c3fde86e.jpeg)

### 7. SDP란?

IETF의 RFC 4566 으로 스트리밍 미디어의 초기화 인수를 기술하기 위한 포맷. Offer / Answer 구조로 주고 받는다.
SDP(Session Description Protocol)는 WebRTC에서 스트리밍 미디어의 해상도나 형식, 코덱 등의 멀티미디어 컨텐츠의 초기 인수를 설명하기 위해 채택한 프로토콜. 
(ex: 웹캠 비디오의 해상도를 보낼 수 있고, 오디오 전송 또는 수신 여부)

![tp3](https://user-images.githubusercontent.com/30948477/114662071-0eac5500-9d33-11eb-9323-f4bfb41209af.JPG)

### 8. ICE Framework란?
NAT 통과 기법을 위한 네트워크 표준.
ICE는 두 개의 단말이 P2P 연결을 가능하게 하도록 최적의 경로를 찾아주는 프레임워크.
** ICE 프레임워크가 STUN, 또는 TURN 서버를 이용해 상대방과 연결 가능한 후보(Candidate) 를 찾는다. **

### 9. Hole Puncing

P2P 통신을 목적으로 Routing Table 을 작성하기 위해 사전에 상대방과 패킷을 주고받게 하여 각자의 공유기에 Routing Table을 작성하는 것을 홀 펀칭이라고 합니다. Stun 서버를 이용한다.

### 10. ICE Candidate 란?

ICE 즉, STUN, TURN 서버를 이용해서 획득했던 IP 주소와 프로토콜, 포트의 조합으로 구성된 연결 가능한 네트워크 주소들을 후보(Candidate)로
만든 것. 이 과정을 Finding Candidate or Gathering Candidate라고 한다.

이렇게 후보들을 수집하면 일반적으로 3개의 주소를 얻게 됩니다.

자신의 사설 IP와 포트 넘버
자신의 공인 IP와 포트 넘버 (STUN, TURN 서버로부터 획득 가능)
TURN 서버의 IP와 포트 넘버 (TURN 서버로부터 획득 가능)

### 11. TLS와 유사한 DTLS 
사실상 같은 동작방식이지만 DTLS는 UDP 기반이므로  핸드 셰이크 시퀀스 에서만 
***"mini-TCP" *** 를 구현

-> 핸드 셰이크 레코드에 대한 명시 적 조각 오프셋 및 시퀀스 번호를 추가

->양측은 예상 간격 내에 응답이 수신되지 않으면 간단한 타이머를 사용하여 핸드 셰이크 레코드를 재전송해 패킷손실을 처리

![tp7](https://user-images.githubusercontent.com/30948477/114664684-0524ec00-9d37-11eb-8d0e-a2e5c05baea4.JPG)

### 12. WebRTC오픈소스 걷어내고 리펙토링

PeerJS/ PeerServerJS를 걷어내고 WebRTC API를 사용한다.
+ 대표 WebRTC API
  + new RTCPeerConnection(config);
  + new RTCSessionDescription(SDP);
  + new RTCIceCandidate(candidate)

---
## 해결한 문제 
![tp9](https://user-images.githubusercontent.com/30948477/114671172-c004b800-9d3e-11eb-9228-38e1195b83d7.png)

## 해결방법
1. Turn 서버 사용 : p2p가 아니게 되므로 서버 부하 증가. 아래의 케이스에서만 turn 서버 사용
![tp11](https://user-images.githubusercontent.com/30948477/114671605-3c979680-9d3f-11eb-870e-fd525f9c6abc.JPG)

***2. Answer가 Offer로 재전송 로직 ***


참조
1. https://brunch.co.kr/@linecard/141
2. https://hpbn.co/webrtc/

