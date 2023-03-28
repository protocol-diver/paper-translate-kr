# The RLPx Transport Protocol (RLPx 전송 프로토콜)

이 사양은 이더리움 노드 간의 통신에 사용되는 TCP 기반 전송 프로토콜인 RLPx 전송 프로토콜을 정의합니다. 이
프로토콜은 연결 설정 중에 협상되는 하나 이상의 '기능'에 속하는 암호화된 메시지를 전달합니다. RLPx는 [RLP] 직렬화
형식의 이름을 따서 명명되었습니다. 이 이름은 약어가 아니며 특별한 의미가 없습니다.

현재 프로토콜 버전은 **5**입니다. 이전 버전의 변경사항 목록은 이 문서의 끝부분에서 확인할 수 있습니다. 에서 이전
버전의 변경 사항 목록을 확인할 수 있습니다.

## Notation (표기법)

`X || Y`\
    는 X와 Y의 연결을 나타냅니다.\
`X ^ Y`\
    는 X와 Y의 바이트 단위 XOR입니다.\
`X[:N]`\
    는 N바이트의 접두사 X를 나타냅니다.\
`[X, Y, Z, ...]`\
   는 재귀 인코딩을 RLP 목록으로 나타냅니다.\
`keccak256(MESSAGE)`\
    는 이더리움에서 사용하는 Keccak256 해시 함수입니다.\
`ecies.encrypt(PUBKEY, MESSAGE, AUTHDATA)`\
    는 RLPx에서 사용하는 비대칭 인증 암호화 기능입니다.\
    AUTHDATA는 결과 암호 텍스트의 일부가 아닌 인증된 데이터지만,\
    메시지 태그를 생성하기 전에 HMAC-256에 기록됩니다.\
`ecdh.agree(PRIVKEY, PUBKEY)`\
    는 타원 곡선 Diffie-Hellman 키 계약으로, PRIVKEY와 PUBKEY 간의 키 계약입니다.

## ECIES Encryption (ECIES 암호화)

ECIES(Elliptic Curve Integrated Encryption Scheme)는 비대칭 암호화 방식으로 비대칭 암호화 방식입니다.
RLPx에서 사용하는 암호화 시스템은 다음과 같습니다

- 제너레이터 `G`가 있는 타원 곡선 secp256k1입니다.
- `KDF(k, len)`: NIST SP 800-56 연결 키 도출 함수.
- `MAC(k, m)`: SHA-256 해시 함수를 사용하는 HMAC.
- `AES(k, iv, m)`: CTR 모드 AES-128 암호화 함수.

앨리스는 밥의 정적 개인 키 <code>k<sub>B</sub></code>로 복호화할 수 있는 암호화된 메시지를 보내려고 합니다. 앨리스는
밥의 정적 공개 키 <code>K<sub>B</sub></code>에 대해 알고 있습니다.

메시지 `m`을 암호화하기 위해 앨리스는 난수 `r`과 해당 타원 곡선 공개 키 `R = r * G`를 생성하고 <code>(P<sub>x</sub>, P<sub>y</sub>) = r * K<sub>B</sub></code>인 
공유 secret <code>S = P<sub>x</sub></code>를 계산합니다. 앨리스는 암호화 및 인증을 위한 키 자료를 <code>k<sub>E</sub> || k<sub>M</sub> = KDF(S, 32)</code>와
무작위 초기화 벡터 `iv`를 도출합니다. 앨리스는 암호화된 메시지 `R || iv || c || d`를 보내며, 여기서 <code>c = AES(k<sub>E</sub>, iv , m)</code>와
<code>d = MAC(sha256(k<sub>M</sub>), iv || c)</code>를 밥에게 보냅니다.

밥이 메시지 `R || iv || c || d`를 복호화하기 위해서는 공유 secret <code>S = P<sub>x</sub></code>에서 <code>(P<sub>x</sub>, P<sub>y</sub>) = k<sub>B</sub> * R</code>과
암호화 및 인증 키 <code>k<sub>E</sub> || k<sub>M</sub> = KDF(S, 32)</code>를 도출해야 합니다. 
밥은 <code>d == MAC(sha256(k<sub>M</sub>), iv || c)</code>인지 확인하여 메시지의 진위 여부를 확인한 다음 <code>m = AES(k<sub>E</sub>, iv || c)</code>로 평문 텍스트를 얻습니다.


