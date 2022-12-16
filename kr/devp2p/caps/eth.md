# Ethereum Wire Protocol (ETH)

'eth'는 이더리움의 교환을 용이하게 하는 [RLPx] 전송 프로토콜입니다. 피어 간의 블록체인 정보. 현재 프로토콜 버전은
**eth/67**입니다. 과거 프로토콜 버전의 변경 사항 목록은 문서 끝 부분을 참조하십시오.

### Basic Operation (기본 동작)

연결이 설정되면 [Status] 메시지를 보내야 합니다. 피어의 상태 메시지 수신 후 Ethereum 세션이 활성화되고 다른
메시지를 보낼 수 있습니다.

세션 내에서 체인 동기화, 블록 전파 및 트랜잭션 교환의 세 가지 상위 수준 작업을 수행할 수 있습니다. 이러한 작업은
연결되지 않은 프로토콜 메시지 세트를 사용하며 클라이언트는 일반적으로 모든 피어 연결에서 동시 활동으로 이를 수행합니다.

클라이언트 구현은 프로토콜 메시지 크기에 제한을 적용해야 합니다. 기본 RLPx 전송은 단일 메시지의 크기를 16.7MiB로
제한합니다. eth 프로토콜의 실제 제한은 더 낮으며 일반적으로 10MiB입니다. 수신된 메시지가 제한보다 크면 피어 연결을
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

### Block Encoding and Validity

Ethereum blocks are encoded as follows:

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

In certain protocol messages, the transaction and ommer lists are relayed together as a
single item called the 'block body'.

    block-body = [transactions, ommers]

The validity of block headers depends on the context in which they are used. For a single
block header, only the validity of the proof-of-work seal (`mix-digest`, `block-nonce`)
can be verified. When a header is used to extend the client's local chain, or multiple
headers are processed in sequence during chain synchronization, the following rules apply:

- Headers must form a chain where block numbers are consecutive and the `parent-hash` of
  each header matches the hash of the preceding header.
- When extending the locally-stored chain, implementations must also verify that the
  values of `difficulty`, `gas-limit` and `time` are within the bounds of protocol rules
  given in the [Yellow Paper].
- The `gas-used` header field must be less than or equal to the `gas-limit`.
- The `basefee-per-gas` header field must be present for blocks after the [London hard
  fork]. Note that `basefee-per-gas` must be absent for earlier blocks. This rule was
  added implicity by [EIP-1559], which added the field into the definition of the Ethereum
  block hash.

For complete blocks, we distinguish between the validity of the block's EVM state
transition, and the (weaker) 'data validity' of the block. The definition of state
transition rules is not dealt with in this specification. We require data validity of the
block for the purposes of immediate [block propagation] and during [state synchronization].

To determine the data validity of a block, use the rules below. Implementations should
disconnect peers sending invalid blocks.

- The block `header` must be valid.
- The `transactions` contained in the block must be valid for inclusion into the chain at
  the block's number. This means that, in addition to the transaction validation rules
  given earlier, validating whether the `tx-type` is permitted at the block number is
  required, and validation of transaction gas must take the block number into account.
- The sum of the `gas-limit`s of all transactions must not exceed the `gas-limit` of the
  block.
- The `transactions` of the block must be verified against the `txs-root` by computing and
  comparing the merkle trie hash of the transactions list.
- The `ommers` list may contain at most two headers.
- `keccak256(ommers)` must match the `ommers-hash` of the block header.
- The headers contained in the `ommers` list must be valid headers. Their block number
  must not be greater than that of the block they are included in. The parent hash of an
  ommer header must refer to an ancestor of depth 7 or less of its including block, and it
  must not have been included in any earlier block contained in this ancestor set.

### Receipt Encoding and Validity

Receipts are the output of the EVM state transition of a block. Like transactions,
receipts have two distinct encodings and we will refer to either encoding using the
identifier `receiptₙ`.

    receipt = {legacy-receipt, typed-receipt}

Untyped, legacy receipts are encoded as follows:

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

[EIP-2718] typed receipts are encoded as RLP byte arrays where the first byte gives the
receipt type (matching `tx-type`) and the remaining bytes are opaque data specific to the
type.

    typed-receipt = tx-type || receipt-data

In the Ethereum Wire Protocol, receipts are always transferred as the complete list of all
receipts contained in a block. It is also assumed that the block containing the receipts
is valid and known. When a list of block receipts is received by a peer, it must be
verified by computing and comparing the merkle trie hash of the list against the
`receipts-root` of the block. Since the valid list of receipts is determined by the EVM
state transition, it is not necessary to define any further validity rules for receipts in
this specification.

