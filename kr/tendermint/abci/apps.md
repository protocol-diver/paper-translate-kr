---
order: 2
title: Applications
---

### Comment
Source: https://github.com/tendermint/tendermint/blob/main/spec/abci/apps.md <br>
Date: 2024-05-11 (last commit hash is `eed27ad`) <br>

# Applications

먼저 [ABCI Methods and Types](abci.md)에 대한 사양을 읽어보시기 바랍니다.

여기에서는 ABCI 애플리케이션의 다음 구성 요소를 다룹니다:

- [Connection State](#connection-state) - ABCI Connection 과 애플리케이션 상태 간의 상호 작용 및 `CheckTx`와 `DeliverTx` 간의 차이점입니다.
- [Transaction Results](#transaction-results) - 거래 결과 및 유효성에 관한 규칙
- [Validator Set Updates](#validator-updates) - `InitChain` 및 `EndBlock` 중에 검증자 세트가 변경되는 방법
- [Query](#query) - Query 메서드 사용에 대한 표준 및 애플리케이션 상태에 대한 증명
- [Crash Recovery](#crash-recovery) - 핸드셰이크 프로토콜을 사용하여 시작 시 Tendermint 와 애플리케이션을 동기화합니다.
- [State Sync](#state-sync) - 상태 머신 스냅샷 복원을 통한 새 노드의 신속한 부트스트래핑

## Connection State

Tendermint 는 4개의 동시 ABCI Connection 을 유지하므로, 애플리케이션이 각각 별개의 상태를 유지하고 `Commit` 하는 동안 상태를 동기화하는 것이 일반적입니다.

### Concurrency

원칙적으로 4개의 ABCI Connection 은 각각 서로 동시에 작동합니다. 즉, 애플리케이션은 상태에 대한 액세스가 스레드 안전성을 보장해야 합니다. 실제로는 [default in-process ABCI client](https://github.com/tendermint/tendermint/blob/v0.34.4/abci/client/local_client.go#L18)와 [default Go ABCI server](https://github.com/tendermint/tendermint/blob/v0.34.4/abci/server/socket_server.go#L32) 모두 모든 Connection 에서 전역 잠금을 사용하므로 전혀 동시적이지 않습니다. 즉, 앱이 Go로 작성되고 기본 `NewLocalClient`를 사용하여 Tendermint 에서 프로세스 중 컴파일되거나 기본 `SocketServer`를 사용하여 프로세스 외로 실행되는 경우 모든 Connection 의 ABCI 메시지는 선형화(한 번에 하나씩 수신)가 가능합니다.

이 글로벌 mutex 의 존재는 Go 애플리케이션 개발자가 *모든* 읽기 및 쓰기를 ABCI 시스템을 통해 라우팅함으로써 애플리케이션 상태에 대한 스레드 안전성을 확보할 수 있음을 의미합니다. 따라서 애플리케이션 상태를 RPC 인터페이스에 직접 노출하는 것은 *안전하지 않을 수* 있으며, 명시적인 조치를 취하지 않는 한 모든 쿼리는 ABCI 쿼리 메서드를 통해 라우팅되어야 합니다.

### BeginBlock

BeginBlock 요청은 모든 블록이 시작될 때 일부 코드를 실행하는 데 사용할 수 있습니다. 또한 트랜잭션을 전송하기 전에 Tendermint 가 현재 블록 해시와 헤더를 애플리케이션에 전송할 수 있습니다.

앱은 최신 높이와 헤더(즉, 성공적인 Commit 을 실행한 곳)를 기억하여 다시 시작할 때 Tendermin 가 어디에서 시작할지 알려줄 수 있어야 합니다. 아래 Handshake 에 대한 정보를 참조하세요.

### Commit

애플리케이션 상태는 `Commit`하는 동안에만 디스크에 유지되어야 합니다.

`Commit`이 호출되기 전에 Tendermint 는 mempool 을 잠그고 플러시하여 Mempool Connection 에 새 메시지가 수신되지 않도록 합니다. 이렇게 하면 네 가지 Connection 상태를 모두 최신 커밋 상태로 한 번에 안전하게 업데이트할 수 있습니다.

`Commit`이 완료되면 멤풀의 잠금이 해제됩니다.

WARNING: `Commit` 메시지를 처리하는 ABCI 앱 로직이 `/broadcast_tx_sync` 또는 `/broadcast_tx_commit`을 전송하고 응답을 기다린 후 계속 진행하면 교착 상태가 됩니다. 이러한 `broadcast_tx` 호출을 실행하려면 `Commit` 호출 중에 유지되는 잠금을 획득해야 하므로 실행할 수 없습니다. `broadcast_tx` 엔드포인트를 동시에 호출하는 것은 문제가 되지 않으며, 단지 `Commit` 함수의 순차적 로직에 포함될 수 없을 뿐입니다.

### Consensus Connection

Consensus Connection 은 블록 실행을 위한 작업 상태인 `DeliverTxState` 를 유지해야 합니다. 이는 블록 실행 중에 `BeginBlock`, `DeliverTx` 그리고 `EndBlock` 호출에 의해 업데이트되어야 하며, 커밋 중에 "latest committed state" 로 디스크에 `Commit`되어야 합니다.

각 메서드 호출에 의해 `DeliverTxState`에 대한 업데이트는 각 후속 메서드에서 읽을 수 있어야 합니다 - 즉, 업데이트는 선형화할 수 있어야 합니다.

### Mempool Connection

Mempool Connection 은 아직 커밋되지 않은 mempool 의 보류 중인 트랜잭션을 순차적으로 처리하기 위해 `CheckTxState`를 유지해야 합니다. 모든 `Commit`이 끝날 때마다 최신 커밋 상태로 초기화되어야 합니다.

`Commit`을 호출하기 전에 Tendermint 는 Mempool Connection 을 잠그고 플러시하여 기존의 모든 CheckTx 에 응답하고 새로운 Connection 이 시작되지 않도록 합니다. Consensus, Mempool Connection 에서 메시지가 동시에 전송될 수 있으므로 `CheckTxState`는 `DeliverTxState`와 동시에 업데이트될 수 있습니다.

`Commit` 후 mempool 잠금을 유지한 상태에서 블록에 포함된 트랜잭션을 필터링한 후 노드의 로컬 mempool 에 남아 있는 모든 트랜잭션에 대해 CheckTx 가 다시 실행됩니다. 들어오는 트랜잭션이 신규 트랜잭션(`CheckTxType_New`)인지, 재확인 트랜잭션(`CheckTxType_Recheck`)인지를 나타내는 추가 `Type` 파라미터를 CheckTx 함수에 사용할 수 있습니다.

마지막으로, mempool 에서 트랜잭션을 다시 확인한 후 Tendermint 는 Mempool Connection 을 잠금 해제합니다. 새로운 트랜잭션은 다시 한번 CheckTx를 통해 처리할 수 있습니다.

CheckTx 는 유효하지 않은 트랜잭션을 블록체인에서 걸러내는 약한 필터일 뿐이라는 점에 유의하세요. CheckTx 는 트랜잭션 유효성에 영향을 미치는 모든 것을 검사할 필요는 없으며, 비용이 많이 드는 것은 건너뛸 수 있습니다. 비잔틴 노드는 CheckTx 에 신경 쓰지 않고 원한다면 무효 트랜잭션으로 가득 찬 블록을 제안할 수 있기 때문에 약한 필터입니다.

#### Replay Protection

이전 트랜잭션이 재생되는 것을 방지하기 위해 CheckTx 는 재생 방지 기능을 구현해야 합니다.

오래된 트랜잭션이 애플리케이션으로 전송될 수 있습니다. 따라서 이를 처리하기 위한 몇 가지 로직을 구현하는 것이 중요합니다.

### Query Connection

Info Connection 은 사용자의 쿼리에 대한 응답과 Tendermint 가 처음 시작될 때 초기화를 위해 `QueryState`를 유지해야 합니다(둘 다 아래에 자세히 설명되어 있습니다). 여기에는 항상 최신 커밋된 블록과 관련된 최신 커밋 상태가 포함되어야 합니다.

전체 블록이 처리되고 상태가 디스크에 커밋된 후 모든 `Commit`이 끝날 때 `QueryState`를 최신 `DeliverTxState`로 설정해야 합니다. 그렇지 않으면 절대로 수정해서는 안 됩니다.

Tendermint 코어는 현재 Query Connection 을 사용하여 연결 시 IP 주소 또는 노드 ID 에 따라 피어를 필터링합니다. 예를 들어, 다음 쿼리 중 하나에 대해 OK가 아닌 ABCI 응답을 반환하면 Tendermint는 해당 피어에 연결하지 않습니다:

- `p2p/filter/addr/<ip addr>`, 여기서 `<ip addr>`은 IP 주소입니다.
- `p2p/filter/id/<id>`, 여기서 `<is>`는 16진수로 인코딩된 노드 ID (노드 P2P 공개키의 해시)입니다.

참고: 이러한 쿼리 형식은 변경될 수 있습니다!

### Snapshot Connection

Snapshot Connection 은 선택 사항이며, 다른 노드에 상태 동기화 스냅샷을 제공하거나 부트스트랩 중인 로컬 노드로 상태 동기화 스냅샷을 복원하는 데만 사용됩니다.

자세한 내용은 이 문서의 [the state sync section of this document](#state-sync)을 참조하세요.

## Transaction Results

The `Info` and `Log` fields are non-deterministic values for debugging/convenience purposes
that are otherwise ignored.

The `Data` field must be strictly deterministic, but can be arbitrary data.

### Gas

Ethereum introduced the notion of `gas` as an abstract representation of the
cost of resources used by nodes when processing transactions. Every operation in the
Ethereum Virtual Machine uses some amount of gas, and gas can be accepted at a market-variable price.
Users propose a maximum amount of gas for their transaction; if the tx uses less, they get
the difference credited back. Tendermint adopts a similar abstraction,
though uses it only optionally and weakly, allowing applications to define
their own sense of the cost of execution.

In Tendermint, the [ConsensusParams.Block.MaxGas](../proto/types/params.proto) limits the amount of `gas` that can be used in a block.
The default value is `-1`, meaning no limit, or that the concept of gas is
meaningless.

Responses contain a `GasWanted` and `GasUsed` field. The former is the maximum
amount of gas the sender of a tx is willing to use, and the latter is how much it actually
used. Applications should enforce that `GasUsed <= GasWanted` - ie. tx execution
should halt before it can use more resources than it requested.

When `MaxGas > -1`, Tendermint enforces the following rules:

- `GasWanted <= MaxGas` for all txs in the mempool
- `(sum of GasWanted in a block) <= MaxGas` when proposing a block

If `MaxGas == -1`, no rules about gas are enforced.

Note that Tendermint does not currently enforce anything about Gas in the consensus, only the mempool.
This means it does not guarantee that committed blocks satisfy these rules!
It is the application's responsibility to return non-zero response codes when gas limits are exceeded.

The `GasUsed` field is ignored completely by Tendermint. That said, applications should enforce:

- `GasUsed <= GasWanted` for any given transaction
- `(sum of GasUsed in a block) <= MaxGas` for every block

In the future, we intend to add a `Priority` field to the responses that can be
used to explicitly prioritize txs in the mempool for inclusion in a block
proposal. See [#1861](https://github.com/tendermint/tendermint/issues/1861).

### CheckTx

If `Code != 0`, it will be rejected from the mempool and hence
not broadcasted to other peers and not included in a proposal block.

`Data` contains the result of the CheckTx transaction execution, if any. It is
semantically meaningless to Tendermint.

`Events` include any events for the execution, though since the transaction has not
been committed yet, they are effectively ignored by Tendermint.

### DeliverTx

DeliverTx is the workhorse of the blockchain. Tendermint sends the
DeliverTx requests asynchronously but in order, and relies on the
underlying socket protocol (ie. TCP) to ensure they are received by the
app in order. They have already been ordered in the global consensus by
the Tendermint protocol.

If DeliverTx returns `Code != 0`, the transaction will be considered invalid,
though it is still included in the block.

DeliverTx also returns a [Code, Data, and Log](../../proto/abci/types.proto#L189-L191).

`Data` contains the result of the CheckTx transaction execution, if any. It is
semantically meaningless to Tendermint.

Both the `Code` and `Data` are included in a structure that is hashed into the
`LastResultsHash` of the next block header.

`Events` include any events for the execution, which Tendermint will use to index
the transaction by. This allows transactions to be queried according to what
events took place during their execution.

## Updating the Validator Set

The application may set the validator set during InitChain, and may update it during
EndBlock.

Note that the maximum total power of the validator set is bounded by
`MaxTotalVotingPower = MaxInt64 / 8`. Applications are responsible for ensuring
they do not make changes to the validator set that cause it to exceed this
limit.

Additionally, applications must ensure that a single set of updates does not contain any duplicates -
a given public key can only appear once within a given update. If an update includes
duplicates, the block execution will fail irrecoverably.

### InitChain

The `InitChain` method can return a list of validators.
If the list is empty, Tendermint will use the validators loaded in the genesis
file.
If the list returned by `InitChain` is not empty, Tendermint will use its contents as the validator set.
This way the application can set the initial validator set for the
blockchain.

### EndBlock

Updates to the Tendermint validator set can be made by returning
`ValidatorUpdate` objects in the `ResponseEndBlock`:

```protobuf
message ValidatorUpdate {
  tendermint.crypto.keys.PublicKey pub_key
  int64 power
}

message PublicKey {
  oneof {
    ed25519 bytes = 1;
  }
```

The `pub_key` currently supports only one type:

- `type = "ed25519"`

The `power` is the new voting power for the validator, with the
following rules:

- power must be non-negative
- if power is 0, the validator must already exist, and will be removed from the
  validator set
- if power is non-0:
    - if the validator does not already exist, it will be added to the validator
    set with the given power
    - if the validator does already exist, its power will be adjusted to the given power
- the total power of the new validator set must not exceed MaxTotalVotingPower

Note the updates returned in block `H` will only take effect at block `H+2`.

## Consensus Parameters

ConsensusParams enforce certain limits in the blockchain, like the maximum size
of blocks, amount of gas used in a block, and the maximum acceptable age of
evidence. They can be set in InitChain and updated in EndBlock.

### BlockParams.MaxBytes

The maximum size of a complete Protobuf encoded block.
This is enforced by Tendermint consensus.

This implies a maximum transaction size that is this MaxBytes, less the expected size of
the header, the validator set, and any included evidence in the block.

Must have `0 < MaxBytes < 100 MB`.

### BlockParams.MaxGas

The maximum of the sum of `GasWanted` that will be allowed in a proposed block.
This is *not* enforced by Tendermint consensus.
It is left to the app to enforce (ie. if txs are included past the
limit, they should return non-zero codes). It is used by Tendermint to limit the
txs included in a proposed block.

Must have `MaxGas >= -1`.
If `MaxGas == -1`, no limit is enforced.

### EvidenceParams.MaxAgeDuration

This is the maximum age of evidence in time units.
This is enforced by Tendermint consensus.

If a block includes evidence older than this (AND the evidence was created more
than `MaxAgeNumBlocks` ago), the block will be rejected (validators won't vote
for it).

Must have `MaxAgeDuration > 0`.

### EvidenceParams.MaxAgeNumBlocks

This is the maximum age of evidence in blocks.
This is enforced by Tendermint consensus.

If a block includes evidence older than this (AND the evidence was created more
than `MaxAgeDuration` ago), the block will be rejected (validators won't vote
for it).

Must have `MaxAgeNumBlocks > 0`.

### EvidenceParams.MaxNum

This is the maximum number of evidence that can be committed to a single block.

The product of this and the `MaxEvidenceBytes` must not exceed the size of
a block minus it's overhead ( ~ `MaxBytes`).

Must have `MaxNum > 0`.

### Updates

The application may set the ConsensusParams during InitChain, and update them during
EndBlock. If the ConsensusParams is empty, it will be ignored. Each field
that is not empty will be applied in full. For instance, if updating the
Block.MaxBytes, applications must also set the other Block fields (like
Block.MaxGas), even if they are unchanged, as they will otherwise cause the
value to be updated to 0.

#### InitChain

ResponseInitChain includes a ConsensusParams.
If ConsensusParams is nil, Tendermint will use the params loaded in the genesis
file. If ConsensusParams is not nil, Tendermint will use it.
This way the application can determine the initial consensus params for the
blockchain.

#### EndBlock

ResponseEndBlock includes a ConsensusParams.
If ConsensusParams nil, Tendermint will do nothing.
If ConsensusParam is not nil, Tendermint will use it.
This way the application can update the consensus params over time.

Note the updates returned in block `H` will take effect right away for block
`H+1`.

## Query

Query is a generic method with lots of flexibility to enable diverse sets
of queries on application state. Tendermint makes use of Query to filter new peers
based on ID and IP, and exposes Query to the user over RPC.

Note that calls to Query are not replicated across nodes, but rather query the
local node's state - hence they may return stale reads. For reads that require
consensus, use a transaction.

The most important use of Query is to return Merkle proofs of the application state at some height
that can be used for efficient application-specific light-clients.

Note Tendermint has technically no requirements from the Query
message for normal operation - that is, the ABCI app developer need not implement
Query functionality if they do not wish too.

### Query Proofs

The Tendermint block header includes a number of hashes, each providing an
anchor for some type of proof about the blockchain. The `ValidatorsHash` enables
quick verification of the validator set, the `DataHash` gives quick
verification of the transactions included in the block, etc.

The `AppHash` is unique in that it is application specific, and allows for
application-specific Merkle proofs about the state of the application.
While some applications keep all relevant state in the transactions themselves
(like Bitcoin and its UTXOs), others maintain a separated state that is
computed deterministically *from* transactions, but is not contained directly in
the transactions themselves (like Ethereum contracts and accounts).
For such applications, the `AppHash` provides a much more efficient way to verify light-client proofs.

ABCI applications can take advantage of more efficient light-client proofs for
their state as follows:

- return the Merkle root of the deterministic application state in
`ResponseCommit.Data`. This Merkle root will be included as the `AppHash` in the next block.
- return efficient Merkle proofs about that application state in `ResponseQuery.Proof`
  that can be verified using the `AppHash` of the corresponding block.

For instance, this allows an application's light-client to verify proofs of
absence in the application state, something which is much less efficient to do using the block hash.

Some applications (eg. Ethereum, Cosmos-SDK) have multiple "levels" of Merkle trees,
where the leaves of one tree are the root hashes of others. To support this, and
the general variability in Merkle proofs, the `ResponseQuery.Proof` has some minimal structure:

```protobuf
message ProofOps {
  repeated ProofOp ops
}

message ProofOp {
  string type = 1;
  bytes key = 2;
  bytes data = 3;
}
```

Each `ProofOp` contains a proof for a single key in a single Merkle tree, of the specified `type`.
This allows ABCI to support many different kinds of Merkle trees, encoding
formats, and proofs (eg. of presence and absence) just by varying the `type`.
The `data` contains the actual encoded proof, encoded according to the `type`.
When verifying the full proof, the root hash for one ProofOp is the value being
verified for the next ProofOp in the list. The root hash of the final ProofOp in
the list should match the `AppHash` being verified against.

### Peer Filtering

When Tendermint connects to a peer, it sends two queries to the ABCI application
using the following paths, with no additional data:

- `/p2p/filter/addr/<IP:PORT>`, where `<IP:PORT>` denote the IP address and
  the port of the connection
- `p2p/filter/id/<ID>`, where `<ID>` is the peer node ID (ie. the
  pubkey.Address() for the peer's PubKey)

If either of these queries return a non-zero ABCI code, Tendermint will refuse
to connect to the peer.

### Paths

Queries are directed at paths, and may optionally include additional data.

The expectation is for there to be some number of high level paths
differentiating concerns, like `/p2p`, `/store`, and `/app`. Currently,
Tendermint only uses `/p2p`, for filtering peers. For more advanced use, see the
implementation of
[Query in the Cosmos-SDK](https://github.com/cosmos/cosmos-sdk/blob/v0.23.1/baseapp/baseapp.go#L333).

## Crash Recovery

On startup, Tendermint calls the `Info` method on the Info Connection to get the latest
committed state of the app. The app MUST return information consistent with the
last block it succesfully completed Commit for.

If the app succesfully committed block H, then `last_block_height = H` and `last_block_app_hash = <hash returned by Commit for block H>`. If the app
failed during the Commit of block H, then `last_block_height = H-1` and
`last_block_app_hash = <hash returned by Commit for block H-1, which is the hash in the header of block H>`.

We now distinguish three heights, and describe how Tendermint syncs itself with
the app.

```md
storeBlockHeight = height of the last block Tendermint saw a commit for
stateBlockHeight = height of the last block for which Tendermint completed all
    block processing and saved all ABCI results to disk
appBlockHeight = height of the last block for which ABCI app succesfully
    completed Commit

```

Note we always have `storeBlockHeight >= stateBlockHeight` and `storeBlockHeight >= appBlockHeight`
Note also Tendermint never calls Commit on an ABCI app twice for the same height.

The procedure is as follows.

First, some simple start conditions:

If `appBlockHeight == 0`, then call InitChain.

If `storeBlockHeight == 0`, we're done.

Now, some sanity checks:

If `storeBlockHeight < appBlockHeight`, error
If `storeBlockHeight < stateBlockHeight`, panic
If `storeBlockHeight > stateBlockHeight+1`, panic

Now, the meat:

If `storeBlockHeight == stateBlockHeight && appBlockHeight < storeBlockHeight`,
replay all blocks in full from `appBlockHeight` to `storeBlockHeight`.
This happens if we completed processing the block, but the app forgot its height.

If `storeBlockHeight == stateBlockHeight && appBlockHeight == storeBlockHeight`, we're done.
This happens if we crashed at an opportune spot.

If `storeBlockHeight == stateBlockHeight+1`
This happens if we started processing the block but didn't finish.

If `appBlockHeight < stateBlockHeight`
    replay all blocks in full from `appBlockHeight` to `storeBlockHeight-1`,
    and replay the block at `storeBlockHeight` using the WAL.
This happens if the app forgot the last block it committed.

If `appBlockHeight == stateBlockHeight`,
    replay the last block (storeBlockHeight) in full.
This happens if we crashed before the app finished Commit

If `appBlockHeight == storeBlockHeight`
    update the state using the saved ABCI responses but dont run the block against the real app.
This happens if we crashed after the app finished Commit but before Tendermint saved the state.

## State Sync

A new node joining the network can simply join consensus at the genesis height and replay all
historical blocks until it is caught up. However, for large chains this can take a significant
amount of time, often on the order of days or weeks.

State sync is an alternative mechanism for bootstrapping a new node, where it fetches a snapshot
of the state machine at a given height and restores it. Depending on the application, this can
be several orders of magnitude faster than replaying blocks.

Note that state sync does not currently backfill historical blocks, so the node will have a
truncated block history - users are advised to consider the broader network implications of this in
terms of block availability and auditability. This functionality may be added in the future.

For details on the specific ABCI calls and types, see the [methods and types section](abci.md).

### Taking Snapshots

Applications that want to support state syncing must take state snapshots at regular intervals. How
this is accomplished is entirely up to the application. A snapshot consists of some metadata and
a set of binary chunks in an arbitrary format:

- `Height (uint64)`: The height at which the snapshot is taken. It must be taken after the given
  height has been committed, and must not contain data from any later heights.

- `Format (uint32)`: An arbitrary snapshot format identifier. This can be used to version snapshot
  formats, e.g. to switch from Protobuf to MessagePack for serialization. The application can use
  this when restoring to choose whether to accept or reject a snapshot.

- `Chunks (uint32)`: The number of chunks in the snapshot. Each chunk contains arbitrary binary
  data, and should be less than 16 MB; 10 MB is a good starting point.

- `Hash ([]byte)`: An arbitrary hash of the snapshot. This is used to check whether a snapshot is
  the same across nodes when downloading chunks.

- `Metadata ([]byte)`: Arbitrary snapshot metadata, e.g. chunk hashes for verification or any other
  necessary info.

For a snapshot to be considered the same across nodes, all of these fields must be identical. When
sent across the network, snapshot metadata messages are limited to 4 MB.

When a new node is running state sync and discovering snapshots, Tendermint will query an existing
application via the ABCI `ListSnapshots` method to discover available snapshots, and load binary
snapshot chunks via `LoadSnapshotChunk`. The application is free to choose how to implement this
and which formats to use, but must provide the following guarantees:

- **Consistent:** A snapshot must be taken at a single isolated height, unaffected by
  concurrent writes. This can be accomplished by using a data store that supports ACID
  transactions with snapshot isolation.

- **Asynchronous:** Taking a snapshot can be time-consuming, so it must not halt chain progress,
  for example by running in a separate thread.

- **Deterministic:** A snapshot taken at the same height in the same format must be identical
  (at the byte level) across nodes, including all metadata. This ensures good availability of
  chunks, and that they fit together across nodes.

A very basic approach might be to use a datastore with MVCC transactions (such as RocksDB),
start a transaction immediately after block commit, and spawn a new thread which is passed the
transaction handle. This thread can then export all data items, serialize them using e.g.
Protobuf, hash the byte stream, split it into chunks, and store the chunks in the file system
along with some metadata - all while the blockchain is applying new blocks in parallel.

A more advanced approach might include incremental verification of individual chunks against the
chain app hash, parallel or batched exports, compression, and so on.

Old snapshots should be removed after some time - generally only the last two snapshots are needed
(to prevent the last one from being removed while a node is restoring it).

### Bootstrapping a Node

An empty node can be state synced by setting the configuration option `statesync.enabled =
true`. The node also needs the chain genesis file for basic chain info, and configuration for
light client verification of the restored snapshot: a set of Tendermint RPC servers, and a
trusted header hash and corresponding height from a trusted source, via the `statesync`
configuration section.

Once started, the node will connect to the P2P network and begin discovering snapshots. These
will be offered to the local application via the `OfferSnapshot` ABCI method. Once a snapshot
is accepted Tendermint will fetch and apply the snapshot chunks. After all chunks have been
successfully applied, Tendermint verifies the app's `AppHash` against the chain using the light
client, then switches the node to normal consensus operation.

#### Snapshot Discovery

When the empty node join the P2P network, it asks all peers to report snapshots via the
`ListSnapshots` ABCI call (limited to 10 per node). After some time, the node picks the most
suitable snapshot (generally prioritized by height, format, and number of peers), and offers it
to the application via `OfferSnapshot`. The application can choose a number of responses,
including accepting or rejecting it, rejecting the offered format, rejecting the peer who sent
it, and so on. Tendermint will keep discovering and offering snapshots until one is accepted or
the application aborts.

#### Snapshot Restoration

Once a snapshot has been accepted via `OfferSnapshot`, Tendermint begins downloading chunks from
any peers that have the same snapshot (i.e. that have identical metadata fields). Chunks are
spooled in a temporary directory, and then given to the application in sequential order via
`ApplySnapshotChunk` until all chunks have been accepted.

The method for restoring snapshot chunks is entirely up to the application.

During restoration, the application can respond to `ApplySnapshotChunk` with instructions for how
to continue. This will typically be to accept the chunk and await the next one, but it can also
ask for chunks to be refetched (either the current one or any number of previous ones), P2P peers
to be banned, snapshots to be rejected or retried, and a number of other responses - see the ABCI
reference for details.

If Tendermint fails to fetch a chunk after some time, it will reject the snapshot and try a
different one via `OfferSnapshot` - the application can choose whether it wants to support
restarting restoration, or simply abort with an error.

#### Snapshot Verification

Once all chunks have been accepted, Tendermint issues an `Info` ABCI call to retrieve the
`LastBlockAppHash`. This is compared with the trusted app hash from the chain, retrieved and
verified using the light client. Tendermint also checks that `LastBlockHeight` corresponds to the
height of the snapshot.

This verification ensures that an application is valid before joining the network. However, the
snapshot restoration may take a long time to complete, so applications may want to employ additional
verification during the restore to detect failures early. This might e.g. include incremental
verification of each chunk against the app hash (using bundled Merkle proofs), checksums to
protect against data corruption by the disk or network, and so on. However, it is important to
note that the only trusted information available is the app hash, and all other snapshot metadata
can be spoofed by adversaries.

Apps may also want to consider state sync denial-of-service vectors, where adversaries provide
invalid or harmful snapshots to prevent nodes from joining the network. The application can
counteract this by asking Tendermint to ban peers. As a last resort, node operators can use
P2P configuration options to whitelist a set of trusted peers that can provide valid snapshots.

#### Transition to Consensus

Once the snapshots have all been restored, Tendermint gathers additional information necessary for
bootstrapping the node (e.g. chain ID, consensus parameters, validator sets, and block headers)
from the genesis file and light client RPC servers. It also fetches and records the `AppVersion`
from the ABCI application.

Once the state machine has been restored and Tendermint has gathered this additional
information, it transitions to block sync (if enabled) to fetch any remaining blocks up the chain
head, and then transitions to regular consensus operation. At this point the node operates like
any other node, apart from having a truncated block history at the height of the restored snapshot.