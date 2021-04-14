# WebRTC_Mesh

## 목적

### 1. WebRTC오픈소스 걷어내고 리펙토링

기존 NodeJs기반 WebRTC 웹어플리케이션  "CatchMind" 에서 WebRTC 구현을 위해 사용한 PeerJS와 시그널링 서버인 PeerServerJS를 걷어내고 
Native WebRTC API를 직접 사용해 Mesh 구조의 P2P 프로젝트 제작.

### 2. ICE Framework 이해

단말간의 통신경로 확보를 위한 방법 중 하나인 ICE Framework의 이해(Turn/Stun)와 Nat TraverSal 를 바탕으로 코드를 작성하여 코드로 학습한 내용을 확인한다.

### 3. Nat의 이해 + 실패시 재전송

Nat 는 단순히 외부IP/Port 와 내부 IP/Port를 변환해주는게 아닌

+Full Cone NAT
+Restricted Cone NAT
+Port Restricted Cone NAT
+Symmetric NAT

가 있음을 이해하고 이를 바탕으로 **양쪽 피어가 모두 Symmetric NAT가 아닐때** Offer와 Answer를 바꿔 전송하도록 코드를 작성한다.

####양쪽 피어가 모두 Symmetric NAT 일때는 Turn 서버가 필수.



