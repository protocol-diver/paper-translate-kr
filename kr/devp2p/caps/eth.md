# Ethereum Wire Protocol (ETH)

'eth'는 이더리움의 교환을 용이하게 하는 [RLPx] 전송 프로토콜입니다. 피어 간의 블록체인 정보. 현재 프로토콜 버전은
**eth/67**입니다. 과거 프로토콜 버전의 변경 사항 목록은 문서 끝 부분을 참조하십시오.

### Basic Operation (기본 동작)

연결이 설정되면 [Status] 메시지를 보내야 합니다. 피어의 상태 메시지 수신 후 Ethereum 세션이 활성화되고 다른
메시지를 보낼 수 있습니다.

세션 내에서 체인 동기화, 블록 전파 및 트랜잭션 교환의 세 가지 상위 수준 작업을 수행할 수 있습니다. 이러한 작업은
연결되지 않은 프로토콜 메시지 세트를 사용하며 클라이언트는 일반적으로 모든 피어 연결에서 동시 활동으로 이를 수행합니다.

클라이언트 구현은 프로토콜 메시지 크기에 제한을 적용해야 합니다. 기본 RLPx 전송은 단일 메시지의 크기를 16.7 MiB로
제한합니다. eth 프로토콜의 실제 제한은 더 낮으며 일반적으로 10 MiB입니다. 수신된 메시지가 제한보다 크면 피어 연결을
끊어야 합니다.

수신된 메시지에 대한 엄격한 제한 외에도 클라이언트는 보내는 요청 및 응답에 'soft(부드러운)' 제한을 적용해야 합니다.
권장되는 soft 제한은 메시지 유형에 따라 다릅니다. 요청 및 응답을 제한하면 동시 활동이 보장됩니다. 예를 들어 블록
동기화 및 트랜잭션 교환은 동일한 피어 연결에서 원활하게 작동합니다.

### Chain Synchronization (체인 동기화)

eth 프로토콜에 참여하는 노드는 제네시스 블록에서 현재 최신 블록까지 모든 블록의 전체 체인에 대한 지식을 가지고
있어야 합니다. 체인은 다른 피어에서 다운로드하여 얻습니다.

연결 시 두 피어는 총 난이도(TD)와 '가장' 잘 알려진 블록의 해시를 포함하는 [Status] 메시지를 보냅니다.

최악의 TD를 가진 클라이언트는 [GetBlockHeaders] 메시지를 사용하여 블록 헤더 다운로드를 진행합니다. 수신된 헤더의
작업 증명 값을 확인하고 [GetBlockBodies] 메시지를 사용하여 블록 바디를 가져옵니다. 받은 블록은 이더리움 가상
머신을 사용하여 실행되어 상태 트리와 영수증을 다시 생성합니다.

헤더 다운로드, 블록 본문 다운로드 및 블록 실행은 동시에 발생할 수 있습니다.

### State Synchronization (a.k.a. "fast sync") 상태 동기화(일명 "빠른 동기화")

eth/63에서 eth/66까지의 프로토콜 버전도 상태 트리 동기화를 허용했습니다. 프로토콜 버전 eth/67부터 이더리움 상태
트리는 더 이상 eth 프로토콜을 사용하여 검색할 수 없으며 대신 보조 [snap protocol]에서 상태 다운로드를 제공합니다.

상태 동기화는 일반적으로 블록 헤더 체인을 다운로드하여 유효성을 확인하는 방식으로 진행됩니다. 체인 동기화 섹션에서와
같이 블록 바디를 요청하지만 트랜잭션이 실행되지 않고 '데이터 유효성'만 확인됩니다. 클라이언트는 체인의 헤드 근처에 있는
블록('피벗 블록')을 선택하고 해당 블록의 상태를 다운로드합니다.

### Block Propagation (블록 전파)