## Node Identity (노드 신원)

모든 암호화 연산은 secp256k1 타원 곡선을 기반으로 합니다. 각 노드는 세션 간에 저장 및 복원되는 정적 secp256k1
개인키를 유지해야 합니다. 개인 키는 파일이나 데이터베이스 항목을 삭제하는 등 수동으로만 재설정할 수 있는 것이 좋습니다.

## Initial Handshake (초기 핸드셰이크)

RLPx 연결은 TCP 연결을 생성하고 추가 암호화 및 인증된 통신을 위한 임시 키 자료에 동의함으로써 설정됩니다. 이러한 세션
키를 생성하는 과정을 'handshake'라고 하며, 'initiator'(TCP 연결을 연 노드)와 'recipient'(연결을 수락한 노드)
사이에서 수행됩니다.

1. message initiator가 recipient에게 연결하고 `auth` 메시지를 보냄
2. recipient가 `auth`을 수락, 복호화 및 확인(recovery of signature ==`keccak256(ephemeral-pubk)`인지 확인)
3. recipient는 `remote-ephemeral-pubk`과 `nonce` 에서 `auth-ack` 메시지를 생성
4. recipient는 secret을 도출하고 [Hello] 메시지가 포함된 첫 번째 암호화된 프레임을 전송
5. initiator는 `auth-ack`을 수신하고 secret을 도출
6. initiator가 initiator [Hello] 메시지가 포함된 첫 번째 암호화된 프레임을 전송
7. recipient는 첫 번째 암호화된 프레임을 수신하고 인증
8. initiator가 첫 번째 암호화된 프레임을 수신하고 인증
9. 첫 번째 암호화된 프레임의 MAC이 양쪽에서 모두 유효하면 암호화 핸드셰이크가 완료

첫 번째 프레임 패킷의 인증이 실패하면 어느 쪽이든 연결이 끊어질 수 있습니다.

Handshake 메세지:

    auth = auth-size || enc-auth-body
    auth-size = size of enc-auth-body, encoded as a big-endian 16-bit integer
    auth-vsn = 4
    auth-body = [sig, initiator-pubk, initiator-nonce, auth-vsn, ...]
    enc-auth-body = ecies.encrypt(recipient-pubk, auth-body || auth-padding, auth-size)
    auth-padding = arbitrary data

    ack = ack-size || enc-ack-body
    ack-size = size of enc-ack-body, encoded as a big-endian 16-bit integer
    ack-vsn = 4
    ack-body = [recipient-ephemeral-pubk, recipient-nonce, ack-vsn, ...]
    enc-ack-body = ecies.encrypt(initiator-pubk, ack-body || ack-padding, ack-size)
    ack-padding = arbitrary data

구현은 `auth-vsn`과 `ack-vsn`의 불일치를 무시해야 합니다. 구현은 `auth-body` 및 `ack-body`의 추가 목록
요소도 무시해야 합니다.

핸드셰이크 메시지를 교환한 후 생성된 secret:

    static-shared-secret = ecdh.agree(privkey, remote-pubk)
    ephemeral-key = ecdh.agree(ephemeral-privkey, remote-ephemeral-pubk)
    shared-secret = keccak256(ephemeral-key || keccak256(nonce || initiator-nonce))
    aes-secret = keccak256(ephemeral-key || shared-secret)
    mac-secret = keccak256(ephemeral-key || aes-secret)

## Framing (프레임 구성)

초기 핸드셰이크 이후의 모든 메시지는 프레임으로 구성됩니다. 프레임은 한 기능에 속하는 하나의 암호화된 메시지를
전달합니다.

프레임의 목적은 단일 연결을 통해 여러 기능을 멀티플렉싱하는 것입니다. 둘째로, 프레임 메시지는 메시지 인증 코드에 대한
합리적인 경계 지점을 생성하므로 암호화되고 인증된 스트림을 지원하는 것이 간단해집니다. 프레임은 핸드셰이크 중에 생성된
키 자료를 통해 암호화 및 인증됩니다.

