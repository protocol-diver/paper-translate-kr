---
title: "Facilitator"
description: "이 페이지는 x402 프로토콜에서 **facilitator**의 역할을 설명합니다."
---

facilitator 는 클라이언트(구매자)와 서버(판매자) 간의 결제 검증 및 정산 과정을 단순화하는 선택적이지만 권장되는 서비스입니다.

### Facilitator 란?

facilitator는 다음과 같은 역할을 수행하는 서비스입니다:

* 클라이언트가 제출한 결제 페이로드를 검증합니다.
* 서버를 대신하여 블록체인에 결제를 정산합니다.

facilitator 를 사용하면 서버는 블록체인과 직접 연결하거나 자체적으로 결제 검증 로직을 구현할 필요가 없습니다. 이를 통해 운영 복잡성이 줄어들고, 트랜잭션의 정확하고 실시간 검증이 보장됩니다.

### Facilitator 의 책임

* **결제 검증:** 클라이언트의 결제 페이로드가 서버의 결제 요구사항을 충족하는지 확인합니다.
* **결제 정산:** 검증된 결제를 블록체인에 제출하고 확인 상태를 모니터링합니다.
* **응답 제공:** 검증 및 정산 결과를 서버에 반환하여, 서버가 클라이언트 요청을 처리할지 결정할 수 있도록 합니다.

facilitator는 자금을 보관하거나 커스터디 역할을 하지 않습니다. 클라이언트가 제공한 서명된 페이로드를 기반으로 온체인 트랜잭션의 검증과 실행만 수행합니다.

### 왜 Facilitator 를 사용하는가?

facilitator를 사용하면 다음과 같은 이점이 있습니다:

* **운영 복잡성 감소:** 서버가 블록체인 노드와 직접 상호작용할 필요가 없습니다.
* **프로토콜 일관성:** 서비스 간 표준화된 검증 및 정산 흐름을 제공합니다.
* **빠른 통합:** 최소한의 블록체인 관련 개발로 결제 수용을 시작할 수 있습니다.

로컬에서 검증 및 정산을 직접 구현하는 것도 가능하지만, facilitator 를 사용하면 도입 속도가 빨라지고 프로토콜의 올바른 동작을 보장할 수 있습니다.

### Live Facilitators

여러 facilitator가 현재 프로덕션 환경에서 운영 중이며, Base, Solana, Polygon, Avalanche 등 다양한 네트워크를 지원합니다. 최신 목록은 [x402 Ecosystem](https://www.x402.org/ecosystem?filter=facilitators)에서 확인할 수 있습니다.

### 상호작용 흐름

1. `Client`가 `resource server`에 HTTP 요청을 보냅니다.
2. `Resource server`는 `402 Payment Required` 상태와 함께 Base64로 인코딩된 결제 요구사항이 포함된 `PAYMENT-REQUIRED` 헤더를 반환합니다.
3. `Client`는 서버 응답의 `accepts` 필드에서 하나의 `paymentDetails`를 선택하고, 해당 `scheme`에 따라 `Payment Payload`를 생성합니다.
4. `Client`는 Base64로 인코딩된 `Payment Payload`를 `PAYMENT-SIGNATURE` 헤더에 담아 다시 요청을 보냅니다.
5. `Resource server`는 로컬 검증을 수행하거나, `Payment Payload`와 `Payment Details`를 facilitator 서버의 `/verify` 엔드포인트에 POST하여 검증합니다.
6. `Facilitator server`는 `scheme`과 `networkId`를 기반으로 검증을 수행하고 `Verification Response`를 반환합니다.
7. 검증이 유효하면 서버는 요청을 처리합니다. 유효하지 않으면 `402 Payment Required`와 함께 오류를 반환합니다.
8. `Resource server`는 블록체인에 직접 정산하거나, facilitator 서버의 `/settle` 엔드포인트로 요청을 보냅니다.
9. `Facilitator server`는 `scheme`과 `networkId`에 따라 결제를 블록체인에 제출합니다.
10. `Facilitator server`는 블록체인에서 결제 확인을 기다립니다.
11. `Facilitator server`는 `Payment Execution Response`를 서버에 반환합니다.
12. `Resource server`는 `PAYMENT-RESPONSE` 헤더에 Base64로 인코딩된 `Settlement Response`를 포함하여 클라이언트에 응답합니다. 성공 시 `200 OK`, 실패 시 `402 Payment Required`를 반환합니다.

### 중복 정산 (Solana)

Solana에서는 동일한 결제 트랜잭션이 블록체인에서 확인되기 전에 `/settle` 엔드포인트로 여러 번 제출될 경우 경쟁 상태(race condition)가 발생할 수 있습니다. Solana의 RPC는 중복 제출에 대해서도 "성공"을 반환하기 때문에, facilitator는 각 요청에 대해 성공 응답을 반환할 수 있습니다. 악의적인 클라이언트는 이를 악용해 한 번의 결제로 여러 리소스에 접근할 수 있습니다.

이를 방지하기 위해 x402 SVM 메커니즘 패키지에는 `SettlementCache`라는 짧은 수명의 인메모리 캐시가 포함되어 있습니다. 이 캐시는 동일한 트랜잭션 페이로드에 대한 중복 정산 시도를 감지하고 차단합니다. 별도의 저장소가 필요 없으며, 항목은 약 120초(대략 Solana blockhash 수명의 두 배) 후 자동으로 제거됩니다.

이 보호 기능은 TypeScript 및 Python의 표준 SVM facilitator 등록 헬퍼를 사용할 경우 기본적으로 활성화됩니다. Go에서는 V1 및 V2 SVM facilitator 스킴 등록 시 공유 `SettlementCache` 인스턴스를 전달해야 합니다.

**facilitator 없이 직접 결제를 정산하는 경우, 반드시 동일한 중복 방지 로직을 구현해야 합니다.** 자세한 내용은 [Exact SVM Scheme Specification](https://github.com/x402-foundation/x402/blob/main/specs/schemes/exact/scheme_exact_svm.md)를 참고하세요.

### 요약

facilitator는 x402 프로토콜에서 독립적인 검증 및 정산 계층으로 동작합니다. 서버가 블록체인 인프라 없이도 결제를 검증하고 온체인 트랜잭션을 수행할 수 있도록 도와줍니다.

다음 문서를 참고하세요:

* [Client / Server](/docs/core-concepts/client-server.md) — 클라이언트와 서버의 역할과 책임 이해
* [HTTP 402](/docs/core-concepts/http-402.md) — 클라이언트에 결제 요구사항을 전달하는 방식 이해