새로 채굴된 블록은 모든 노드에 전달되어야 합니다. 이는 2단계 프로세스인 블록 전파를 통해 발생합니다. 피어로부터
[NewBlock] 알림 메시지를 받으면 클라이언트는 먼저 블록의 기본 헤더 유효성을 확인하여 작업 증명 값이 유효한지
확인합니다. 그런 다음 [NewBlock] 메시지를 사용하여 연결된 피어의 작은 부분(일반적으로 전체 피어 수의 제곱근)에
블록을 보냅니다.

헤더 유효성 검사 후 클라이언트는 블록에 포함된 모든 트랜잭션을 실행하고 블록의 '이후 상태'를 계산하여 블록을 로컬
체인으로 가져옵니다. 블록의 `state-root` 해시는 계산된 사후 상태 루트와 일치해야 합니다. 블록이 완전히 채워지면
처리되고 유효한 것으로 간주되면 클라이언트는 이전에 알리지 않은 모든 피어에게 블록에 대한 [NewBlockHashes] 메시지를
보냅니다. 해당 피어는 다른 사람으로부터 [NewBlock]을 통해 수신하지 못한 경우 나중에 전체 블록을 요청할 수 있습니다.

노드는 이전에 동일한 블록을 알린 피어에게 블록 알림을 다시 보내서는 안 됩니다. 이는 일반적으로 각 피어와 최근에
릴레이된 대규모 블록 해시 세트를 기억함으로써 달성됩니다.

블록 알림 수신은 블록이 클라이언트의 현재 최신 블록의 바로 후속 블록이 아닌 경우 체인 동기화를 트리거할 수도 있습니다.

### Transaction Exchange (트랜잭션 교환)

모든 노드는 블록체인에 포함할 트랜잭션을 선택하는 채굴자에게 전달하기 위해 보류 중인 트랜잭션을 교환해야 합니다.
클라이언트 구현은 '트랜잭션 풀'에서 보류 중인 트랜잭션 세트를 추적합니다. 풀은 클라이언트별 제한이 적용되며 많은(즉,
수천 개의) 트랜잭션을 포함할 수 있습니다.

새 피어 연결이 설정되면 양쪽의 트랜잭션 풀을 동기화해야 합니다. 초기에 양쪽 끝은 교환을 시작하기 위해 로컬 풀의 모든
트랜잭션 해시를 포함하는 [NewPooledTransactionHashes] 메시지를 보내야 합니다.

NewPooledTransactionHashes 알림을 수신하면 클라이언트는 수신된 세트를 필터링하여 자체 로컬 풀에 아직 없는
트랜잭션 해시를 수집합니다. 그런 다음 [GetPooledTransactions] 메시지를 사용하여 트랜잭션을 요청할 수 있습니다.

클라이언트의 풀에 새로운 트랜잭션이 나타나면 [Transactions] 및 [NewPooledTransactionHashes] 메시지를
사용하여 네트워크에 전파해야 합니다. 트랜잭션 메시지는 완전한 트랜잭션 개체를 중계하며 일반적으로 연결된 피어의 작은
임의 부분으로 전송됩니다. 다른 모든 피어는 트랜잭션 해시에 대한 알림을 받고 알 수 없는 경우 완전한 트랜잭션 개체를
요청할 수 있습니다. 완전한 트랜잭션을 피어의 일부에게 배포하면 일반적으로 모든 노드가 트랜잭션을 수신하고 이를 요청할
필요가 없습니다.

노드는 트랜잭션을 이미 알고 있다고 판단할 수 있는 피어에게 트랜잭션을 되돌려 보내서는 안 됩니다(이전에 전송되었거나
원래 이 피어로부터 정보를 받았기 때문). 이것은 일반적으로 피어가 최근에 릴레이한 일련의 트랜잭션 해시를 기억함으로써
달성됩니다.

### Transaction Encoding and Validity (트랜잭션 인코딩 및 유효성)

피어가 교환하는 트랜잭션 개체에는 두 가지 인코딩 중 하나가 있습니다. 이 사양의 정의에서 우리는 식별자 `txₙ`를
사용하는 인코딩 트랜잭션을 참조합니다.

    tx = {legacy-tx, typed-tx}

