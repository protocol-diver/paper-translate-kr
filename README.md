# paper-translate-kr
분산시스템, 통신 및 프로토콜에 관련된 RFC, 논문의 번역본을 보관하는 저장소입니다. <br>
[문서 규격](https://github.com/protocol-diver/paper-translate-kr#%EB%AC%B8%EC%84%9C-%EA%B7%9C%EA%B2%A9)을 참고하세요.

# Index
1. [목록](https://github.com/protocol-diver/paper-translate-kr/blob/main/README.md#%EB%AA%A9%EB%A1%9D)
   - [Network](https://github.com/protocol-diver/paper-translate-kr#network)
   - [Protocol](https://github.com/protocol-diver/paper-translate-kr#protocol)
2. [문서 규격](https://github.com/protocol-diver/paper-translate-kr#%EB%AC%B8%EC%84%9C-%EA%B7%9C%EA%B2%A9)
3. [기여](https://github.com/protocol-diver/paper-translate-kr#%EA%B8%B0%EC%97%AC)

# 목록

### Network
   - [x] [RFC894](https://github.com/protocol-diver/rfc-translate-kr/blob/main/kr/rfc894.txt) - A Standard for the Transmission of IP Datagrams over Ethernet Networks (이더넷 네트워크를 통한 IP 데이터그램 전송 표준)
   - [x] [RFC1918](https://github.com/protocol-diver/rfc-translate-kr/blob/main/kr/rfc1918.txt) - Address Allocation for Private Internets (사설 네트워크를 위한 주소 할당)
   - [x] [RFC5128](https://github.com/protocol-diver/rfc-translate-kr/blob/main/kr/rfc5128.txt) - State of Peer-to-Peer (P2P) Communication across Network Address Translators (NATs)

### Protocol
   - [ ] [RFC793](https://github.com/protocol-diver/rfc-translate-kr/blob/main/kr/rfc793.txt) - Transmission Control Protocol
   - [ ] [RFC3550](https://github.com/protocol-diver/rfc-translate-kr/blob/main/kr/rfc3550.txt) - RTP: A Transport Protocol for Real-Time Applications
   - [x] [RFC6886](https://github.com/protocol-diver/rfc-translate-kr/blob/main/kr/rfc6886.txt) - NAT Port Mapping Protocol (NAT-PMP)
   - [ ] [RFC9293]() - Transmission Control Protocol
   - [x] [devp2p/eth](https://github.com/protocol-diver/paper-translate-kr/blob/main/kr/devp2p/caps/eth.md) - Etheruem devp2p eth procotol
   - [ ] [devp2p/rlpx](https://github.com/protocol-diver/paper-translate-kr/blob/main/kr/devp2p/rlpx.md) - Etheruem devp2p rlpx procotol

# 문서 규격
한/영 변환 Github Page 운영에 대비하여 원본과 번역본 둘 다 저장합니다.

### 페이지 관련 내용은 삭제하여 커밋합니다(번역하는 과정에서 원본의 페이징이 틀어질 것 같아 미리 제거).

<b>> 목차</b><br>
기존)
```
   Table of Contents

   1. Introduction ....................................................3
      1.1. Transition to Port Control Protocol ........................4
   2. Conventions and Terminology Used in This Document ...............5
```
요구)
```
   Table of Contents

   1. Introduction
      1.1. Transition to Port Control Protocol
   2. Conventions and Terminology Used in This Document
```

<b>> 페이지</b><br>
```
   9.3.  Efficiency

   In addition to low-cost home gateways, many of the clients will also
   be similarly constrained low-cost devices with limited RAM resources.

   When implementing a NAT-PMP client on a constrained device, it's
   beneficial to have well-defined bounds on RAM requirements that are
   fixed and known in advance.  For example, when requesting the
   gateway's external IPv4 address, a NAT-PMP client on Ethernet knows





Cheshire & Krochmal           Informational                    [Page 27]

RFC 6886                         NAT-PMP                      April 2013


   that to receive the reply it will require 14 bytes for the Ethernet
   header, 20 bytes for the IPv4 header, 8 bytes for the UDP header, and
   12 bytes for the NAT-PMP payload, making a total of 54 bytes.
```
요구)
```
   9.3.  Efficiency

   In addition to low-cost home gateways, many of the clients will also
   be similarly constrained low-cost devices with limited RAM resources.

   When implementing a NAT-PMP client on a constrained device, it's
   beneficial to have well-defined bounds on RAM requirements that are
   fixed and known in advance.  For example, when requesting the
   gateway's external IPv4 address, a NAT-PMP client on Ethernet knows
   that to receive the reply it will require 14 bytes for the Ethernet
   header, 20 bytes for the IPv4 header, 8 bytes for the UDP header, and
   12 bytes for the NAT-PMP payload, making a total of 54 bytes.
```


### RFC 2119에 따른 ("MUST", "SHOULD"...) 표현은 아래와 같이 대체합니다.
기존)
```
   This protocol SHOULD only be used when the client determines that its
   primary IPv4 address is in one of the private IPv4 address ranges
   defined in "Address Allocation for Private Internets" [RFC1918].
```
요구)
```
   이 프로토콜은 SHOULD:{클라이언트가 기본 IPv4 주소가 "Address Allocation for
   Private Internets"[RFC1918]에 정의된 개인 IPv4 주소 범위 중 하나에 있다고 판단하는
   경우에만 사용}해야 합니다 [RFC1918].
```
# 기여
1. 작업을 진행하기 전, 페이지 관련 내용을 삭제한 뒤 `/en`, `/kr` 디렉토리에 문서를 먼저 저장합니다(작업이 진행중임을 파악하기 위함).
2. branch를 만들어 번역을 진행하되, 적절한 크기로 commit을 나눕니다(e.g. section 1, section2 ...). commit의 크기를 작게 유지하면 merge 전 검수할 때 원활합니다. 