## Protocol Messages

In most messages, the first element of the message data list is the `request-id`. For
requests, this is a 64-bit integer value chosen by the requesting peer. The responding
peer must mirror the value in the `request-id` element of the response message.

### Status (0x00)

`[version: P, networkid: P, td: P, blockhash: B_32, genesis: B_32, forkid]`

Inform a peer of its current state. This message should be sent just after the connection
is established and prior to any other eth protocol messages.

- `version`: the current protocol version
- `networkid`: integer identifying the blockchain, see table below
- `td`: total difficulty of the best chain. Integer, as found in block header.
- `blockhash`: the hash of the best (i.e. highest TD) known block
- `genesis`: the hash of the genesis block
- `forkid`: An [EIP-2124] fork identifier, encoded as `[fork-hash, fork-next]`.

This table lists common Network IDs and their corresponding networks. Other IDs exist
which aren't listed, i.e. clients should not require that any particular network ID is
used. Note that the Network ID may or may not correspond with the EIP-155 Chain ID used
for transaction replay prevention.

| ID | chain                         |
|----|-------------------------------|
| 0  | Olympic (disused)             |
| 1  | Frontier (now mainnet)        |
| 2  | Morden (disused)              |
| 3  | Ropsten (current PoW testnet) |
| 4  | [Rinkeby]                     |

For a community curated list of chain IDs, see <https://chainid.network>.

### NewBlockHashes (0x01)

`[[blockhash₁: B_32, number₁: P], [blockhash₂: B_32, number₂: P], ...]`

Specify one or more new blocks which have appeared on the network. To be maximally
helpful, nodes should inform peers of all blocks that they may not be aware of. Including
hashes that the sending peer could reasonably be considered to know (due to the fact they
were previously informed of because that node has itself advertised knowledge of the
hashes through NewBlockHashes) is considered bad form, and may reduce the reputation of
the sending node. Including hashes that the sending node later refuses to honour with a
proceeding [GetBlockHeaders] message is considered bad form, and may reduce the reputation
of the sending node.

### Transactions (0x02)

`[tx₁, tx₂, ...]`

Specify transactions that the peer should make sure is included on its transaction queue.
The items in the list are transactions in the format described in the main Ethereum
specification. Transactions messages must contain at least one (new) transaction, empty
Transactions messages are discouraged and may lead to disconnection.

Nodes must not resend the same transaction to a peer in the same session and must not
relay transactions to a peer they received that transaction from. In practice this is
often implemented by keeping a per-peer bloom filter or set of transaction hashes which
have already been sent or received.

### GetBlockHeaders (0x03)

`[request-id: P, [startblock: {P, B_32}, limit: P, skip: P, reverse: {0, 1}]]`

Require peer to return a BlockHeaders message. The response must contain a number of block
headers, of rising number when `reverse` is `0`, falling when `1`, `skip` blocks apart,
beginning at block `startblock` (denoted by either number or hash) in the canonical chain,
and with at most `limit` items.

### BlockHeaders (0x04)

`[request-id: P, [header₁, header₂, ...]]`

This is the response to GetBlockHeaders, containing the requested headers. The header list
may be empty if none of the requested block headers were found. The number of headers that
can be requested in a single message may be subject to implementation-defined limits.

The recommended soft limit for BlockHeaders responses is 2 MiB.

### GetBlockBodies (0x05)

`[request-id: P, [blockhash₁: B_32, blockhash₂: B_32, ...]]`

This message requests block body data by hash. The number of blocks that can be requested
in a single message may be subject to implementation-defined limits.

### BlockBodies (0x06)

`[request-id: P, [block-body₁, block-body₂, ...]]`

This is the response to GetBlockBodies. The items in the list contain the body data of the
requested blocks. The list may be empty if none of the requested blocks were available.

The recommended soft limit for BlockBodies responses is 2 MiB.

### NewBlock (0x07)

`[block, td: P]`

Specify a single complete block that the peer should know about. `td` is the total
difficulty of the block, i.e. the sum of all block difficulties up to and including this
block.

### NewPooledTransactionHashes (0x08)

`[txhash₁: B_32, txhash₂: B_32, ...]`

This message announces one or more transactions that have appeared in the network and
which have not yet been included in a block. To be maximally helpful, nodes should inform
peers of all transactions that they may not be aware of.

The recommended soft limit for this message is 4096 hashes (128 KiB).

Nodes should only announce hashes of transactions that the remote peer could reasonably be
considered not to know, but it is better to return more transactions than to have a nonce
gap in the pool.

### GetPooledTransactions (0x09)

`[request-id: P, [txhash₁: B_32, txhash₂: B_32, ...]]`