프레임 헤더는 메시지의 크기와 메시지의 소스 기능에 대한 정보를 제공합니다. 패딩은 버퍼 고갈을 방지하기 위해 사용되며,
프레임 구성 요소는 암호의 블록 크기에 맞게 바이트 정렬됩니다.

    frame = header-ciphertext || header-mac || frame-ciphertext || frame-mac
    header-ciphertext = aes(aes-secret, header)
    header = frame-size || header-data || header-padding
    header-data = [capability-id, context-id]
    capability-id = integer, always zero
    context-id = integer, always zero
    header-padding = zero-fill header to 16-byte boundary
    frame-ciphertext = aes(aes-secret, frame-data || frame-padding)
    frame-padding = zero-fill frame-data to 16-byte boundary

`frame-data` 및 `frame-size`의 정의는 [Capability Messaging] 섹션을 참조하십시오.

### MAC

RLPx의 메시지 인증은 통신 방향별로 하나씩 두 개의 keccak256 상태를 사용합니다. `egress-mac` 및 `ingress-mac`
keccak 상태는 전송(egress) 또는 수신(ingress)된 바이트의 암호문으로 지속적으로 업데이트됩니다. 초기 핸드셰이크
이후 MAC 상태는 다음과 같이 초기화됩니다:

Initiator:

    egress-mac = keccak256.init((mac-secret ^ recipient-nonce) || auth)
    ingress-mac = keccak256.init((mac-secret ^ initiator-nonce) || ack)

Recipient:

    egress-mac = keccak256.init((mac-secret ^ initiator-nonce) || ack)
    ingress-mac = keccak256.init((mac-secret ^ recipient-nonce) || auth)

프레임이 전송되면 전송할 데이터로 `egress-mac` 상태를 업데이트하여 해당 MAC 값을 계산합니다. 업데이트는 헤더를
해당 MAC의 암호화된 출력과 XOR하는 방식으로 수행됩니다. 이는 일반 텍스트 MAC과 암호 텍스트 모두에 대해 균일한
작업이 수행되도록 하기 위해 수행됩니다. 모든 MAC은 일반 텍스트로 전송됩니다.

    header-mac-seed = aes(mac-secret, keccak256.digest(egress-mac)[:16]) ^ header-ciphertext
    egress-mac = keccak256.update(egress-mac, header-mac-seed)
    header-mac = keccak256.digest(egress-mac)[:16]

`frame-mac` 계산:

    egress-mac = keccak256.update(egress-mac, frame-ciphertext)
    frame-mac-seed = aes(mac-secret, keccak256.digest(egress-mac)[:16]) ^ keccak256.digest(egress-mac)[:16]
    egress-mac = keccak256.update(egress-mac, frame-mac-seed)
    frame-mac = keccak256.digest(egress-mac)[:16]

수신 프레임의 MAC 확인은 `egress-mac`과 동일한 방식으로 `ingress-mac` 상태를 업데이트하고 수신 프레임의
`header-mac` 및 `frame-mac` 값과 비교하여 수행합니다. 이 작업은 `header-ciphertext`와
`frame-ciphertext`를 복호화하기 전에 수행해야 합니다.

# Capability Messaging (기능 메시지)

초기 핸드셰이크 이후의 모든 메시지는 '기능'과 연관됩니다. 하나의 RLPx 연결에서 여러 개의 기능을 동시에 사용할
수 있습니다.

기능은 짧은 ASCII 이름(최대 8자)과 버전 번호로 식별됩니다. 연결의 양쪽에서 지원되는 기능은 모든 연결에서 사용할
수 있어야 하는 'p2p' 기능에 속하는 [Hello] 메시지에서 교환됩니다.

## Message Encoding (메시지 인코딩)

초기 [Hello] 메시지는 다음과 같이 인코딩됩니다:

    frame-data = msg-id || msg-data
    frame-size = length of frame-data, encoded as a 24bit big-endian integer

여기서 `msg-id`는 메시지를 식별하는 RLP 인코딩된 정수이고 `msg-data`는 메시지 데이터가 포함된 RLP 목록입니다.

