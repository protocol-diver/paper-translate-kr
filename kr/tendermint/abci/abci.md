---
order: 1
title: Method and Types
---

### Comment
Source: https://github.com/tendermint/tendermint/blob/main/spec/abci/abci.md <br>
Date: 2024-05-11 (last commit hash is `eed27ad`) <br>

# Methods and Types

## Connections

ABCI 애플리케이션은 Tendermint 상태 머신 복제 엔진과 동일한 프로세스 내에서 실행하거나 상태 머신 복제 엔진과 _별도의_ 프로세스로 실행할 수 있습니다. 동일한 프로세스 내에서 실행되는 경우 Tendermint는 ABCI 애플리케이션 메서드를 Go 메서드 호출로 직접 호출합니다.

Tendermint와 ABCI 애플리케이션이 별도의 프로세스로 실행되는 경우 Tendermint는 ABCI 메서드에 대한 애플리케이션에 대한 네 개의 connection 을 엽니다. 연결은 각각 ABCI 메서드 호출의 하위 집합을 처리합니다. 이러한 하위 집합은 다음과 같이 정의됩니다:

#### **Consensus** connection

* 합의 프로토콜에 의해 구동되며 블록 실행을 담당합니다.
* `InitChain`, `BeginBlock`, `DeliverTx`, `EndBlock`, `Commit` 메서드 호출을 처리합니다.

#### **Mempool** connection

* 새로운 트랜잭션이 공유되거나 블록에 포함되기 전에 트랜잭션의 유효성을 검사합니다.
* `CheckTx` 호출을 처리합니다.

#### **Info** connection

* 초기화 및 사용자의 쿼리를 처리합니다.
* `Info` 및 `Query` 호출을 처리합니다.

#### **Snapshot** connection