유형이 지정되지 않은 레거시 트랜잭션은 RLP 목록으로 제공됩니다.

    legacy-tx = [
        nonce: P,
        gas-price: P,
        gas-limit: P,
        recipient: {B_0, B_20},
        value: P,
        data: B,
        V: P,
        R: P,
        S: P,
    ]

[EIP-2718] 유형이 지정된 트랜잭션은 첫 번째 바이트가 트랜잭션 유형(`tx-type`)이고 나머지 바이트가 불투명한
유형별 데이터인 RLP 바이트 배열로 인코딩됩니다.

    typed-tx = tx-type || tx-data

트랜잭션은 수신 시 유효성을 검사해야 합니다. 유효성은 Ethereum 체인 상태에 따라 다릅니다. 이 사양과 관련된 특정
종류의 유효성은 트랜잭션이 EVM에 의해 성공적으로 실행될 수 있는지 여부가 아니라 로컬 풀의 임시 저장 및 다른 피어와의
교환에 허용되는지 여부입니다.

트랜잭션은 아래 규칙에 따라 검증됩니다. 유형이 지정된 트랜잭션의 인코딩은 불투명하지만 `tx-data`는 `nonce`,
`gas-price`, `gas limit`에 대한 값을 제공한다고 가정하며 트랜잭션의 발신자 계정은 서명에서 확인할 수 있습니다.

- 유형이 지정된 트랜잭션은 `tx-type`이 구현에 알려져야 합니다. 정의된 트랜잭션 유형은 블록에 포함될 수 있게 되기
  전에도 유효한 것으로 간주될 수 있습니다. 구현은 알 수 없는 유형의 트랜잭션을 보내는 피어의 연결을 끊어야 합니다.
- 서명은 체인에서 지원하는 서명 체계에 따라 유효해야 합니다. 유형이 지정된 트랜잭션의 경우 서명 처리는 유형을 도입하는
  EIP에 의해 정의됩니다. 레거시 트랜잭션의 경우 현재 사용되고 있는 두 가지 방식은 기본 'Homestead' 방식과
  [EIP-155] 방식이다.
- `gas-limit`는 트랜잭션의 `intrinsic gas`를 포함해야 합니다.
- 서명에서 파생된 트랜잭션의 발신자 계정에는 거래 비용(`gas-limit * gas-price + value`)을 충당할 수 있는
  충분한 이더 잔고가 있어야 합니다.
- 트랜잭션의 `nonce`는 발신자 계정의 현재 nonce보다 크거나 같아야 합니다.
- 로컬 풀에 포함할 트랜잭션을 고려할 때 현재 계정 nonce보다 더 큰 nonce을 가진 'future' 트랜잭션이 얼마나
  유효한지, 허용 가능한 'nonce gaps'의 정도를 결정하는 것은 구현에 달려 있습니다.

구현은 트랜잭션에 대한 다른 유효성 검사 규칙을 적용할 수 있습니다. 예를 들어 128kB보다 큰 인코딩된 트랜잭션을
거부하는 것이 일반적입니다.

달리 명시되지 않는 한, 구현은 유효하지 않은 트랜잭션을 보내기 위해 피어의 연결을 끊지 않아야 하며 대신 단순히 폐기해야
합니다. 이는 피어가 약간 다른 유효성 검사 규칙에 따라 작동할 수 있기 때문입니다.

### Block Encoding and Validity (블록 인코딩 및 유효성)

Ethereum 블록은 다음과 같이 인코딩됩니다:

    block = [header, transactions, ommers]
    transactions = [tx₁, tx₂, ...]
    ommers = [header₁, header₂, ...]
    header = [
        parent-hash: B_32,
        ommers-hash: B_32,
        coinbase: B_20,
        state-root: B_32,
        txs-root: B_32,
        receipts-root: B_32,
        bloom: B_256,
        difficulty: P,
        number: P,
        gas-limit: P,
        gas-used: P,
        time: P,
        extradata: B,
        mix-digest: B_32,
        block-nonce: B_8,
        basefee-per-gas: P,
    ]