Hello 다음에 오는 모든 메시지는 Snappy 알고리즘을 사용하여 압축됩니다.

    frame-data = msg-id || snappyCompress(msg-data)
    frame-size = length of frame-data encoded as a 24bit big-endian integer

압축된 메시지의 `frame-size`는 `msg-data`의 압축된 크기를 나타냅니다. 압축된 메시지는 압축 해제 후 매우 큰
크기로 부풀어 오를 수 있으므로, 구현은 메시지를 디코딩하기 전에 데이터의 압축되지 않은 크기를 확인해야 합니다. 이는
[snappy format]에 길이 헤더가 포함되어 있기 때문에 가능합니다. 압축되지 않은 데이터가 16 MiB보다 큰 메시지는
연결을 닫아 거부해야 합니다.

## Message ID-based Multiplexing (메시지 ID 기반 멀티플렉싱)

프레이밍 레이어는 `capability-id`를 지원하지만, 현재 버전의 RLPx에서는 이 필드를 서로 다른 기능 간의
멀티플렉싱에 해당 필드를 사용하지 않습니다. 대신 멀티플렉싱은 는 순전히 메시지 ID에만 의존합니다.

각 기능에는 필요한 만큼의 메시지 ID 공간이 주어집니다. 이러한 모든 기능은 필요한 메시지 ID의 수를 정적으로 지정해야
합니다. [Hello] 메시지를 연결하고 수신할 때 두 피어는 서로가 공유하는 기능(버전 포함)에 대한 동등한 정보를 가지고
있으며 메시지 ID 공간의 구성에 대한 합의를 형성할 수 있습니다.

메시지 ID는 ID 0x10 이후부터는 압축된 것으로 간주되며(0x00-0x0f는 "p2p" 기능을 위해 예약됨), 각 공유(동일한 버전,
동일한 이름) 기능에 알파벳 순서로 부여됩니다. 기능 이름은 대소문자를 구분합니다. 공유되지 않는 기능은 무시됩니다.
동일한(동일한 이름의) 기능을 여러 버전이 공유한 경우, 숫자가 가장 높은 버전이 승리하고 다른 버전은 무시됩니다.

## "p2p" Capability (p2p 기능)

"p2p" 기능은 모든 연결에 존재합니다. 최초 핸드셰이크가 끝나면 연결 양쪽에서 [Hello] 또는 [Disconnect] 메시지를
보내야 합니다. [Hello] 메시지를 받으면 세션이 활성 상태이며 다른 메시지를 보낼 수 있습니다. 구현은 프로토콜 버전 차이를
무시해야 하며, 이는 하위 호환성을 위해서입니다. 하위 버전의 피어와 통신할 때 구현은 해당 버전을 모방하려고 시도해야 합니다.

프로토콜 협상 후 언제든지 [Disconnect] 메시지가 전송될 수 있습니다.

### Hello (0x00)

`[protocolVersion: P, clientId: B, capabilities, listenPort: P, nodeKey: B_64, ...]`

연결을 통해 전송되는 첫 번째 패킷으로, 양쪽에서 한 번씩 전송됩니다. 헬로가 수신될 때까지 다른 메시지는 전송할 수 없습니다.
구현은 Hello의 추가 목록 요소를 향후 버전에서 사용할 수 있으므로 무시해야 합니다.

- `protocolVersion`은 'p2p'기능의 버전입니다, **5**.
- `clientId`는 사람이 읽을 수 있는 문자열(e.g. "Ethereum(++)/1.0.0")로 클라이언트 소프트웨어 ID를 지정합니다.
- `capabilities`은 지원되는 기능 및 해당 버전 목록입니다:
  `[[cap1, capVersion1], [cap2, capVersion2], ...]`.
- `listenPort`는 클라이언트가 수신 대기 중인 포트(현재 연결이 통과하는 인터페이스에서)를 지정합니다. 0이면 클라이언트가
  수신 대기 중이 아님을 나타냅니다.
- `nodeId`는 노드의 개인키에 해당하는 secp256k1 공개키입니다.

### Disconnect (0x01)

`[reason: P]`

상대방에게 연결 해제가 임박했음을 알리고, 상대방이 이를 수신하면 즉시 연결을 끊어야 합니다. 전송할 때 올바른 호스트는
연결을 끊기 전에 상대방이 연결을 끊을 수 있는 시간(읽기: 2초 기다리기)을 줍니다.