* [state sync snapshots](apps.md#state-sync)을 제공하고 복원하는 데 사용됩니다.
* `ListSnapshots`, `LoadSnapshotChunk`, `OfferSnapshot` 및 `ApplySnapshotChunk` 호출을 처리합니다.

또한 모든 connection에서 호출되는 `Flush` 메서드와 디버깅 전용 메서드인 `Echo` 메서드가 있습니다.

connection 전반의 상태 관리에 대한 자세한 내용은 [ABCI Applications](apps.md) 섹션에서 확인할 수 있습니다.

## Errors

`Query`, `CheckTx` 및 `DeliverTx` 메서드에는 `Response*` 에 `Code` 필드가 포함되어 있습니다. 이 필드에는 애플리케이션별 응답 코드가 포함되어야 합니다. 응답 코드가 `0` 이면 오류가 없음을 나타냅니다. 다른 응답 코드는 오류가 발생했음을 Tendermint에 나타냅니다.

이 메서드는 `Codespace` 문자열도 Tendermint에 반환합니다. 이 필드는 애플리케이션의 여러 도메인에서 반환된 `Code` 값을 구분하는 데 사용됩니다. `Codesapce` 는 `Code` 의 네임스페이스입니다.

`Echo`, `Info`, `InitChain`, `BeginBlock`, `EndBlock`, `Commit` 메서드는 오류를 반환하지 않습니다. 이러한 메서드 중 하나에서 오류가 발생하면 텐더민트에서 처리할 수 있는 합리적인 방법이 없는 심각한 문제를 나타냅니다. 이러한 메서드 중 하나에서 오류가 발생하면 애플리케이션이 충돌해야 운영자가 오류를 안전하게 처리할 수 있습니다.

0이 아닌 응답 코드를 처리하는 Tendermint의 방법은 아래에 설명되어 있습니다.

### CheckTx

`CheckTx` ABCI 메서드는 블록에 포함할 트랜잭션을 제어합니다. Tendermint `Code` 가 0이 아닌 `ResponseCheckTx` 를 수신하면 관련 트랜잭션은 Tendermint 멤풀에 추가되지 않거나 이미 포함되어 있는 경우 제거됩니다.

### DeliverTx

`DeliverTx` ABCI 메서드는 Tendermint 에서 애플리케이션으로 트랜잭션을 전달합니다. Tendermint 가 0이 아닌 `Code` 가 포함된 `ResponseDeliverTx` 를 수신하면 응답 코드가 기록됩니다. 트랜잭션은 이미 블록에 포함되었으므로 `Code` 는 Tendermint 합의에 영향을 미치지 않습니다.

### Query

`Query` ABCI 메서드는 애플리케이션에 애플리케이션 상태에 대한 정보를 쿼리합니다. Tendermint가 0이 아닌 `Code` 가 포함된 응답 쿼리를 받으면 이 코드를 쿼리를 시작한 클라이언트에 직접 반환됩니다.

## Events

`CheckTx`, `BeginBlock`, `DeliverTx`, `EndBlock` 메서드는 `Response*` 에 `Events` 필드를 포함합니다. 애플리케이션은 일련의 이벤트로 이러한 ABCI 메서드에 응답할 수 있습니다. 이벤트를 통해 애플리케이션은 ABCI 메서드 실행에 대한 메타데이터를 해당 메타데이터와 관련된 트랜잭션 및 블록에 연결할 수 있습니다. 이러한 ABCI 메서드를 통해 반환되는 이벤트는 어떤 방식으로든 Tendermint 컨센서스에 영향을 미치지 않으며, Tendermint 상태의 구독 및 쿼리를 지원하기 위해 존재합니다.

`Event` 에는 메서드 실행 중에 일어난 일에 대한 메타데이터를 나타내는 키-값 문자열 쌍인 `tyep` 과 `EventAttributes` 목록이 포함됩니다. `Event` 값은 트랜잭션과 블록이 실행되는 동안 일어난 일에 따라 트랜잭션과 블록을 색인하는 데 사용할 수 있습니다. `BeginBlock` 과 `EndBlock` 에서 블록에 대해 반환되는 이벤트 세트는 병합된다는 점에 유의하세요. 두 메서드가 동일한 키를 반환하는 경우 `EndBlock` 에 정의된 값만 사용됩니다.

각 이벤트에는 특정 `Response*` 또는 `Tx` 에 대한 이벤트를 분류하기 위한 `type` 이 있습니다. `Response*` 또는 `Tx` 에는 `type` 값이 중복되는 여러 이벤트가 포함될 수 있으며, 각 고유 항목은 특정 이벤트의 속성을 분류하기 위한 것입니다. 이벤트 속성의 모든 키와 값은 이벤트 유형 자체와 함께 UTF-8로 인코딩된 문자열이어야 합니다.

```protobuf
message Event {
  string                  type       = 1;
  repeated EventAttribute attributes = 2;
}
```

`Event` 의 속성은 `key`, `value`, `index` 플래그로 구성됩니다. `index` 플래그는 Tendermint 인덱서에게 속성을 인덱싱하도록 알립니다. `index` 플래그의 값은 비결정적이며 네트워크의 여러 노드에 따라 다를 수 있습니다.

```protobuf
message EventAttribute {
  bytes key   = 1;
  bytes value = 2;
  bool  index = 3;  // nondeterministic
}
```

예시:

```go
 abci.ResponseDeliverTx{
  // ...
 Events: []abci.Event{
  {
   Type: "validator.provisions",
   Attributes: []abci.EventAttribute{
    abci.EventAttribute{Key: []byte("address"), Value: []byte("..."), Index: true},
    abci.EventAttribute{Key: []byte("amount"), Value: []byte("..."), Index: true},
    abci.EventAttribute{Key: []byte("balance"), Value: []byte("..."), Index: true},
   },
  },
  {
   Type: "validator.provisions",
   Attributes: []abci.EventAttribute{
    abci.EventAttribute{Key: []byte("address"), Value: []byte("..."), Index: true},
    abci.EventAttribute{Key: []byte("amount"), Value: []byte("..."), Index: false},
    abci.EventAttribute{Key: []byte("balance"), Value: []byte("..."), Index: false},
   },
  },
  {
   Type: "validator.slashed",
   Attributes: []abci.EventAttribute{
    abci.EventAttribute{Key: []byte("address"), Value: []byte("..."), Index: false},
    abci.EventAttribute{Key: []byte("amount"), Value: []byte("..."), Index: true},
    abci.EventAttribute{Key: []byte("reason"), Value: []byte("..."), Index: true},
   },
  },
  // ...
 },
}
```

## EvidenceType

Tendermint 의 보안 모델은 "evidence" 사용에 의존합니다. Evidence 는 네트워크 참여자의 악의적인 행동에 대한 증거입니다. 이러한 악의적인 행동을 탐지하는 것은 Tendermint 의 책임입니다. 악의적인 행동이 감지되면, Tendermint 는 해당 행동에 대한 증거를 다른 노드에 가십하고 모든 검증자가 검증한 후 체인에 증거를 커밋합니다. 그런 다음 이 증거는 ABCI를 통해 애플리케이션에 전달됩니다. Evidence 를 처리하고 처벌을 행사하는 것은 애플리케이션의 책임입니다.

EvidenceType 에는 다음과 같은 프로토타입 형식이 있습니다:

```proto
enum EvidenceType {
  UNKNOWN               = 0;
  DUPLICATE_VOTE        = 1;
  LIGHT_CLIENT_ATTACK   = 2;
}
```

Evidence 에는 두 가지 형태가 있습니다: 중복 투표와 라이트 클라이언트 공격입니다. 자세한 내용은 [data structures](https://github.com/tendermint/tendermint/blob/v0.34.x/spec/core/data_structures.md) 또는 [accountability](https://github.com/tendermint/tendermint/blob/v0.34.x/spec/light-client/accountability/)에서 확인할 수 있습니다.

## Determinism

ABCI 애플리케이션은 Tendermint 합의 엔진에 의해 안전하게 복제되기 위해 결정론적 유한 상태 머신을 구현해야 합니다. 즉, 합의 연결을 통한 블록 실행은 엄격하게 결정론적이어야 합니다. 동일한 순서의 요청 세트가 주어지면 모든 노드는 모든 BeginBlock, DeliverTx, EndBlock 및 Commit에 대해 동일한 응답을 계산합니다. 이는 매우 중요한데, 응답이 머클 루트를 통해 또는 직접적으로 다음 블록의 헤더에 포함되므로 모든 노드가 정확히 일치해야 하기 때문입니다.

따라서 애플리케이션은 Tendermint 코어와 같은 합의 엔진에 대한 ABCI 연결을 제외하고는 외부 사용자나 프로세스에 노출되지 않도록 하는 것이 좋습니다. 애플리케이션은 다른 종류의 요청이 아닌 블록 실행(BeginBlock, DeliverTx, EndBlock, Commit)의 입력에 의해서만 상태를 변경해야 합니다. 이것이 모든 노드가 동일한 트랜잭션을 보고 동일한 결과를 계산할 수 있는 유일한 방법입니다.

상태 머신에 비결정성이 있는 경우, 블록 헤더의 올바른 값에 대해 노드가 동의하지 않기 때문에 결국 합의에 실패하게 됩니다. 비결정성을 수정하고 노드를 다시 시작해야 합니다.

애플리케이션에서 비결정성의 원인은 다음과 같습니다:

* 하드웨어 오류
    * 우주선, 과열, 등
* 노드 종속 상태
    * 난수
    * 시간
* 사양 미달
    * 라이브러리 버전 변경
    * Race conditions
    * Floating point numbers
    * JSON serialization
    * Iterating through hash-tables/maps/dictionaries
* 외부 요인
    * Filesystem
    * Network calls (eg. 일부 외부 REST API service)

원래 논의 내용은 [#56](https://github.com/tendermint/abci/issues/56)을 참조하세요.

일부 메서드(`Query`, `CheckTx`, `DeliverTx`)는 명시적으로 비결정적인 데이터를 `Info` 및 `Log` 필드의 형태로 반환합니다. `Log` 는 애플리케이션 로거의 리터럴 출력을 위한 것이고, `Info` 는 반환해야 하는 추가 정보입니다. 이 필드는 블록 헤더 계산에 포함되지 않는 유일한 필드이므로 동의가 필요하지 않습니다. `Response*` 의 다른 모든 필드는 엄격하게 결정론적이어야 합니다.

## Block Execution

새로운 블록체인이 처음 시작될 때, Tendermint 는 `InitChain` 을 호출합니다. 이후 각 블록에 대해 다음과 같은 메서드 순서가 실행됩니다:

`BeginBlock, [DeliverTx], EndBlock, Commit`

여기서 블록의 각 트랜잭션에 대해 하나의 `DeliverTx` 가 호출됩니다. 그 결과 애플리케이션 상태가 업데이트됩니다. DeliverTx, EndBlock, Commit 의 결과에 대한 암호화 커미트먼트은 다음 블록의 헤더에 포함됩니다.

## State Sync

State Sync 를 사용하면 새 노드가 과거 블록을 재생하는 대신 상태 머신 스냅샷을 검색, 가져와서 적용함으로써 빠르게 부트스트랩할 수 있습니다. 자세한 내용은 [state sync section](../spec/p2p/messages/state-sync.md)을 참조하세요.

새 노드는 P2P 네트워크의 다른 노드에서 스냅샷을 발견하고 요청합니다. 피어로부터 스냅샷 요청을 받은 Tendermint 노드는 애플리케이션에서 `ListSnapshots` 를 호출하여 로컬 상태 스냅샷을 검색합니다. 피어로부터 스냅샷을 받은 후, 새 노드는 `OfferSnapshot` 메서드를 통해 피어로부터 받은 각 스냅샷을 로컬 애플리케이션에 제공합니다.

스냅샷은 상당히 클 수 있으므로 전체 스냅샷으로 조립할 수 있는 작은 "chunks" 로 나뉩니다. 애플리케이션이 스냅샷을 수락하고 복원을 시작하면 Tendermint 는 기존 노드에서 스냅샷 "chunks" 를 가져옵니다. "chunks" 를 제공하는 노드는 `LoadSnapshotChunk` 메서드를 사용하여 로컬 애플리케이션에서 "chunks" 를 가져옵니다.

새 노드가 "chunks" 를 받으면 `ApplySnapshotChunk` 를 사용하여 로컬 애플리케이션에 순차적으로 적용합니다. 모든 chunks 가 적용되면 `Info` 쿼리를 통해 애플리케이션 `AppHash` 가 검색됩니다. 그런 다음 `AppHash` 는 [light client verification](../spec/light-client/verification/README.md) 검증을 통해 확인된 블록체인의 `AppHash` 와 비교됩니다.

## Messages

### Echo

* **Request**:
    * `Message (string)`: 다시 에코할 문자열
* **Response**:
    * `Message (string)`: 입력 문자열
* **Usage**:
    * 문자열 에코를 통해 abci 클라이언트/서버 구현 테스트

### Flush

* **Usage**:
    * 클라이언트에 대기 중인 메시지를 서버로 플러시해야 한다는 신호를 보냅니다. 비동기 요청이 실제로 전송되었는지 확인하기 위해 클라이언트 구현에서 주기적으로 호출되며, 동기 요청을 하기 위해 즉시 호출되어 플러시 응답이 돌아오면 반환됩니다.

### Info

* **Request**:

    | Name          | Type   | Description                              | Field Number |
    |---------------|--------|----------------------------------------- |--------------|
    | version       | string | Tendermint 소프트웨어 시맨틱 버전             | 1            |
    | block_version | uint64 | Tendermint 블록 프로토콜 버전                | 2            |
    | p2p_version   | uint64 | Tendermint P2P 프로토콜 버전                | 3            |
    | abci_version  | string | Tendermint ABCI 시맨틱 버전                | 4            |

* **Response**:
  
    | Name                | Type   | Description                                      | Field Number |
    |---------------------|--------|--------------------------------------------------|--------------|
    | data                | string | 일부 임의 정보                                      | 1            |
    | version             | string | 애플리케이션 소프트웨어 시맨틱 버전                       | 2            |
    | app_version         | uint64 | 애플리케이션 프로토콜 버전                             | 3            |
    | last_block_height   | int64  | 앱이 커밋을 호출한 가장 최근 블록                       | 4            |
    | last_block_app_hash | bytes  | 최근 커밋 결과                                      | 5            |

* **Usage**:
    * 애플리케이션 상태에 대한 정보를 반환합니다.
    * 시작 시 발생하는 핸드셰이크 중에 Tendermint 와 애플리케이션을 동기화하는 데 사용됩니다.
    * 반환된 `app_version` 은 모든 블록의 헤더에 포함됩니다.
    * Tendermint 는 `last_block_app_hash` 와 `last_block_height` 가 `Commit` 중에 업데이트되어 동일한 블록 높이에 대해 `Commit` 이 두 번 호출되지 않도록 합니다.

> 참고: 시맨틱 버전은 [semantic versioning](https://semver.org/)에 대한 참조입니다. 정보에서 시맨틱 버전은 X.X.x로 표시됩니다.

### InitChain

* **Request**:

    | Name             | Type                                                                                                                                 | Description                                         | Field Number |
    |------------------|--------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------|--------------|
    | time             | [google.protobuf.Timestamp](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#google.protobuf.Timestamp) | Genesis time                                        | 1            |
    | chain_id         | string                                                                                                                               | 블록체인의 ID.                               | 2            |
    | consensus_params | [ConsensusParams](#consensusparams)                                                                                                  | 초기 합의에 중요한 매개변수.              | 3            |
    | validators       | repeated [ValidatorUpdate](#validatorupdate)                                                                                         | 투표권을 기준으로 정렬된 초기 제네시스 검증인.	 | 4            |
    | app_state_bytes  | bytes                                                                                                                                | 직렬화된 초기 애플리케이션 상태입니다. JSON bytes.   | 5            |
    | initial_height   | int64                                                                                                                                | 초기 블록의 높이(일반적으로 `1`)입니다.	        | 6            |

* **Response**:

    | Name             | Type                                         | Description                                     | Field Number |
    |------------------|----------------------------------------------|-------------------------------------------------|--------------|
    | consensus_params | [ConsensusParams](#consensusparams)          | 초기 합의에 중요한 매개변수(선택 사항) | 1            |
    | validators       | repeated [ValidatorUpdate](#validatorupdate) | 초기 검증인 세트(선택 사항).	               | 2            |
    | app_hash         | bytes                                        | 초기 애플리케이션 해시.                       | 3            |

* **Usage**:
    * 생성 시 한 번 호출됩니다.
    * ResponseInitChain.Validators 가 비어 있으면, 초기 검증자 세트는 RequestInitChain.Validators 가 됩니다.
    * ResponseInitChain.Validators 가 비어 있지 않은 경우, 초기 검증자 세트가 됩니다 (RequestInitChain.Validators 에 있는 항목과 상관없이).
    * 이를 통해 앱은 Tendermint 가 제안한 초기 검증자 세트(즉, 제네시스 파일)를 사용할지 아니면 다른 검증자 세트(제네시스 파일의 일부 애플리케이션 특정 정보에 따라 계산된 것)를 사용할지 결정할 수 있습니다.

### Query

* **Request**:
  
    | Name   | Type   | Description                                                                                                                                                                                                                                                                            | Field Number |
    |--------|--------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|
    | data   | bytes  | Raw query bytes. Path와 함께 또는 대신 사용할 수 있습니다.	                                                                                                                                                                                                                                  | 1            |
    | path   | string | 요청 URI의 경로 필드입니다. `data` 와 함께 또는 `data` 대신 사용할 수 있습니다. 앱은 `/store` 를 기본 스토어의 키별 쿼리로 해석해야 합니다. 키는 `data` 필드에 지정되어야 (SHOULD) 합니다. 앱은 `/accounts/...` 또는 `/votes/...` 와 같은 특정 유형에 대한 쿼리를 허용해야 (SHOULD) 합니다.	 | 2            |
    | height | int64  | 쿼리하려는 블록 높이(default=0 은 가장 최근에 커밋된 블록의 데이터를 반환합니다). 이 값은 애플리케이션의 머클 루트 해시가 포함된 블록의 높이로, 블록을 Height-1로 커밋한 후의 상태를 나타냅니다.	| 3            |
    | prove  | bool   | 가능한 경우 응답과 함께 머클 증명을 반환합니다.	 | 4            |

* **Response**:

    | Name      | Type                  | Description                                                                                                                                                                                                        | Field Number |
    |-----------|-----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|
    | code      | uint32                | Response code.                                                                                                                                                                                                     | 1            |
    | log       | string                | 애플리케이션 로거의 출력입니다. **비결정적일 수 있습니다.**                                                                                                                                              | 3            |
    | info      | string                | 추가 정보. **비결정적일 수 있습니다.**                                                                                                                                                              | 4            |
    | index     | int64                 | The index of the key in the tree.                                                                                                                                                                                  | 5            |
    | key       | bytes                 | The key of the matching data.                                                                                                                                                                                      | 6            |
    | value     | bytes                 | The value of the matching data.                                                                                                                                                                                    | 7            |
    | proof_ops | [ProofOps](#proofops) | 요청이 있는 경우 주어진 Height에 대한 `app_hash` 와 비교하여 검증할 값 데이터에 대한 직렬화된 증명입니다.	                       | 8            |
    | height    | int64                 | 데이터가 파생된 블록 높이입니다. 애플리케이션의 머클 루트 해시가 포함된 블록의 높이로, 블록을 Height-1로 커밋한 후의 상태를 나타냅니다.	 | 9            |
    | codespace | string                | `code` 의 네임스페이스.                                                                                                                                                                                          | 10           |

* **Usage**:
    * 현재 또는 과거 높이의 애플리케이션에서 데이터를 쿼리합니다.
    * 선택적으로 머클 증명을 반환합니다.
    * 머클 증명에는 다양한 유형의 머클 트리와 인코딩 형식을 지원하기 위한 자체 설명 `type` 필드가 포함되어 있습니다.

### BeginBlock

* **Request**:

    | Name                 | Type                                        | Description                                                                                                       | Field Number |
    |----------------------|---------------------------------------------|-------------------------------------------------------------------------------------------------------------------|--------------|
    | hash                 | bytes                                       | 블록의 해시입니다. 블록 헤더에서 유추할 수 있습니다.	                                                      | 1            |
    | header               | [Header](../core/data_structures.md#header) | 블록의 헤더.                                                                                                 | 2            |
    | last_commit_info     | [LastCommitInfo](#lastcommitinfo)           | 라운드를 포함한 마지막 커밋에 대한 정보와 검증자 목록 및 마지막 블록에 서명한 검증자 목록입니다.	 | 3            |
    | byzantine_validators | repeated [Evidence](#evidence)              | 악의적으로 행동한 검증자의 evidence 목록입니다.	                                                            | 4            |

* **Response**:

    | Name   | Type                      | Description                         | Field Number |
    |--------|---------------------------|-------------------------------------|--------------|
    | events | repeated [Event](#events) | 인덱싱을 위한 유형 & 키-값 이벤트	 | 1           |

* **Usage**:
    * 새 블록의 시작을 알립니다.
    * `DeliverTx` 메서드 호출 전에 호출됩니다.
    * 헤더에는 높이, 타임스탬프 등이 포함되며, 이는 Tendermint 블록 헤더와 정확히 일치합니다. 향후 이를 일반화할 수 있습니다.
    * `LastCommitInfo` 와 `ByzantineValidators` 는 검증자에 대한 보상과 처벌을 결정하는 데 사용할 수 있습니다.

### CheckTx

* **Request**:

    | Name | Type        | Description                                                                                                                                                                                                                                         | Field Number |
    |------|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|
    | tx   | bytes       | 요청 트랜잭션 바이트 bytes                                                                                                                                                                                                                       | 1            |
    | type | CheckTxType | `CheckTx_New` 또는 `CheckTx_Recheck` 중 하나입니다. `CheckTx_New` 가 기본값이며 트랜잭션의 전체 확인이 필요하다는 의미입니다. 멤풀이 트랜잭션의 정상적인 재검사를 시작할 때 `CheckTx_Recheck` 유형이 사용됩니다.	             | 2            |

* **Response**:

    | Name       | Type                      | Description                                                           | Field Number |
    |------------|---------------------------|-----------------------------------------------------------------------|--------------|
    | code       | uint32                    | 응답 코드.                                                        | 1            |
    | data       | bytes                     | 결과 바이트(있는 경우).                                                 | 2            |
    | log        | string                    | 애플리케이션 로거의 출력입니다. **비결정적일 수 있습니다.** | 3            |
    | info       | string                    | 추가 정보. **비결정적일 수 있습니다.**                 | 4            |
    | gas_wanted | int64                     | 트랜잭션이 필요로 하는 가스량입니다.	                              | 5            |
    | gas_used   | int64                     | 트랜잭션별로 소비된 가스 양입니다.	                                | 6            |
    | events     | repeated [Event](#events) | 트랜잭션 인덱싱을 위한 유형 & 키-값 이벤트(e.g. 계정별).	   | 7            |
    | codespace  | string                    | `code` 의 네임스페이스.                                             | 8            |
    | sender     | string                    | 거래 발신자(예: 서명자)	                            | 9            |
    | priority   | int64                     | 트랜잭션의 우선순위(mempool ordering 의 경우)	                     | 10           |

* **Usage**:
    * 기술적으로 선택 사항이며 블록 처리에는 관여하지 않습니다.
    * 멤풀의 가디언: 모든 노드는 트랜잭션을 로컬 멤풀에 허용하기 전에 `CheckTx` 를 실행합니다.
    * 트랜잭션은 외부 사용자 또는 다른 노드로부터 올 수 있습니다.
    * `CheckTx` 는 애플리케이션의 현재 상태(예: 서명 및 계정 잔액 확인)에 대해 트랜잭션의 유효성을 검사하지만, 트랜잭션에 설명된 상태 변경 사항은 적용하지 않습니다.
    가상 머신에서 코드를 실행하지 않음.
    * `ResponseCheckTx.Code != 0` 인 트랜잭션은 거부되며, 다른 노드에 브로드캐스트되거나 제안 블록에 포함되지 않습니다.
    * Tendermint 는 응답 코드에 다른 값을 부여하지 않습니다.

### DeliverTx

* **Request**:

    | Name | Type  | Description                    | Field Number |
    |------|-------|--------------------------------|--------------|
    | tx   | bytes | 요청 트랜잭션 바이트입니다.	 | 1            |

* **Response**:

    | Name       | Type                      | Description                                                           | Field Number |
    |------------|---------------------------|-----------------------------------------------------------------------|--------------|
    | code       | uint32                    | 응답 코드.	                                                        | 1            |
    | data       | bytes                     | 결과 바이트(있는 경우).                                                 | 2            |
    | log        | string                    | 애플리케이션 로거의 출력입니다. **비결정적일 수 있습니다.** | 3            |
    | info       | string                    | 추가 정보. **비결정적일 수 있습니다.**                 | 4            |
    | gas_wanted | int64                     | 트랜잭션이 필요로 하는 가스량입니다.                              | 5            |
    | gas_used   | int64                     | 트랜잭션별로 소비된 가스 양입니다.                                | 6            |
    | events     | repeated [Event](#events) | 트랜잭션 인덱싱을 위한 유형 & 키-값 이벤트(e.g. 계정별).   | 7            |
    | codespace  | string                    | `code` 의 네임스페이스.                                             | 8            |

* **Usage**:
    * [**Required**] 애플리케이션의 핵심 메서드입니다.
    * `DeliverTx` 가 호출되면 애플리케이션은 트랜잭션을 완전히 실행한 후 Tendermint 에 제어권을 반환해야 합니다.
    * 트랜잭션이 완전히 유효한 경우에만 `ResponseDeliverTx.Code == 0` 을 반환합니다.
### EndBlock

* **Request**:

    | Name   | Type  | Description                        | Field Number |
    |--------|-------|------------------------------------|--------------|
    | height | int64 | 방금 실행된 블록의 높이입니다. | 1            |

* **Response**:

    | Name                    | Type                                         | Description                                                     | Field Number |
    |-------------------------|----------------------------------------------|-----------------------------------------------------------------|--------------|
    | validator_updates       | repeated [ValidatorUpdate](#validatorupdate) | 검증자 세트 변경(투표권을 0으로 설정하여 제거).	     | 1            |
    | consensus_param_updates | [ConsensusParams](#consensusparams)          | 합의에 중요한 time, size 및 기타 매개변수에 대한 변경 사항.	 | 2            |
    | events                  | repeated [Event](#events)                    | 인덱싱을 위한 유형 & 키-값 이벤트	                            | 3            |

* **Usage**:
    * 블록의 끝을 알립니다.
    * 현재 블록의 모든 트랜잭션이 전달된 후 블록의 `Commit` 메시지가 전송되기 전에 호출됩니다.
    * 블록 `H` 에 의해 트리거되는 선택적 `validator_updates`. 이러한 업데이트는 블록 `H+1`, `H+2`, `H+3`에 대한 유효성 검사에 영향을 줍니다.
    * 검증자 업데이트 이후의 높이는 다음과 같은 방식으로 영향을 받습니다:
        * `H+1`: `NextValidatorsHash` 에 새로운 `validator_updates` 값이 포함됩니다.
        * `H+2`: 검증자 세트 변경이 적용되고 `ValidatorsHash` 가 업데이트됩니다.
        * `H+3`: 변경된 검증자 집합을 포함하도록 `LastCommitInfo` 가 변경됩니다.
    * 블록 `H`에 대해 반환된 `consensus_param_updates` 는 블록 `H+1` 의 합의 파라미터에 적용됩니다. 합의 파라미터에 대한 자세한 내용은 [application spec entry on consensus parameters](../spec/abci/apps.md#consensus-parameters)을 참조하세요.

### Commit

* **Request**:

    | Name   | Type  | Description                        | Field Number |
    |--------|-------|------------------------------------|--------------|

    커밋은 애플리케이션 상태가 유지되도록 애플리케이션에 신호를 보냅니다. 매개 변수가 필요하지 않습니다.

* **Response**:

    | Name          | Type  | Description                                                            | Field Number |
    |---------------|-------|------------------------------------------------------------------------|--------------|
    | data          | bytes | 애플리케이션 상태의 머클 루트 해시입니다.	                         | 2            |
    | retain_height | int64 | 이 높이 미만의 블록은 제거될 수 있습니다. 기본값은 `0` (모두 유지)입니다.	 | 3            |

* **Usage**:
    * 애플리케이션 상태를 유지하도록 애플리케이션에 신호를 보냅니다.
    * 애플리케이션 상태의 머클 루트 해시(선택 사항)를 반환합니다.
    * `ResponseCommit.Data` 는 다음 블록의 `Header.AppHash` 로 포함됩니다.
        * 비어 있을 수 있습니다.
    * 나중에 `Query` 를 호출하면 이 머클 루트 해시에 앵커링된 애플리케이션 상태에 대한 증명을 반환할 수 있습니다.
    * 개발자는 여기에 원하는 모든 것을 반환할 수 있지만(아무것도 아닐 수도 있고 상수 문자열 등), 결정론적이기만 하면 됩니다(BeginBlock/DeliverTx/EndBlock 메서드에서 오지 않은 함수가 아니어야 합니다).
    * `RetainHeight` 는 주의해서 사용하세요! 네트워크의 모든 노드가 히스토리 블록을 제거하면 이 데이터는 영구적으로 손실되며 새로운 노드가 네트워크에 가입하여 부트스트랩할 수 없습니다. 기록 블록은 감사, 지속되지 않는 높이의 재생, 가벼운 클라이언트 확인 등 다른 목적으로도 필요할 수 있습니다.

### ListSnapshots

* **Request**:

    | Name   | Type  | Description                        | Field Number |
    |--------|-------|------------------------------------|--------------|

    애플리케이션에 스냅샷 목록을 요청하는 빈 요청입니다.

* **Response**:

    | Name      | Type                           | Description                    | Field Number |
    |-----------|--------------------------------|--------------------------------|--------------|
    | snapshots | repeated [Snapshot](#snapshot) | 로컬 상태 스냅샷 목록입니다. | 1            |

* **Usage**:
    * 상태 동기화 중에 피어에서 사용 가능한 스냅샷을 검색하는 데 사용됩니다.
    * 자세한 내용은 `Snapshot` 데이터 유형을 참조하세요.

### LoadSnapshotChunk

* **Request**:

    | Name   | Type   | Description                                                           | Field Number |
    |--------|--------|-----------------------------------------------------------------------|--------------|
    | height | uint64 | chunk 가 속한 스냅샷의 높이입니다.                     | 1            |
    | format | uint32 | chunk 가 속한 스냅샷의 애플리케이션별 형식입니다. | 2            |
    | chunk  | uint32 | 초기 chunk 의 `0`부터 시작하는 chunk 인덱스입니다.             | 3            |

* **Response**:

    | Name  | Type  | Description                                                                                                                                           | Field Number |
    |-------|-------|-------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|
    | chunk | bytes | 바이너리 chunk 콘텐츠, 배열 형식입니다. chunk 메시지는 _메타데이터 포함_ 16MB를 초과할 수 없으므로 10MB가 좋은 출발점입니다. | 1            |

* **Usage**:
    * 상태 동기화 중에 피어에서 스냅샷 chunk 를 검색하는 데 사용됩니다.

### OfferSnapshot

* **Request**:

    | Name     | Type                  | Description                                                              | Field Number |
    |----------|-----------------------|--------------------------------------------------------------------------|--------------|
    | snapshot | [Snapshot](#snapshot) | 복원을 위해 제공되는 스냅샷입니다.                                    | 1            |
    | app_hash | bytes                 | 블록체인에서 이 높이에 대해 라이트 클라이언트가 검증한 app hash 입니다. | 2            |

* **Response**:

    | Name   | Type              | Description                       | Field Number |
    |--------|-------------------|-----------------------------------|--------------|
    | result | [Result](#result) | 스냅샷 오퍼의 결과입니다. | 1            |

#### Result

```proto
  enum Result {
    UNKNOWN       = 0;  // 결과를 알 수 없음, 모든 스냅샷 복원 중단
    ACCEPT        = 1;  // 스냅샷이 수락되면 chunk 적용을 시작합니다.
    ABORT         = 2;  // 스냅샷 복원을 중단하고 다른 스냅샷을 시도하지 마세요.
    REJECT        = 3;  // 해당 스냅샷을 거부하고 다른 스냅샷을 시도하세요.
    REJECT_FORMAT = 4;  // 이 'format' 의 모든 스냅샷을 거부하고 다른 스냅샷을 시도하세요.
    REJECT_SENDER = 5;  // 이 스냅샷의 모든 발신자의 모든 스냅샷을 거부하고 다른 스냅샷을 시도하세요.
  }
```

* **Usage**:
    * `OfferSnapshot` 은 상태 동기화를 사용하여 노드를 부트스트랩할 때 호출됩니다. 애플리케이션은 스냅샷을 적절히 수락하거나 거부할 수 있습니다. 수락하면 Tendermint 는 `ApplySnapshotChunk` 를 통해 스냅샷 chunk 를 검색하고 적용합니다. 애플리케이션은 chunk 응답에서 스냅샷을 거부하도록 선택할 수도 있으며, 이 경우 추가 `OfferSnapshot` 호출을 수락할 준비가 되어 있어야 합니다.
    * 라이트 클라이언트에 의해 확인된 `AppHash` 만 신뢰할 수 있습니다. 다른 모든 데이터는 공격자가 스푸핑할 수 있으므로 애플리케이션은 DoS 공격을 피하기 위해 추가 검증 체계를 사용해야 합니다. 확인된 `AppHash` 는 스냅샷 복원이 끝날 때 복원된 애플리케이션에 대해 자동으로 확인됩니다.
    * 자세한 내용은 `Snapshot` 데이터 유형 또는 [state sync section](../spec/p2p/messages/state-sync.md)을 참조하세요.

### ApplySnapshotChunk

* **Request**:

    | Name   | Type   | Description                                                                 | Field Number |
    |--------|--------|-----------------------------------------------------------------------------|--------------|
    | index  | uint32 | `0`부터 시작하는 chunk 인덱스입니다. Tendermint 는 chunk 를 순차적으로 적용합니다. | 1            |
    | chunk  | bytes  | `LoadSnapshotChunk`가 반환한 바이너리 chunk 입니다.              | 2            |
    | sender | string | 이 chunk 를 전송한 노드의 P2P ID입니다.                                 | 3            |

* **Response**:

    | Name           | Type                | Description                                                                                                                                                                                                                             | Field Number |
    |----------------|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|
    | result         | Result  (see below) | 이 chunk 를 적용한 결과입니다.                                                                                                                                                                                                      | 1            |
    | refetch_chunks | repeated uint32     | 'result' 에 관계없이 주어진 chunk 를 다시 가져와 다시 적용합니다. 나열된 chunk 만 리프레시하고 순차적으로 다시 적용합니다.                                                                                              | 2            |
    | reject_senders | repeated string     | `Result` 에 관계없이 지정된 P2P 발신자를 거부합니다. 이미 적용된 chunk 는 명시적으로 요청하지 않는 한 다시 가져오지 않지만 이러한 발신자로부터 대기 중인 chunk 는 삭제되고 새 chunk 나 다른 스냅샷은 거부됩니다. | 3            |

```proto
  enum Result {
    UNKNOWN         = 0;  // Result 를 알 수 없음, 모든 스냅샷 복원 중단
    ACCEPT          = 1;  // chunk 가 수락됨.
    ABORT           = 2;  // 스냅샷 복원을 중단하고 다른 스냅샷을 시도하지 마세요.
    RETRY           = 3;  // 이 chunk 를 다시 적용하고 `RefetchChunks` 및 `RejectSenders`와 적절히 결합합니다.
    RETRY_SNAPSHOT  = 4;  // 별도의 지시가 없는 한 chunk 를 재사용하면서 `OfferSnapshot`에서 이 스냅샷을 다시 시작합니다.
    REJECT_SNAPSHOT = 5;  // 이 스냅샷을 거부하고 다른 스냅샷을 시도하세요.
  }
```

* **Usage**:
    * 애플리케이션은 적절하게 chunk 를 다시 가져오거나 P2P 피어를 밴하도록 선택할 수 있습니다. 애플리케이션의 지시가 없는 한 Tendermint 는 이를 수행하지 않습니다.
    * 애플리케이션은 `Snapshot.Metadata` 에 chunk 해시를 첨부하거나 `AppHash` 와 비교하여 콘텐츠를 점진적으로 검증하는 등 각 chunk 를 검증할 수 있습니다.
    * 모든 chunk 가 수락되면, Tendermint 는 ABCI `Info` 호출을 통해 `LastBlockAppHash` 와 `LastBlockHeight` 가 예상 값과 일치하는지 확인하고, 노드 상태에 `AppVersion` 을 기록합니다. 그런 다음 fast sync 또는 consensus 로 전환하고 네트워크에 참여합니다.
    * 일정 시간이 지나도 적절한 피어를 찾을 수 없는 경우(e.g. 사용할 수 있는 피어가 없음) Tendermint 는 스냅샷을 거부하고 `OfferSnapshot` 을 통해 다른 스냅샷을 시도합니다. 애플리케이션은 적절하게 재설정하고 수락하거나 중단할 준비를 해야 합니다.

## Data Types

ABCI에서 사용되는 대부분의 데이터 구조는 공유되는 [common data structures](../spec/core/data_structures.md) 입니다. 경우에 따라 ABCI는 여기에 설명된 다른 데이터 구조를 사용합니다:

### Validator

* **Fields**:

    | Name    | Type  | Description                                                         | Field Number |
    |---------|-------|---------------------------------------------------------------------|--------------|
    | address | bytes | 검증자의 [주소](../core/data_structures.md#address)          | 1            |
    | power   | int64 | 검증자의 Voting power                                       | 3            |

* **Usage**:
    * 주소로 식별된 검증자입니다
    * VoteInfo 의 일부로 RequestBeginBlock 에 사용됩니다
    * ABCI를 통해 잠재적으로 대량의 퀀텀 퍼블릭키를 전송하지 않기 위해 퍼블릭키를 포함하지 않습니다.

### ValidatorUpdate

* **Fields**:

    | Name    | Type                                             | Description                   | Field Number |
    |---------|--------------------------------------------------|-------------------------------|--------------|
    | pub_key | [Public Key](../core/data_structures.md#pub_key) | 검증자의 Public key   | 1            |
    | power   | int64                                            | 검증자의 Voting power | 2            |

* **Usage**:
    * PubKey로 식별된 검증자
    * Tendermint 에게 검증자 세트를 업데이트하도록 지시하는 데 사용됩니다.

### VoteInfo

* **Fields**:

    | Name              | Type                    | Description                                                  | Field Number |
    |-------------------|-------------------------|--------------------------------------------------------------|--------------|
    | validator         | [Validator](#validator) | 검증자                                                  | 1            |
    | signed_last_block | bool                    | 검증자가 마지막 블록에 서명했는지 여부를 나타냅니다. | 2            |

* **Usage**:
    * 검증자가 마지막 블록에 서명했는지 여부를 나타내며, 검증자 가용성에 따라 보상을 받을 수 있습니다.

### Evidence

* **Fields**:

    | Name               | Type                                                                                                                                 | Description                                                                  | Field Number |
    |--------------------|--------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------|--------------|
    | type               | [EvidenceType](#evidencetype)                                                                                                        | Evidence 유형입니다. 가능한 Evidence 의 enum 입니다.                        | 1            |
    | validator          | [Validator](#validator)                                                                                                              | 문제를 일으킨 검증자                                                      | 2            |
    | height             | int64                                                                                                                                | 문제를 일으킨 높이                                             | 3            |
    | time               | [google.protobuf.Timestamp](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#google.protobuf.Timestamp) | 문제가 발생한 블록 높이가 커밋된 블록의 시간 | 4            |
    | total_voting_power | int64                                                                                                                                | 높이 `Height`로 설정된 검증자의 총 Voting Power                   | 5            |

#### EvidenceType

* **Fields**

    EvidenceType 은 나열된 필드가 있는 enum 입니다:

    | Name                | Field Number |
    |---------------------|--------------|
    | UNKNOWN             | 0            |
    | DUPLICATE_VOTE      | 1            |
    | LIGHT_CLIENT_ATTACK | 2            |

### LastCommitInfo

* **Fields**:

    | Name  | Type                           | Description                                                                                                           | Field Number |
    |-------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------|--------------|
    | round | int32                          | Commit round. 현재 블록에 대한 합의에 도달하는 데 걸린 총 라운드 수를 반영합니다.                 | 1            |
    | votes | repeated [VoteInfo](#voteinfo) | 투표권을 가진 마지막 검증자 세트의 검증자 주소와 투표에 서명했는지에 대한 여부 목록입니다. | 2            |

### ConsensusParams

* **Fields**:

    | Name      | Type                                                          | Description                                                                  | Field Number |
    |-----------|---------------------------------------------------------------|------------------------------------------------------------------------------|--------------|
    | block     | [BlockParams](../core/data_structures.md#blockparams)                                   | 블록의 크기와 연속된 블록 사이의 시간을 제한하는 파라미터입니다. | 1            |
    | evidence  | [EvidenceParams](../core/data_structures.md#evidenceparams)   | 비잔틴 행동에 대한 Evidence 의 유효성을 제한하는 파라미터입니다.         | 2            |
    | validator | [ValidatorParams](../core/data_structures.md#validatorparams) | 검증자가 사용할 수 있는 공개 키의 유형을 제한하는 파라미터입니다.             | 3            |
    | version   | [VersionsParams](../core/data_structures.md#versionparams)       | ABCI 애플리케이션 버전입니다.                                                | 4            |

### ProofOps

* **Fields**:

    | Name | Type                         | Description                                                                                                                                                                                                                  | Field Number |
    |------|------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|
    | ops  | repeated [ProofOp](#proofop) | 서로 다른 유형의 연쇄 머클 증명 목록입니다. 한 연산자의 머클 루트는 다음 연산에서 증명되는 값입니다. 최종 연산에 대한 머클 루트는 검증 중인 최종 루트 해시와 같아야 합니다. | 1            |

### ProofOp

* **Fields**:

    | Name | Type   | Description                                    | Field Number |
    |------|--------|------------------------------------------------|--------------|
    | type | string | 머클 증명의 유형 및 인코딩 방법.     | 1            |
    | key  | bytes  | 이 증명을 위한 머클 트리의 Key 를 입력합니다. | 2            |
    | data | bytes  | Key 에 대해 인코딩된 머클 증명.              | 3            |

### Snapshot

* **Fields**:

    | Name     | Type   | Description                                                                                                                                                                       | Field Number |
    |----------|--------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|
    | height   | uint64 | 스냅샷을 찍은 높이(커밋 후)입니다.                                                                                                                        | 1            |
    | format   | uint32 | 애플리케이션별 스냅샷 포맷으로, 애플리케이션이 스냅샷 데이터 포맷을 버전화하고 이전 버전과 호환되지 않는 변경을 할 수 있습니다. Tendermint 는 이를 해석하지 않습니다. | 2            |
    | chunks   | uint32 | 스냅샷의 chunk 수입니다. 반드시 1 이상이어야 합니다 (비어있는 경우에도).                                                                                                         | 3            |
    | hash     | bytes  | 임의의 스냅샷 해시입니다. 노드 간에 동일한 스냅샷에 대해서만 동일해야 합니다. Tendermint 는 해시를 해석하지 않고 비교만 합니다.                              | 3            |
    | metadata | bytes  | 임의의 애플리케이션 메타데이터(예: chunk 해시 또는 기타 인증 데이터).                                                                                              | 3            |

* **Usage**:
    * When sent across the network, a snapshot message can be at most 4 MB.
    * 상태 동기화 스냅샷에 사용되며, 자세한 내용은 [state sync section](../spec/p2p/messages/state-sync.md)을 참조하세요.
    * 스냅샷은 _모든_ 필드(`Metadata` 포함)가 동일한 경우에만 노드 간에 일치하는 것으로 간주됩니다. chunk 는 동일한 스냅샷을 가진 모든 노드에서 검색할 수 있습니다.
    * 네트워크를 통해 전송되는 스냅샷 메시지는 최대 4MB까지 가능합니다.