특정 프로토콜 메시지에서 트랜잭션 및 ommers 목록은 'block body'이라는 단일 항목으로 함께 릴레이됩니다.

    block-body = [transactions, ommers]

블록 헤더의 유효성은 사용되는 컨텍스트에 따라 다릅니다. 단일 블록 헤더의 경우 작업 증명 봉인(`mix-digest`,
`block-nonce`)의 유효성만 확인할 수 있습니다. 헤더가 클라이언트의 로컬 체인을 확장하는 데 사용되거나 체인 동기화
중에 여러 헤더가 순차적으로 처리되는 경우 다음 규칙이 적용됩니다:

- 헤더는 블록 번호가 연속적이고 각 헤더의 `parent-hash`가 이전 헤더의 해시와 일치하는 체인을 형성해야 합니다.
- 로컬에 저장된 체인을 확장할 때 구현은 `difficulty`, `gas-limit` 및 `time` 값이 [Yellow Paper]에
  제공된 프로토콜 규칙 범위 내에 있는지도 확인해야 합니다.
- `gas-used` 헤더 필드는 `gas-limit`보다 작거나 같아야 합니다.
- [London hard fork] 이후 블록에 대해 `basefee-per-gas` 헤더 필드가 있어야 합니다. 이전 블록에는
  `basefee-per-gas`가 없어야 합니다. 이 규칙은 [EIP-1559]에 의해 이더리움 블록 해시의 정의에 필드를 추가하여
  암시성을 추가했습니다.

완전한 블록의 경우 블록의 EVM 상태 전환 유효성과 블록의 (약한) '데이터 유효성'을 구별합니다. 상태 전이 규칙의 정의는
이 명세서에서 다루지 않습니다. 즉각적인 [block propagation]와 [state synchronization] 동안 블록의 데이터
유효성이 필요합니다.

블록의 데이터 유효성을 확인하려면 아래 규칙을 사용하십시오. 구현은 유효하지 않은 블록을 보내는 피어의 연결을 끊어야
합니다.

- 블록 `header`는 유효해야 합니다.
- 블록에 포함된 `transactions`은 블록 번호의 체인에 포함되기 위해 유효해야 합니다. 즉, 앞서 제시한 트랜잭션 유효성
  검사 규칙 외에도 블록 번호에서 `tx-type`이 허용되는지 유효성 검사가 필요하며 트랜잭션 가스의 유효성 검사는 블록
  번호를 고려해야 합니다.
- 모든 트랜잭션의 `gas-limit`의 합은 블록의 `gas-limit`를 초과하지 않아야 합니다.
- 트랜잭션 목록의 머클 트리 해시를 계산하고 비교하여 블록의 `transactions`을 `txs-root`에 대해 확인해야 합니다.
- `ommers` 목록에는 최대 2개의 헤더가 포함될 수 있습니다.
- `keccak256(ommers)`는 블록 헤더의 `ommers-hash`와 일치해야 합니다.
- `ommers` 목록에 포함된 헤더는 유효한 헤더여야 합니다. 그들의 블록 번호는 그들이 포함된 블록의 번호보다 크지 않아야
  합니다. ommer 헤더의 부모 해시는 포함 블록의 깊이 7 이하의 조상을 참조해야 하며 이 조상 집합에 포함된 이전 블록에
  포함되지 않아야 합니다.

### Receipt Encoding and Validity (영수증 인코딩 및 유효성)

영수증은 블록의 EVM 상태 전환의 출력입니다. 거래와 마찬가지로 영수증에는 두 가지 고유한 인코딩이 있으며 식별자
`receiptₙ`를 사용하여 인코딩을 참조합니다.

    receipt = {legacy-receipt, typed-receipt}

유형이 지정되지 않은 레거시 영수증은 다음과 같이 인코딩됩니다:

    legacy-receipt = [
        post-state-or-status: {B_32, {0, 1}},
        cumulative-gas: P,
        bloom: B_256,
        logs: [log₁, log₂, ...]
    ]
    log = [
        contract-address: B_20,
        topics: [topic₁: B, topic₂: B, ...],
        data: B
    ]

