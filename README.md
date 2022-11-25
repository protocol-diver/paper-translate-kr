# paper-translate-kr
분산시스템, 통신 및 프로토콜에 관련된 RFC, 논문의 번역본을 보관하는 저장소입니다. <br>

# 목차
1. Network


## 1. Network
- [x] [RFC894](https://github.com/protocol-diver/rfc-translate-kr/blob/main/kr/rfc894.txt) - A Standard for the Transmission of IP Datagrams over Ethernet Networks (이더넷 네트워크를 통한 IP 데이터그램 전송 표준)
- [ ] [RFC5128](https://github.com/protocol-diver/rfc-translate-kr/blob/main/kr/rfc5128.txt) - State of Peer-to-Peer (P2P) Communication across Network Address Translators (NATs)
- [ ] [RFC6886](https://github.com/protocol-diver/rfc-translate-kr/blob/main/kr/rfc6886.txt) - NAT Port Mapping Protocol (NAT-PMP)


# 규격
한/영 변환 Github Page 운영에 대비하여 원본과 번역본 둘 다 저장합니다.

### 페이지 관련 내용은 삭제하여 커밋합니다(번역하는 과정에서 원본의 페이징이 틀어질 것 같아 미리 제거).

<b>목차</b><br>
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

<b>페이지</b><br>
기존)
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
   SHOULD:{이 프로토콜은 클라이언트가 기본 IPv4 주소가 "Address Allocation for
   Private Internets"[RFC1918]에 정의된 개인 IPv4 주소 범위 중 하나에 있다고 판단하는
   경우에만 사용}해야 합니다.
```
