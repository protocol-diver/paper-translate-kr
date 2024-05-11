---
order: 1
parent:
  title: ABCI
  order: 2
---

### Comment
Source: https://github.com/tendermint/tendermint/blob/main/spec/abci/README.md <br>
Date: 2024-05-11 (last commit hash is `eed27ad`) <br>

# ABCI

ABCI는 "**A**pplication **B**lock**c**hain **I**nterface"의 약자입니다. ABCI는 Tendermint (스테이트 머신 복제 엔진)와 애플리케이션 (실제 스테이트 머신) 사이의 인터페이스입니다. 이는 각각 해당 `Request` 및 `Response` 메시지 유형이 있는 _method_ 집합으로 구성됩니다. 상태 머신 복제를 수행하기 위해 Tendermint는 `Request*` 메시지를 보내고 그 대가로 `Response*` 메시지를 수신하여 ABCI 애플리케이션에서 ABCI 메서드를 호출합니다.

모든 ABCI 메시지와 메서드는 [protocol buffers](https://github.com/tendermint/tendermint/blob/v0.34.x/proto/abci/types.proto) 에 정의되어 있습니다.
이를 통해 Tendermint 는 다양한 프로그래밍 언어로 작성된 애플리케이션과 함께 실행할 수 있습니다.

사양은 다음과 같이 나뉩니다:

- [Methods and Types](./abci.md) - 모든 ABCI 메소드 및 메시지 유형에 대한 전체 세부 정보
- [Applications](./apps.md) - ABCI 애플리케이션 상태를 관리하는 방법 및 ABCI 애플리케이션 빌드에 대한 기타 세부 사항
- [Client and Server](./client-server.md) - 자체 ABCI 애플리케이션 서버를 구현하고자 하는 사용자를 위한 정보입니다.