This message requests transactions from the recipient's transaction pool by hash.

The recommended soft limit for GetPooledTransactions requests is 256 hashes (8 KiB). The
recipient may enforce an arbitrary limit on the response (size or serving time), which
must not be considered a protocol violation.

### PooledTransactions (0x0a)

`[request-id: P, [tx₁, tx₂...]]`

This is the response to GetPooledTransactions, returning the requested transactions from
the local pool. The items in the list are transactions in the format described in the main
Ethereum specification.

The transactions must be in same order as in the request, but it is OK to skip
transactions which are not available. This way, if the response size limit is reached,
requesters will know which hashes to request again (everything starting from the last
returned transaction) and which to assume unavailable (all gaps before the last returned
transaction).

It is permissible to first announce a transaction via NewPooledTransactionHashes, but then
to refuse serving it via PooledTransactions. This situation can arise when the transaction
is included in a block (and removed from the pool) in between the announcement and the
request.

A peer may respond with an empty list iff none of the hashes match transactions in its
pool.

### GetReceipts (0x0f)

`[request-id: P, [blockhash₁: B_32, blockhash₂: B_32, ...]]`

Require peer to return a Receipts message containing the receipts of the given block
hashes. The number of receipts that can be requested in a single message may be subject to
implementation-defined limits.

### Receipts (0x10)

`[request-id: P, [[receipt₁, receipt₂], ...]]`

This is the response to GetReceipts, providing the requested block receipts. Each element
in the response list corresponds to a block hash of the GetReceipts request, and must
contain the complete list of receipts of the block.

The recommended soft limit for Receipts responses is 2 MiB.

## Change Log

### eth/67 ([EIP-4938], March 2022)

Version 67 removed the GetNodeData and NodeData messages.

- GetNodeData (0x0d)
  `[request_id: P, [hash_0: B_32, hash_1: B_32, ...]]`
- NodeData (0x0e)
  `[request_id: P, [value_0: B, value_1: B, ...]]`

### eth/66 ([EIP-2481], April 2021)

Version 66 added the `request-id` element in messages [GetBlockHeaders], [BlockHeaders],
[GetBlockBodies], [BlockBodies], [GetPooledTransactions], [PooledTransactions],
GetNodeData, NodeData, [GetReceipts], [Receipts].

### eth/65 with typed transactions ([EIP-2976], April 2021)

When typed transactions were introduced by [EIP-2718], client implementers decided to
accept the new transaction and receipt formats in the wire protocol without increasing the
protocol version. This specification update also added definitions for the encoding of all
consensus objects instead of referring to the Yellow Paper.

### eth/65 ([EIP-2464], January 2020)

Version 65 improved transaction exchange, introducing three additional messages:
[NewPooledTransactionHashes], [GetPooledTransactions], and [PooledTransactions].

Prior to version 65, peers always exchanged complete transaction objects. As activity and
transaction sizes increased on the Ethereum mainnet, the network bandwidth used for
transaction exchange became a significant burden on node operators. The update reduced the
required bandwidth by adopting a two-tier transaction broadcast system similar to block
propagation.

### eth/64 ([EIP-2364], November 2019)

Version 64 changed the [Status] message to include the [EIP-2124] ForkID. This allows
peers to determine mutual compatibility of chain execution rules without synchronizing the
blockchain.

### eth/63 (2016)

Version 63 added the GetNodeData, NodeData, [GetReceipts] and [Receipts] messages
which allow synchronizing transaction execution results.

### eth/62 (2015)

In version 62, the [NewBlockHashes] message was extended to include block numbers
alongside the announced hashes. The block number in [Status] was removed. Messages
GetBlockHashes (0x03), BlockHashes (0x04), GetBlocks (0x05) and Blocks (0x06) were
replaced by messages that fetch block headers and bodies. The BlockHashesFromNumber (0x08)
message was removed.

Previous encodings of the reassigned/removed message codes were:

- GetBlockHashes (0x03): `[hash: B_32, max-blocks: P]`
- BlockHashes (0x04): `[hash₁: B_32, hash₂: B_32, ...]`
- GetBlocks (0x05): `[hash₁: B_32, hash₂: B_32, ...]`
- Blocks (0x06): `[[header, transactions, ommers], ...]`
- BlockHashesFromNumber (0x08): `[number: P, max-blocks: P]`

### eth/61 (2015)

Version 61 added the BlockHashesFromNumber (0x08) message which could be used to request
blocks in ascending order. It also added the latest block number to the [Status] message.

### eth/60 and below

Version numbers below 60 were used during the Ethereum PoC development phase.

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