[EIP-2718] 유형이 지정된 영수증은 첫 번째 바이트가 영수증 유형(`tx-type`과 일치)을 제공하고 나머지 바이트는 해당
유형에 특정한 불투명 데이터인 RLP 바이트 배열로 인코딩됩니다.

    typed-receipt = tx-type || receipt-data

Ethereum Wire Protocol에서 영수증은 항상 블록에 포함된 모든 영수증의 전체 목록으로 전송됩니다. 또한 영수증을
포함하는 블록이 유효하고 알려져 있다고 가정합니다. 피어가 블록 수신 목록을 수신하면 목록의 머클 트리 해시를 계산하고
블록의 `receipts-root`와 비교하여 확인해야 합니다. 유효한 영수증 목록은 EVM 상태 전환에 의해 결정되므로 이 사양에서
영수증에 대한 추가 유효성 규칙을 정의할 필요가 없습니다.

## Protocol Messages (프로토콜 메시지)

대부분의 메시지에서 메시지 데이터 목록의 첫 번째 요소는 `request-id`입니다. 요청의 경우 요청 피어에서 선택한
64비트 정수 값입니다. 응답하는 피어는 응답 메시지의 `request-id` 요소 값을 미러링해야 합니다.

### Status (0x00)

`[version: P, networkid: P, td: P, blockhash: B_32, genesis: B_32, forkid]`

피어에게 현재 상태를 알립니다. 이 메시지는 연결이 설정된 직후 다른 eth 프로토콜 메시지보다 먼저 전송되어야 합니다.

- `version`: 현재 프로토콜 버전
- `networkid`: 블록체인을 식별하는 정수, 아래 표 참조
- `td`: 최고의 체인의 총 난이도. 블록 헤더에 있는 정수입니다.
- `blockhash`: 가장 좋은(즉, 가장 높은 TD) 알려진 블록의 해시
- `genesis`: 제네시스 블록의 해시
- `forkid`: `[fork-hash, fork-next]`로 인코딩된 [EIP-2124] 포크 식별자.

이 표에는 일반적인 네트워크 ID와 해당 네트워크가 나열되어 있습니다. 나열되지 않은 다른 ID가 있습니다. 즉,
클라이언트는 특정 네트워크 ID를 사용하도록 요구하지 않아야 합니다. 네트워크 ID는 트랜잭션 재생 방지에 사용되는
EIP-155 Chain ID와 일치할 수도 있고 일치하지 않을 수도 있습니다.

| ID | chain                         |
|----|-------------------------------|
| 0  | Olympic (disused)             |
| 1  | Frontier (now mainnet)        |
| 2  | Morden (disused)              |
| 3  | Ropsten (current PoW testnet) |
| 4  | [Rinkeby]                     |

커뮤니티에서 선별한 체인 ID 목록은 <https://chainid.network>를 참조하세요.

### NewBlockHashes (0x01)

`[[blockhash₁: B_32, number₁: P], [blockhash₂: B_32, number₂: P], ...]`

네트워크에 나타난 하나 이상의 새로운 블록을 지정하십시오. 최대로 도움이 주려면 노드는 피어가 인식하지 못할 수 있는 모든
블록을 피어에게 알려야 합니다. 보내는 피어가 합리적으로 알 수 있는 것으로 간주될 수 있는 해시를 포함하는 것은(해당 노드
자체가 NewBlockHashes를 통해 해시에 대한 지식을 광고했고 이전에 알려졌기 때문에) 잘못된 형식으로 간주되며 보내는
노드의 평판을 떨어뜨릴 수 있습니다. 전송 노드가 나중에 진행 중인 [GetBlockHeaders] 메시지를 준수하기를 거부하는
해시를 포함하는 것은 잘못된 형식으로 간주되며 전송 노드의 평판을 떨어뜨릴 수 있습니다.

### Transactions (0x02)

`[tx₁, tx₂, ...]`

