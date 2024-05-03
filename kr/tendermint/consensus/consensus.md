---
order: 1
---

### Comment
Source: https://github.com/tendermint/tendermint/blob/main/spec/consensus/consensus.md <br>
Date: 2024-05-03 (last commit hash is `eed27ad`) <br>

# Byzantine Consensus Algorithm

## 용어
- 네트워크는 선택적으로 연결된 _nodes_ 로 구성됩니다. 특정 노드에 직접 연결된 노드는 _peers_ 라고 합니다.
- 다음 블록(특정 _height_ `H`)을 결정하는 합의 프로세스는 하나 또는 여러 개의 _rounds_ 로 구성됩니다.
- `NewHeight`, `Propose`, `Prevote`, `Precommit`, 그리고 `Commit` 은 round 의 
  state machine 상태를 나타냅니다. (일명 `RoundStep` 또는 "step").
- 노드는 주어진 height, round, step, 또는 `(H,R,S)`, step 을 생략하는 경우 `(H,R)`에 있다고 합니다.
- _prevote_ 또는 _precommit_ 은 무언가에 대한 
  [prevote vote](https://godoc.org/github.com/tendermint/tendermint/types#Vote) 혹은 
  [첫 번째 precommit vote](https://godoc.org/github.com/tendermint/tendermint/types#FirstPrecommit) 를
  브로드캐스트하는 것을 의미합니다.
- `(H,R)` 에서의 vote 는 [sign-bytes](../core/data_structures.md#vote)에 `H`와 `R`의
  바이트가 포함되어 서명된 투표입니다.
- _+2/3_ 는 “2/3 이상”의 줄임말입니다.
- _1/3+_ 는 “1/3 이상”의 줄임말입니다.
- 특정 블록에 대해 +2/3의 사전투표 또는 `(H,R)` 에 `<nil>` 을 설정하는 것을 _proof-of-lock-change_ 또는
  줄여서 _PoLC_ 라고 합니다.


## State Machine 개요

블록체인의 각 높이에서 round 기반 프로토콜이 실행되어 다음 블록을 결정합니다. 각 round 는 
세 단계(`Propose`, `Prevote`, `Precommit`)와 두 가지 특수 step 인 `Commit` 및 `NewHeight`로 구성됩니다.

최적의 시나리오에서 단계의 순서는 다음과 같습니다:

```md
NewHeight -> (Propose -> Prevote -> Precommit)+ -> Commit -> NewHeight ->...
```

이 순서 `(Propose -> Prevote -> Precommit)` 를 _round_ 라고 합니다. 주어진 
높이에서 블록을 커밋하는 데 한 번 이상의 round 가 필요할 수 있습니다. round 가 더 필요한 이유에
대한 예는 다음과 같습니다:

- 지정된 proposer 가 온라인 상태가 아니었습니다.
- 지정된 proposer 가 제안한 블록이 유효하지 않습니다.
- 지정된 proposer 가 제안한 블록이 제시간에 전파되지 않았습니다.
- 제안된 블록은 유효하지만, `Precommit` 단계에 도달할 때까지 충분한 검증자 노드가 제안된
  블록에 대한 +2/3의 prevote 를 제때 받지 못했습니다. 다음 단계로 진행하려면 +2/3의 
  사전 투표가 필요하지만, 최소 한 명의 검증자가 `<nil>` 에 투표했거나 악의적으로 다른 것에 
  투표했을 수 있습니다.
- 제안된 블록은 유효하고 충분한 노드에서 +2/3의 prevote 를 받았지만, 제안된 블록에 대한 
  +2/3의 precommit 을 충분한 검증자 노드에서 받지 못했습니다.

이러한 문제 중 일부는 다음 round & proposer 로 넘어가면 해결됩니다. 다른 문제들은 각 
round 마다 특정 round timeout 매개변수를 증가시킴으로써 해결됩니다.

## State Machine 다이어그램

```md
                         +-------------------------------------+
                         v                                     |(Wait til `CommmitTime+timeoutCommit`)
                   +-----------+                         +-----+-----+
      +----------> |  Propose  +--------------+          | NewHeight |
      |            +-----------+              |          +-----------+
      |                                       |                ^
      |(Else, after timeoutPrecommit)         v                |
+-----+-----+                           +-----------+          |
| Precommit |  <------------------------+  Prevote  |          |
+-----+-----+                           +-----------+          |
      |(When +2/3 Precommits for block found)                  |
      v                                                        |
+--------------------------------------------------------------------+
|  Commit                                                            |
|                                                                    |
|  * Set CommitTime = now;                                           |
|  * Wait for block, then stage/save/commit block;                   |
+--------------------------------------------------------------------+
```

# Background Gossip

노드는 해당 validator 개인키가 없을 수 있지만, 관련 메타데이터, 제안서, 블록, 투표를 peer 에게 
전달함으로써 합의 과정에서 적극적인 역할을 수행합니다. active validator 의 개인 키를 가지고 
있으며 투표 서명에 참여하는 노드를 _validator-node_ 라고 합니다. validator 노드뿐만 아니라 
모든 노드는 관련 상태(현재 높이, round, step)를 가지며 진행을 위해 노력합니다.

두 노드 사이에는 `Connection`이 존재하며, 이 연결 위에 상당히 스로틀링된 정보의 `Channel`이 
멀티플렉싱되어 있습니다. 이러한 채널 중 일부에 유행성 가십 프로토콜이 구현되어 peer 들이 가장 최근의 
합의 상태를 알 수 있습니다. 예를 들어,

- 노드는 현재 round의 proposer가 제안한 블록의 `PartSet` 부분을 가십합니다. 
  가십 네트워크에 블록을 빠르게 전파하기 위해 LibSwift에서 영감을 얻은 알고리즘이 사용됩니다.
- 노드는 prevote/precommit 투표를 가십합니다. `NODE_B`보다 앞서 있는 노드 `NODE_A`는 
  현재(또는 미래) round 가 진행되도록 `NODE_B`에 prevote 또는 precommit 을 전송할 수 
  있습니다.
- 노드 가십은 PoLC(proof-of-lock-change) round 가 제안되면 제안된 round 에 대한 
  투표를 진행합니다.
- 노드는 이전 블록에 대한 블록 
  [commits](https://godoc.org/github.com/tendermint/tendermint/types#Commit)을 
  통해 블록체인 높이가 뒤처진 노드에게 가십을 보냅니다.
- 노드는 기회에 따라 `ReceivedVote` 메시지를 가십으로 전달하여 이미 보유하고 있는 투표수를 
  peer 에게 암시합니다.
- 노드는 현재 상태를 인접한 모든 peer에게 브로드캐스트합니다. (하지만 더 이상 가십으로 전달되지 
  않습니다.)

이 외에도 다양한 기능이 있지만 여기서 너무 앞서 나가지 않도록 하겠습니다.

## Proposals

A proposal is signed and published by the designated proposer at each
round. The proposer is chosen by a deterministic and non-choking round
robin selection algorithm that selects proposers in proportion to their
voting power (see
[implementation](https://github.com/tendermint/tendermint/blob/v0.34.x/types/validator_set.go)).

A proposal at `(H,R)` is composed of a block and an optional latest
`PoLC-Round < R` which is included iff the proposer knows of one. This
hints the network to allow nodes to unlock (when safe) to ensure the
liveness property.

## State Machine Spec

### Propose Step (height:H,round:R)

Upon entering `Propose`:

- The designated proposer proposes a block at `(H,R)`.

The `Propose` step ends:

- After `timeoutProposeR` after entering `Propose`. --> goto
  `Prevote(H,R)`
- After receiving proposal block and all prevotes at `PoLC-Round`. -->
  goto `Prevote(H,R)`
- After [common exit conditions](#common-exit-conditions)

### Prevote Step (height:H,round:R)

Upon entering `Prevote`, each validator broadcasts its prevote vote.

- First, if the validator is locked on a block since `LastLockRound`
  but now has a PoLC for something else at round `PoLC-Round` where
  `LastLockRound < PoLC-Round < R`, then it unlocks.
- If the validator is still locked on a block, it prevotes that.
- Else, if the proposed block from `Propose(H,R)` is good, it
  prevotes that.
- Else, if the proposal is invalid or wasn't received on time, it
  prevotes `<nil>`.

The `Prevote` step ends:

- After +2/3 prevotes for a particular block or `<nil>`. -->; goto
  `Precommit(H,R)`
- After `timeoutPrevote` after receiving any +2/3 prevotes. --> goto
  `Precommit(H,R)`
- After [common exit conditions](#common-exit-conditions)

### Precommit Step (height:H,round:R)

Upon entering `Precommit`, each validator broadcasts its precommit vote.

- If the validator has a PoLC at `(H,R)` for a particular block `B`, it
  (re)locks (or changes lock to) and precommits `B` and sets
  `LastLockRound = R`.
- Else, if the validator has a PoLC at `(H,R)` for `<nil>`, it unlocks
  and precommits `<nil>`.
- Else, it keeps the lock unchanged and precommits `<nil>`.

A precommit for `<nil>` means "I didn’t see a PoLC for this round, but I
did get +2/3 prevotes and waited a bit".

The Precommit step ends:

- After +2/3 precommits for `<nil>`. --> goto `Propose(H,R+1)`
- After `timeoutPrecommit` after receiving any +2/3 precommits. --> goto
  `Propose(H,R+1)`
- After [common exit conditions](#common-exit-conditions)

### Common exit conditions

- After +2/3 precommits for a particular block. --> goto
  `Commit(H)`
- After any +2/3 prevotes received at `(H,R+x)`. --> goto
  `Prevote(H,R+x)`
- After any +2/3 precommits received at `(H,R+x)`. --> goto
  `Precommit(H,R+x)`

### Commit Step (height:H)

- Set `CommitTime = now()`
- Wait until block is received. --> goto `NewHeight(H+1)`

### NewHeight Step (height:H)

- Move `Precommits` to `LastCommit` and increment height.
- Set `StartTime = CommitTime+timeoutCommit`
- Wait until `StartTime` to receive straggler commits. --> goto
  `Propose(H,0)`

## Proofs

### Proof of Safety

Assume that at most -1/3 of the voting power of validators is byzantine.
If a validator commits block `B` at round `R`, it's because it saw +2/3
of precommits at round `R`. This implies that 1/3+ of honest nodes are
still locked at round `R' > R`. These locked validators will remain
locked until they see a PoLC at `R' > R`, but this won't happen because
1/3+ are locked and honest, so at most -2/3 are available to vote for
anything other than `B`.

### Proof of Liveness

If 1/3+ honest validators are locked on two different blocks from
different rounds, a proposers' `PoLC-Round` will eventually cause nodes
locked from the earlier round to unlock. Eventually, the designated
proposer will be one that is aware of a PoLC at the later round. Also,
`timeoutProposalR` increments with round `R`, while the size of a
proposal are capped, so eventually the network is able to "fully gossip"
the whole proposal (e.g. the block & PoLC).

### Proof of Fork Accountability

Define the JSet (justification-vote-set) at height `H` of a validator
`V1` to be all the votes signed by the validator at `H` along with
justification PoLC prevotes for each lock change. For example, if `V1`
signed the following precommits: `Precommit(B1 @ round 0)`,
`Precommit(<nil> @ round 1)`, `Precommit(B2 @ round 4)` (note that no
precommits were signed for rounds 2 and 3, and that's ok),
`Precommit(B1 @ round 0)` must be justified by a PoLC at round 0, and
`Precommit(B2 @ round 4)` must be justified by a PoLC at round 4; but
the precommit for `<nil>` at round 1 is not a lock-change by definition
so the JSet for `V1` need not include any prevotes at round 1, 2, or 3
(unless `V1` happened to have prevoted for those rounds).

Further, define the JSet at height `H` of a set of validators `VSet` to
be the union of the JSets for each validator in `VSet`. For a given
commit by honest validators at round `R` for block `B` we can construct
a JSet to justify the commit for `B` at `R`. We say that a JSet
_justifies_ a commit at `(H,R)` if all the committers (validators in the
commit-set) are each justified in the JSet with no duplicitous vote
signatures (by the committers).

- **Lemma**: When a fork is detected by the existence of two
  conflicting [commits](../core/data_structures.md#commit), the
  union of the JSets for both commits (if they can be compiled) must
  include double-signing by at least 1/3+ of the validator set.
  **Proof**: The commit cannot be at the same round, because that
  would immediately imply double-signing by 1/3+. Take the union of
  the JSets of both commits. If there is no double-signing by at least
  1/3+ of the validator set in the union, then no honest validator
  could have precommitted any different block after the first commit.
  Yet, +2/3 did. Reductio ad absurdum.

As a corollary, when there is a fork, an external process can determine
the blame by requiring each validator to justify all of its round votes.
Either we will find 1/3+ who cannot justify at least one of their votes,
and/or, we will find 1/3+ who had double-signed.

### Alternative algorithm

Alternatively, we can take the JSet of a commit to be the "full commit".
That is, if light clients and validators do not consider a block to be
committed unless the JSet of the commit is also known, then we get the
desirable property that if there ever is a fork (e.g. there are two
conflicting "full commits"), then 1/3+ of the validators are immediately
punishable for double-signing.

There are many ways to ensure that the gossip network efficiently share
the JSet of a commit. One solution is to add a new message type that
tells peers that this node has (or does not have) a +2/3 majority for B
(or) at (H,R), and a bitarray of which votes contributed towards that
majority. Peers can react by responding with appropriate votes.

We will implement such an algorithm for the next iteration of the
Tendermint consensus protocol.

Other potential improvements include adding more data in votes such as
the last known PoLC round that caused a lock change, and the last voted
round/step (or, we may require that validators not skip any votes). This
may make JSet verification/gossip logic easier to implement.

### Censorship Attacks

Due to the definition of a block
[commit](https://github.com/tendermint/tendermint/blob/v0.34.x/docs/tendermint-core/validators.md), any 1/3+ coalition of
validators can halt the blockchain by not broadcasting their votes. Such
a coalition can also censor particular transactions by rejecting blocks
that include these transactions, though this would result in a
significant proportion of block proposals to be rejected, which would
slow down the rate of block commits of the blockchain, reducing its
utility and value. The malicious coalition might also broadcast votes in
a trickle so as to grind blockchain block commits to a near halt, or
engage in any combination of these attacks.

If a global active adversary were also involved, it can partition the
network in such a way that it may appear that the wrong subset of
validators were responsible for the slowdown. This is not just a
limitation of Tendermint, but rather a limitation of all consensus
protocols whose network is potentially controlled by an active
adversary.

### Overcoming Forks and Censorship Attacks

For these types of attacks, a subset of the validators through external
means should coordinate to sign a reorg-proposal that chooses a fork
(and any evidence thereof) and the initial subset of validators with
their signatures. Validators who sign such a reorg-proposal forego its
collateral on all other forks. Clients should verify the signatures on
the reorg-proposal, verify any evidence, and make a judgement or prompt
the end-user for a decision. For example, a phone wallet app may prompt
the user with a security warning, while a refrigerator may accept any
reorg-proposal signed by +1/2 of the original validators.

No non-synchronous Byzantine fault-tolerant algorithm can come to
consensus when 1/3+ of validators are dishonest, yet a fork assumes that
1/3+ of validators have already been dishonest by double-signing or
lock-changing without justification. So, signing the reorg-proposal is a
coordination problem that cannot be solved by any non-synchronous
protocol (i.e. automatically, and without making assumptions about the
reliability of the underlying network). It must be provided by means
external to the weakly-synchronous Tendermint consensus algorithm. For
now, we leave the problem of reorg-proposal coordination to human
coordination via internet media. Validators must take care to ensure
that there are no significant network partitions, to avoid situations
where two conflicting reorg-proposals are signed.

Assuming that the external coordination medium and protocol is robust,
it follows that forks are less of a concern than [censorship
attacks](#censorship-attacks).

### Canonical vs subjective commit

We distinguish between "canonical" and "subjective" commits. A subjective commit is what
each validator sees locally when they decide to commit a block. The canonical commit is
what is included by the proposer of the next block in the `LastCommit` field of
the block. This is what makes it canonical and ensures every validator agrees on the canonical commit,
even if it is different from the +2/3 votes a validator has seen, which caused the validator to
commit the respective block. Each block contains a canonical +2/3 commit for the previous
block.