`reason`는 여러 연결 해제 이유 중 하나를 지정하는 선택적 정수입니다:

| Reason | Meaning                                                      |
|--------|:-------------------------------------------------------------|
| `0x00` | Disconnect requested                                         |
| `0x01` | TCP sub-system error                                         |
| `0x02` | Breach of protocol, e.g. a malformed message, bad RLP, ...   |
| `0x03` | Useless peer                                                 |
| `0x04` | Too many peers                                               |
| `0x05` | Already connected                                            |
| `0x06` | Incompatible P2P protocol version                            |
| `0x07` | Null node identity received - this is automatically invalid  |
| `0x08` | Client quitting                                              |
| `0x09` | Unexpected identity in handshake                             |
| `0x0a` | Identity is the same as this node (i.e. connected to itself) |
| `0x0b` | Ping timeout                                                 |
| `0x10` | Some other reason specific to a subprotocol                  |

### Ping (0x02)

`[]`

상대방에게 [Pong]의 즉각적인 응답을 요청합니다.

### Pong (0x03)

`[]`

상대방의 [Ping] 패킷에 응답합니다.

# Change Log

### Known Issues in the current version (현재 버전에서 알려진 문제)

- 프레임 암호화/MAC 체계는 `aes-secret`과 `mac-secret`이 읽기 및 쓰기 모두에 재사용되기 때문에 '깨진' 것으로
  간주됩니다. RLPx 연결의 양쪽은 동일한 키, 논스 및 IV에서 두 개의 CTR 스트림을 생성합니다. 공격자가 하나의 평문을
  알고 있다면 재사용된 키 스트림의 알려지지 않은 평문을 복호화할 수 있습니다.
- 검토자들의 일반적인 피드백은 MAC 누산기로 keccak256 상태를 사용하고 MAC 알고리즘에 AES를 사용하는 것은 메시지
  인증을 수행하는 흔하지 않고 지나치게 복잡한 방법이지만 안전한 것으로 간주할 수 있다는 것입니다.
- 프레임 인코딩은 멀티플렉싱을 위해 `capability-id`와 `context-id` 필드를 제공하지만 이 필드들은 사용되지
  않습니다.

### Version 5 (EIP-706, September 2017)

[EIP-706]에 Snappy 메시지 압축이 추가되었습니다.

### Version 4 (EIP-8, December 2015)

[EIP-8]은 초기 핸드셰이크에서 `auth-body`와 `back-body`의 인코딩을 RLP로 변경하고, 핸드셰이크에 버전 번호를
추가했으며, 구현이 핸드셰이크 메시지와 [Hello]에서 추가 목록 요소를 무시하도록 의무화했습니다.

# References (참조)

- Elaine Barker, Don Johnson, and Miles Smid. NIST Special Publication 800-56A Section 5.8.1,
  Concatenation Key Derivation Function. 2017.\
  URL <https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-56ar.pdf>

- Victor Shoup. A proposal for an ISO standard for public key encryption, Version 2.1. 2001.\
  URL <http://www.shoup.net/papers/iso-2_1.pdf>

- Mike Belshe and Roberto Peon. SPDY Protocol - Draft 3. 2014.\
  URL <http://www.chromium.org/spdy/spdy-protocol/spdy-protocol-draft3>

- Snappy compressed format description. 2011.\
  URL <https://github.com/google/snappy/blob/master/format_description.txt>

Copyright &copy; 2014 Alex Leverington.
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">
This work is licensed under a
Creative Commons Attribution-NonCommercial-ShareAlike
4.0 International License</a>.

[Hello]: #hello-0x00
[Disconnect]: #disconnect-0x01
[Ping]: #ping-0x02
[Pong]: #pong-0x03
[Capability Messaging]: #capability-messaging
[EIP-8]: https://eips.ethereum.org/EIPS/eip-8
[EIP-706]: https://eips.ethereum.org/EIPS/eip-706
[RLP]: https://ethereum.org/en/developers/docs/data-structures-and-encoding/rlp
[snappy format]: https://github.com/google/snappy/blob/master/format_description.txt