피어가 트랜잭션 대기열에 포함되도록 해야 하는 트랜잭션을 지정합니다. 목록의 항목은 주요 이더리움 사양에 설명된 형식의
트랜잭션입니다. 트랜잭션 메시지는 적어도 하나의 (새) 트랜잭션을 포함해야 하며 빈 트랜잭션 메시지는 사용하지 않는 것이
좋으며 연결이 끊어질 수 있습니다.

노드는 동일한 세션의 피어에게 동일한 트랜잭션을 다시 보내서는 안 되며 해당 트랜잭션을 받은 피어에게 트랜잭션을
릴레이해서는 안 됩니다. 실제로 이는 이미 보내거나 받은 트랜잭션 해시 세트 또는 피어별 블룸 필터를 유지하여 구현되는
경우가 많습니다.

### GetBlockHeaders (0x03)

`[request-id: P, [startblock: {P, B_32}, limit: P, skip: P, reverse: {0, 1}]]`

BlockHeaders 메시지를 반환하려면 피어가 필요합니다. 응답에는 `reverse`가 `0`일 때 증가하는 숫자, `1`일 때
떨어지는 `skip` 블록, 최대 `limit` 항목이 있는 캐노니컬 체인의 `startblock` 블록에서 시작하는 여러 블록의 헤더가
포함되어야 합니다.

### BlockHeaders (0x04)

`[request-id: P, [header₁, header₂, ...]]`

이것은 요청된 헤더를 포함하는 GetBlockHeaders에 대한 응답입니다. 요청된 블록 헤더가 없으면 헤더 목록이 비어 있을 수
있습니다. 단일 메시지에서 요청할 수 있는 헤더의 수는 구현 정의 제한에 따라 달라질 수 있습니다.

BlockHeaders 응답에 권장되는 soft 제한은 2 MiB입니다.

### GetBlockBodies (0x05)

`[request-id: P, [blockhash₁: B_32, blockhash₂: B_32, ...]]`

이 메시지는 해시로 블록 바디 데이터를 요청합니다. 단일 메시지에서 요청할 수 있는 블록 수는 구현 정의 제한에 따라 달라질
수 있습니다.

### BlockBodies (0x06)

`[request-id: P, [block-body₁, block-body₂, ...]]`

이것은 GetBlockBodies에 대한 응답입니다. 목록의 항목에는 요청된 블록의 바디 데이터가 포함됩니다. 요청된 블록을 사용할
수 없는 경우 목록이 비어 있을 수 있습니다.

BlockBodies 응답에 권장되는 osft 제한은 2 MiB입니다.

### NewBlock (0x07)

`[block, td: P]`

피어가 알아야 하는 단일 전체 블록을 지정합니다. `td`는 블록의 총 난이도, 즉 이 블록을 포함한 모든 블록 난이도의
합입니다.

### NewPooledTransactionHashes (0x08)

`[txhash₁: B_32, txhash₂: B_32, ...]`

이 메시지는 네트워크에 나타나고 아직 블록에 포함되지 않은 하나 이상의 트랜잭션을 알립니다. 최대로 도움이 되려면 노드는
피어가 인식하지 못할 수 있는 모든 트랜잭션을 피어에게 알려야 합니다.

이 메시지에 권장되는 soft 제한은 4096개 해시(128 KiB)입니다.

노드는 원격 피어가 합리적으로 알 수 없는 것으로 간주될 수 있는 트랜잭션의 해시만 발표해야 하지만 풀에 nonce 갭이 있는
것보다 더 많은 트랜잭션을 반환하는 것이 좋습니다.

### GetPooledTransactions (0x09)

`[request-id: P, [txhash₁: B_32, txhash₂: B_32, ...]]`

이 메시지는 해시로 수신자의 트랜잭션 풀에서 트랜잭션을 요청합니다.

GetPooledTransactions 요청에 대한 권장 soft 제한은 256개 해시(8 KiB)입니다. 수신자는 응답(크기 또는 서빙
시간)에 대해 임의의 제한을 적용할 수 있으며, 이는 프로토콜 위반으로 간주되어서는 안 됩니다.

### PooledTransactions (0x0a)

`[request-id: P, [tx₁, tx₂...]]`

로컬 풀에서 요청된 트랜잭션을 반환하는 GetPooledTransactions에 대한 응답입니다. 목록의 항목은 주요 이더리움 사양에
설명된 형식의 트랜잭션입니다.

트랜잭션은 요청과 동일한 순서여야 하지만 사용할 수 없는 트랜잭션은 건너뛰어도 됩니다. 이렇게 하면 응답 크기 제한에
도달하면 요청자는 다시 요청할 해시 (마지막 반환된 트랜잭션에서 시작하는 모든 것)와 사용할 수 없다고 가정할 해시 (마지막
반환된 트랜잭션 이전의 모든 간격)를 알 수 있습니다.

먼저 NewPooledTransactionHashes를 통해 트랜잭션을 알리고 PooledTransactions를 통해 제공을 거부하는 것이
허용됩니다. 이러한 상황은 announcement와 요청 사이에 트랜잭션이 블록에 포함되고 풀에서 제거될 때 발생할 수 있습니다.

풀의 트랜잭션과 일치하는 해시가 없는 경우 피어는 빈 목록으로 응답할 수 있습니다.

### GetReceipts (0x0f)

`[request-id: P, [blockhash₁: B_32, blockhash₂: B_32, ...]]`

피어가 주어진 블록 해시의 영수증을 포함하는 Receipts 메시지를 반환하도록 요구합니다. 단일 메시지에서 요청할 수 있는
확인 수는 구현 정의 제한에 따라 달라질 수 있습니다.

### Receipts (0x10)

`[request-id: P, [[receipt₁, receipt₂], ...]]`

요청된 블록 영수증을 제공하는 GetReceipts에 대한 응답입니다. 응답 목록의 각 요소는 GetReceipts 요청의 블록 해시에
해당하며 블록의 전체 영수증 목록을 포함해야 합니다.

Receipts 응답에 대한 권장 soft 한계는 2 MiB입니다.

## Change Log (변경 로그)

### eth/67 ([EIP-4938], March 2022)

버전 67에서는 GetNodeData 및 NodeData 메시지가 제거되었습니다.

- GetNodeData (0x0d)
  `[request_id: P, [hash_0: B_32, hash_1: B_32, ...]]`
- NodeData (0x0e)
  `[request_id: P, [value_0: B, value_1: B, ...]]`

### eth/66 ([EIP-2481], April 2021)

버전 66은 [GetBlockHeaders], [BlockHeaders], [GetBlockBodies], [BlockBodies],
[GetPooledTransactions], [PooledTransactions], GetNodeData, NodeData, [GetReceipts],
[Receipts] 메시지에 `request-id` 요소를 추가했습니다.

### eth/65 with typed transactions ([EIP-2976], April 2021)

유형이 지정된 트랜잭션이 [EIP-2718]에 의해 도입되었을 때 클라이언트 구현자는 프로토콜 버전을 높이지 않고 유선
프로토콜에서 새로운 트랜잭션 및 영수증 형식을 수락하기로 결정했습니다. 이 사양 업데이트는 또한 Yellow Paper를 참조하는
대신 모든 합의 개체의 인코딩에 대한 정의를 추가했습니다.

### eth/65 ([EIP-2464], January 2020)

버전 65는 트랜잭션 교환을 개선하여 세 가지 추가 메시지를 도입했습니다:
[NewPooledTransactionHashes], [GetPooledTransactions] 및 [PooledTransactions].

버전 65 이전에는 피어가 항상 완전한 트랜잭션 개체를 교환했습니다. 이더리움 메인넷에서 활동 및 트랜잭션 크기가 증가함에
따라 트랜잭션 교환에 사용되는 네트워크 대역폭은 노드 운영자에게 상당한 부담이 되었습니다. 업데이트는 블록 전파와 유사한
2계층 트랜잭션 브로드캐스트 시스템을 채택하여 필요한 대역폭을 줄였습니다.

### eth/64 ([EIP-2364], November 2019)

버전 64는 [EIP-2124] ForkID를 포함하도록 [Status] 메시지를 변경했습니다. 이를 통해 피어는 블록체인을 동기화하지
않고도 체인 실행 규칙의 상호 호환성을 결정할 수 있습니다.

### eth/63 (2016)

버전 63은 트랜잭션 실행 결과를 동기화할 수 있는 GetNodeData, NodeData, [GetReceipts] 및 [Receipts]
메시지를 추가했습니다.

### eth/62 (2015)

버전 62에서 [NewBlockHashes] 메시지는 발표된 해시와 함께 블록 번호를 포함하도록 확장되었습니다. [상태]의 블록
번호가 제거되었습니다. GetBlockHashes(0x03), BlockHashes(0x04), GetBlocks(0x05) 및 Blocks(0x06)
메시지는 블록 헤더 및 본문을 가져오는 메시지로 대체되었습니다. BlockHashesFromNumber(0x08) 메시지가
제거되었습니다.

재할당/제거된 메시지 코드의 이전 인코딩은 다음과 같습니다:

- GetBlockHashes (0x03): `[hash: B_32, max-blocks: P]`
- BlockHashes (0x04): `[hash₁: B_32, hash₂: B_32, ...]`
- GetBlocks (0x05): `[hash₁: B_32, hash₂: B_32, ...]`
- Blocks (0x06): `[[header, transactions, ommers], ...]`
- BlockHashesFromNumber (0x08): `[number: P, max-blocks: P]`

### eth/61 (2015)

버전 61은 블록을 오름차순으로 요청하는 데 사용할 수 있는 BlockHashesFromNumber(0x08) 메시지를 추가했습니다.
또한 [Status] 메시지에 최신 블록 번호를 추가했습니다.

### eth/60 and below

60 미만의 버전 번호는 Ethereum PoC 개발 단계에서 사용되었습니다.

- `0x00` for PoC-1
- `0x01` for PoC-2
- `0x07` for PoC-3
- `0x09` for PoC-4
- `0x17` for PoC-5
- `0x1c` for PoC-6

[block propagation]: #block-propagation
[state synchronization]: #state-synchronization-aka-fast-sync
[snap protocol]: ./snap.md
[Status]: #status-0x00
[NewBlockHashes]: #newblockhashes-0x01
[Transactions]: #transactions-0x02
[GetBlockHeaders]: #getblockheaders-0x03
[BlockHeaders]: #blockheaders-0x04
[GetBlockBodies]: #getblockbodies-0x05
[BlockBodies]: #blockbodies-0x06
[NewBlock]: #newblock-0x07
[NewPooledTransactionHashes]: #newpooledtransactionhashes-0x08
[GetPooledTransactions]: #getpooledtransactions-0x09
[PooledTransactions]: #pooledtransactions-0x0a
[GetReceipts]: #getreceipts-0x0f
[Receipts]: #receipts-0x10
[RLPx]: ../rlpx.md
[Rinkeby]: https://rinkeby.io
[EIP-155]: https://eips.ethereum.org/EIPS/eip-155
[EIP-1559]: https://eips.ethereum.org/EIPS/eip-1559
[EIP-2124]: https://eips.ethereum.org/EIPS/eip-2124
[EIP-2364]: https://eips.ethereum.org/EIPS/eip-2364
[EIP-2464]: https://eips.ethereum.org/EIPS/eip-2464
[EIP-2481]: https://eips.ethereum.org/EIPS/eip-2481
[EIP-2718]: https://eips.ethereum.org/EIPS/eip-2718
[EIP-2976]: https://eips.ethereum.org/EIPS/eip-2976
[EIP-4938]: https://eips.ethereum.org/EIPS/eip-4938
[London hard fork]: https://github.com/ethereum/execution-specs/blob/master/network-upgrades/mainnet-upgrades/london.md
[Yellow Paper]: https://ethereum.github.io/yellowpaper/paper.pdf