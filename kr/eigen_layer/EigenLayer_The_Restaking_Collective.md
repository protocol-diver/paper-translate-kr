# EigenLayer: The Restaking Collective
EigenLayer Team

### Abstract
우리는 이더리움의 restaking 집합체인 EigenLayer 를 제안합니다. EigenLayer 는 이더리움 상의 스마트 컨트랙트 세트로, 합의 계층에서 이더(ETH)를 staking 한 사용자들이 이더리움 생태계를 기반으로 새롭게 구축된 소프트웨어 모듈을 검증하는 데 참여할 수 있도록 선택권을 제공합니다. Staker 들은 EigenLayer 스마트 컨트랙트가 자신의 staking 된 ETH에 대해 추가적인 슬래싱 조건을 부과할 수 있도록 허용함으로써 참여할 수 있으며, 이를 통해 암호경제적 보안을 확장할 수 있습니다. EigenLayer 에 참여함으로써, staker 들은 합의 프로토콜, 데이터 가용성 계층, 가상 머신, 키퍼 네트워크, 오라클 네트워크, 브릿지, 임계값 암호화 스킴, 신뢰 실행 환경(TEE) 등 다양한 유형의 모듈을 검증할 수 있습니다.
이러한 방식은 모듈 간 보안을 분산시키는 대신, 모든 모듈에 걸쳐 ETH 보안을 집약합니다. 이로 인해 모듈에 의존하는 분산 애플리케이션(DApp)의 보안이 강화됩니다. 또한, 이더리움 보유자들은 이러한 다양한 모듈에서 생성되는 새로운 수수료 기반 기회를 통해 추가적인 가치를 추구할 수 있습니다.
EigenLayer 는 이더리움의 실험적 테스트 환경으로도 작동합니다. 이를 통해 Danksharding 및 Proposer/Builder separation 와 같은 혁신들이 다양한 변형 상태에서 실험되고, 최적의 아이디어들이 이더리움에 다시 통합될 수 있습니다.
마지막으로, EigenLayer 는 새로운 비허가형 혁신(permissionless innovation) 시대를 열어줍니다. 이를 통해 혁신가들은 새로운 분산 검증 모듈을 구현하기 위해 자신만의 신뢰 네트워크를 구축할 필요 없이, EigenLayer 를 통해 ETH restaker 들이 제공하는 보안성과 탈중앙화를 활용할 수 있습니다.

## 1. The Problem: Fractured Trust Networks
<b>Bitcoin</b>. 비트코인은 블록체인의 개념을 선구적으로 제시하며 탈중앙화 신뢰의 시대를 열었습니다 [1]. 그러나 비트코인은 애플리케이션 특화 방식으로 설계되어, 피어 투 피어(peer-to-peer) 결제에만 초점을 맞췄습니다. 따라서 새로운 탈중앙화 애플리케이션을 개발하려면 자체 신뢰 네트워크를 갖춘 새로운 블록체인이 필요했습니다. 하지만 탈중앙화 신뢰를 생성하고 유지하는 것은 매우 어려운 일이며, 이로 인해 이러한 모델에서는 새로운 탈중앙화 애플리케이션을 구축하는 데 많은 어려움이 따랐습니다.

<b>Ethereum</b>. 이더리움은 2013년에 구상되어 2015년에 출시되었으며, 그 후 상당한 변화를 가져왔습니다 [2]. 이더리움은 이더리움 가상 머신(EVM) [3]에서 완전하게 프로그래밍 가능함으로써, 분산 애플리케이션(DApp)이 이더리움 신뢰 네트워크 위에 허가 없이 구축될 수 있는 모듈형 블록체인 개념을 선도했습니다. 이더리움의 신뢰 네트워크는 위에 구축된 모든 DApp에 대해 집합적인 보안을 제공합니다. 이로 인해 혁신과 신뢰가 분리되었습니다: 혁신은 어떤 DApp 개발자에게서나 나올 수 있지만, 개발자는 DApp 의 안전성과 실행 지속성에 대해 신뢰를 받을 필요가 없습니다. 왜냐하면 신뢰는 블록체인에 의해 보증되기 때문입니다. 가치 흐름은 블록체인이 DApp 에 신뢰를 공급하고, 그 대가로 수수료를 받는 구조입니다. 우리는 이 혁신과 신뢰의 분리가 가명 경제(pseudonymous economy) 의 핵심 동력임을 주목할 필요가 있습니다. 혁신가는 명성이나 신뢰를 필요로 하지 않으며, DApp 은 신뢰할 수 있는 블록체인을 믿고 DApp 코드를 검증할 수 있는 누구나 사용할 수 있습니다.

<b>Ethereum Layer-2 Era</b>. 이더리움이 롤업 중심 로드맵 [4]으로 전환하면서, 이더리움 위에 허가 없이 구축할 수 있는 애플리케이션의 범위는 크게 확장되었습니다. 롤업은 실행을 단일 노드 또는 소수의 노드에 아웃소싱하지만, EVM 컨트랙트를 통해 계산 결과를 이더리움에 증명함으로써 이더리움의 신뢰를 흡수할 수 있습니다. 이때 계산 증명은 암호경제적 보장(예: 사기 증명, 이 경우 롤업은 "옵티미스틱 롤업"이라고 불림) 또는 암호학적 보장(예: 간결한 유효성 증명, 이 경우 롤업은 종종 "ZK 롤업"이라고 불림)을 사용할 수 있습니다 [5]. 이러한 변화는 롤업 기술에서의 허가 없는 혁신 속도를 대폭 증가시켰으며, 그로 인해 다양한 증명 기술들이 급격히 확산되었습니다.

![Figure1](https://github.com/protocol-diver/paper-translate-kr/blob/main/images/eigen_layer/whitepaper/page_2_1.png)

Figure1: 오늘날 활성화된 서비스 생태계와 EigenLayer 생태계 비교

<b>제한 사항.</b> 그러나 EVM 위에 배포되거나 증명할 수 없는 모듈은 이더리움의 풀링된 신뢰를 흡수할 수 없습니다. 이러한 모듈은 이더리움 외부에서 유래된 입력을 처리해야 하며, 따라서 그 처리 과정은 이더리움 내에서 프로토콜로 검증될 수 없습니다. 이러한 모듈의 예로는 새로운 합의 프로토콜을 기반으로 한 사이드체인, 데이터 가용성 계층, 새로운 가상 머신, 키퍼 네트워크, 오라클 네트워크, 브리지, 임계값 암호화 체계, 신뢰할 수 있는 실행 환경 등이 있습니다. 일반적으로 이러한 모듈은 검증을 수행하기 위해 자체 분산 검증 의미론을 가진 활성 검증 서비스(AVS)가 필요합니다. 보통 이러한 actively validated serviced (“AVS”)는 자체 네이티브 토큰으로 보안이 유지되거나 허가된 방식으로 운영됩니다. 현재 AVS 생태계의 조직 방식에는 네 가지 기본적인 단점이 있습니다.

1. <b>새로운 AVS 의 부트스트래핑 문제.</b> 새로운 AVS 를 개발하려는 혁신자들은 보안을 확보하기 위해 새로운 신뢰 네트워크를 부트스트랩해야 합니다.
2. <b>가치 유출.</b> 각 AVS 가 자체 신뢰 풀을 개발함에 따라, 사용자는 이더리움의 거래 수수료 외에도 이러한 풀에 수수료를 지불해야 합니다. 이 수수료 흐름의 분산은 이더리움에서 가치 유출을 초래합니다.
3. <b>자본 비용 부담.</b> 새로운 AVS 를 보호하기 위해 staking 하는 검증자는 자본 비용을 부담해야 하며, 이는 새로운 시스템에서 staking 과 관련된 기회 비용 및 가격 위험에 해당합니다. 따라서 AVS 는 이 비용을 충당할 수 있도록 충분히 높은 staking 수익을 제공해야 합니다. 현재 운영 중인 대부분의 AVS 에서 staking 의 자본 비용은 운영 비용을 훨씬 초과합니다. 예를 들어, $10B 달러의 stake 가 이를 보호하는 데이터 가용성 계층을 고려하고, staker 가 기대하는 연간 백분율 수익률(APR)이 5%라고 가정해 봅시다. 이 AVS는 자본 비용을 보상하기 위해 staker 에게 최소한 연간 $0.5B 달러를 지급해야 합니다. 이는 데이터 저장 또는 네트워크 비용과 관련된 운영 비용보다 훨씬 큰 금액입니다.
4. <b>DApp 에 대한 낮은 신뢰 모델.</b> 현재 AVS 생태계는 매우 바람직하지 않은 보안 동태를 초래합니다. 일반적으로, DApp 의 미들웨어 의존성 중 하나가 공격의 대상이 될 수 있습니다. 따라서 DApp 의 손상(섹션 3.1에서 설명 참조) 비용은 일반적으로 그 의존성 중 적어도 하나를 손상시키는 최소 비용 이상으로 간주해야 합니다. 그림 3a 에서 예시를 참조하십시오. 애플리케이션이 오라클과 같은 중요한 모듈에 의존하고 이 모듈이 작은 stake 로 보호받는 세상에서는, 이더리움이 제공하는 강력한 경제적 보안 보장이 큰 의미가 없을 수 있습니다. 왜냐하면 오라클을 공격하는 비용이 이더리움을 공격하는 비용보다 훨씬 낮기 때문입니다.

## 2. EigenLayer: The Restaking Collective
EigenLayer 는 두 가지 혁신적인 아이디어인 <b>restaking 을 통한 pooled security</b> 와 <b>자유 시장 거버넌스</b>를 도입하여 이더리움의 보안을 모든 시스템으로 확장하고 기존의 경직된 거버넌스 구조의 비효율성을 제거합니다:

![Figure2](https://github.com/protocol-diver/paper-translate-kr/blob/main/images/eigen_layer/whitepaper/page_3_1.png)

Figure 2: EigenLayer 위에 AVS 를 출시하려면 운영자가 다운로드해야 하는 오프체인 컨테이너와 슬래싱 및 지불 조건을 명시하는 온체인 컨트랙트를 배포해야 합니다.

1. <b>Restaking 을 통한 pooled security.</b> EigenLayer 는 모듈이 자체 토큰이 아닌 restaking 된 ETH로 보호되도록 함으로써 새로운 pooled security 메커니즘을 제공합니다. 특히, 이더리움 검증자는 비콘 체인 인출 자격 증명을 EigenLayer 스마트 컨트랙트로 설정하고, EigenLayer 위에 구축된 새로운 모듈에 참여할 수 있습니다. 검증자는 이러한 모듈에 필요한 추가 노드 소프트웨어를 다운로드하여 실행합니다. 모듈은 해당 모듈에 참여한 검증자의 staking 된 ETH에 추가 슬래싱 조건을 부과할 수 있습니다. 우리는 이 메커니즘을 restaking 이라고 부릅니다. 그 대가로 검증자는 선택한 모듈에 보안 및 검증 서비스를 제공함으로써 추가 수익을 얻습니다. 온체인 검증 가능한 슬래싱 메커니즘과 결합하면, restaking 메커니즘은 깊은 암호경제적 보안 전이를 가능하게 하며, 이에 대해 자세한 내용은 섹션 A에서 다룹니다. 그림 2 를 참고하세요. 예를 들어, 모듈이 데이터 가용성 계층이라면, EigenLayer 를 통한 restaker 는 데이터를 모듈을 통해 저장할 때마다 지불을 받습니다. 그 대가로, restaker 는 proof-of-custody 를 통해 실행되는 슬래싱 조건에 따릅니다. 그림 4 에서 설명하듯이, restaking 은 보안을 풀링할 수 있는 블록체인 애플리케이션의 범위를 크게 확장합니다. 따라서 EigenLayer 는 이더리움이 가능하게 한 스마트 컨트랙트 기반 DApp을 넘어 가상 머신, 합의 프로토콜, 미들웨어로까지 열린 혁신을 확장합니다. 온체인 슬래싱 컨트랙트를 가진 모든 AVS 는 EigenLayer 를 통해 보호될 수 있습니다.
2. <b>오픈 마켓플레이스.</b> EigenLayer 는 검증자가 제공하는 pooled security 가 AVS 에 의해 소비되는 방식을 관리하는 오픈 마켓 메커니즘을 제공합니다. EigenLayer 는 검증자가 EigenLayer 위에 구축된 각 모듈에 참여 여부를 선택할 수 있는 마켓플레이스를 만듭니다(그림 4 참조). 다양한 모듈은 검증자가 restaking 된 ETH 를 자신들의 모듈에 할당하도록 충분히 인센티브를 제공해야 하며, 검증자는 추가 슬래싱 가능성을 고려하여 어떤 모듈에 추가 pooled security 를 할당할 가치가 있는지 결정하는 데 도움을 줍니다. EigenLayer 의 옵트인(dynamic opt-in) 동작은 두 가지 중요한 이점을 제공합니다: (1) 핵심 블록체인의 안정적이고 보수적인 거버넌스를 보완하여 새로운 보조 기능을 신속하고 효율적으로 출시할 수 있는 자유 시장 거버넌스 구조를 제공합니다. (2) 옵트인 검증은 새로운 블록체인 모듈이 검증자 간의 이질적인 리소스를 활용할 수 있도록 하여 보안과 성능 간의 더 잘 조율된 절충안을 가능하게 만듭니다. 이러한 이점에 대해서는 섹션 4에서 더 자세히 탐구합니다.

이러한 아이디어를 결합함으로써, EigenLayer 는 <b>AVS 가 이더리움 검증자가 제공하는 pooled security 를 대여할 수 있는 오픈 마켓플레이스</b> 역할을 합니다.

![Figure3](https://github.com/protocol-diver/paper-translate-kr/blob/main/images/eigen_layer/whitepaper/page_4_1_a.png)

(a) 현재 AVS 경제학. AVS 를 손상시키기 위해 공격자는 $10B 의 stake 로 L1을 보호하는 것보다 훨씬 적은 $1B 의 stake 로 보호되는 모듈 중 하나만 공격하면 됩니다.

![Figure3](https://github.com/protocol-diver/paper-translate-kr/blob/main/images/eigen_layer/whitepaper/page_4_1_b.png)

(b) Pooled security 의 AVS 경제학. 여기에서 DApp 을 손상시키려면 공격자가 $13B 의 pooled stake 를 공격해야 합니다.

Figure 3: EigenLayer 의 pooled security.

EigenLayer 는 위에서 언급된 AVS 생태계의 다양한 문제를 해결합니다:

1. <b>새로운 AVS 의 부트스트래핑 문제:</b> 새로운 AVS 는 이더리움의 대규모 검증자 집합에서 보안을 부트스트랩할 수 있습니다.
2. <b>자본 비용:</b> ETH staker 는 여러 서비스에서 자본을 재사용하므로 자본 비용이 분산됩니다. 특히 EigenLayer 에 참여하는 네이티브 ETH staker 의 한계 자본 비용은 최소화됩니다 (정확히 말하면, 정직한 노드가 슬래시될 위험이 없다면 이론적으로 제로).
3. <b>신뢰 집합:</b> 더 큰 자본 pool 을 restaking 하기 때문에 신뢰 모델이 크게 개선됩니다. 예를 들어, EigenLayer 세계에서 그림 3b 의 시나리오를 고려해 봅시다. 모든 L1 stake 가 세 개의 AVS 모듈에 restaking 된다고 가정하면, DApp 을 손상시키는 비용은 L1 자체에 staking 된 총 금액입니다. 세 개의 AVS 에서 발생하는 <b>추가적인 수익 기회</b> 덕분에 EigenLayer 가 존재하는 L1 에 staking 된 총 금액은 EigenLayer 가 없는 세계에서 L1 과 각 AVS 에 각각 staking 된 금액의 합과 같습니다. 따라서 위의 예에서 EigenLayer 가 존재하는 L1 에 staking 된 총 금액은 $13B 입니다. 이렇게 EigenLayer 는 손상 비용을 최소 stake 에서 stake 의 합으로 대폭 증가시킵니다.
4. <b>가치 축적:</b> EigenLayer 는 ETH staker 에게 참여할 수 있는 여러 추가 수익원을 제공하며, 매우 안전한 AVS 생태계의 존재로 인해 생태계의 네트워크 효과를 더욱 강화합니다.

### 2.1. EigenLayer Enables Multiple Staking Modalities
우리는 여기서 liquid staking, superfluid staking, restaking 과 같은 다양한 staking 관련 패러다임을 비교합니다. 먼저, 그림 5a에 나타난 기존의 체계부터 시작합니다.

![Figure4](https://github.com/protocol-diver/paper-translate-kr/blob/main/images/eigen_layer/whitepaper/page_5_1.png)

Figure 4: EigenLayer 를 통해 permissionless 혁신이 블록체인 스택 깊숙이 침투합니다.

- <b>Liquid Staking.</b> Lido [6]와 Rocket Pool [7]과 같은 liquid staking 서비스는 사용자가 자신의 ETH 를 staking pool 에 예치하고, 자신의 ETH 와 staking 수익에 대한 청구권을 나타내는 Liquid Staking Token (LST)을 받도록 합니다. Staking pool 내에서 ETH 는 합의 프로토콜에 참여하는 여러 검증자 중 한 명에게 위임됩니다. LST 는 Shapella 업그레이드 이후 underlying ETH 가치에 대해 교환할 수 있지만, 교환은 ETH staking 인출 기간과 동일한 대기 기간을 거쳐야 합니다. LST 는 또한 Uniswap 과 Curve 와 같은 거래소를 통해 DeFi 생태계에서 거래될 수 있습니다.

- <b>Superfluid Staking.</b> Superfluid staking 은 liquid staking 의 순서를 뒤집어, 핵심 합의 프로토콜을 수정하여 유동성 제공(LP) 토큰 [8]의 stking 을 허용합니다. LP 토큰은 Uniswap 이나 Curve 와 같은 DeFi 거래소에 포함된 총 유동성의 일부를 나타냅니다.

그림 5b 에 나타난 것처럼, EigenLayer 는 staker 가 새로운 AVS 를 보호함으로써 추가 수익을 얻을 수 있는 여러 경로를 제공합니다. 일반적으로 우리는 블록체인을 세 가지 주요 계층으로 나눌 수 있습니다: 핵심 프로토콜, AVS, 그리고 DeFi. Liquid staking 은 먼저 핵심 프로토콜을 거친 후 DeFi 계층으로 가는 방식으로 수익을 쌓는 것이라고 볼 수 있습니다. Superfluid staking 은 먼저 DeFi 계층을 거친 후 핵심 프로토콜 계층으로 가는 방식이라고 볼 수 있습니다. EigenLayer 에서는 여러 가지 restaking 모달리티가 있을 수 있습니다:

1. <b>Native restaking.</b> 검증자는 자신의 출금 자격 증명을 EigenLayer 컨트랙트로 지정하여 native 방식으로 staking 된 ETH 를 restaking 할 수 있습니다. 이는 L1 → EigenLayer 수익 쌓기와 동일합니다.
2. <b>LST restaking.</b> 검증자는 Lido 와 Rocket Pool 과 같은 프로토콜을 통해 이미 restaking 된 ETH 인 LST 를 staking 하여, 자신의 LSD 를 EigenLayer 스마트 컨트랙트로 전송함으로써 restaking 할 수 있습니다. 이는 DeFi → EigenLayer 수익 쌓기와 동일합니다.
3. <b>ETH LP restaking.</b> 검증자는 ETH 를 포함하는 쌍의 LP 토큰을 staking 합니다. 이는 DeFi → EL 수익 쌓기와 동일합니다.
4. <b>LST LP restaking.</b> 검증자는 Curve 의 stETH-ETH LP 토큰과 같은 liquid staking ETH 토큰을 포함하는 쌍의 LP 토큰을 staking 하여 L1 → DeFi → EL 수익 쌓기 경로를 따릅니다.

이 각 경로는 다양한 종류의 리스크를 동반합니다. 옵트인 거버넌스 원칙에 따라, EigenLayer 는 이러한 리스크의 관리를 모듈 개발자에게 위임합니다. 개발자는 자신이 선택한 AVS 에 대해 어떤 토큰을 stake 로 받을지 결정합니다. 또한, 분배하는 보상에 대해 다른 유형의 staking 된 토큰에 우선 순위를 두는지 여부도 선택할 수 있습니다. 예를 들어, 분산화를 주로 추구하는 모듈은 native 로 restaking 된 ETH 로만 restaking 을 받을 수 있습니다.

![Figure5](https://github.com/protocol-diver/paper-translate-kr/blob/main/images/eigen_layer/whitepaper/page_6_1_a.png)

(a) Liquid staking = L1 → DeFi. Superfluid staking = DeFi → L1

![Figure5](https://github.com/protocol-diver/paper-translate-kr/blob/main/images/eigen_layer/whitepaper/page_6_1_b.png)

(b) (1)Native restaking or LST token 의 staking: L1 → EL. (2) ETH 와 함께 LP 토큰을 staking: DeFi → EL. (3) LST와 함께 LP 토큰을 staking: L1 → DeFi → EL.

Figure 5: Staking patterns.

### 2.2. Delegation in EigenLayer
EigenLayer 의 ETH 또는 LST 를 보유한 많은 restaker 들은 EigenLayer 에 참여하고 싶지만, EigenLayer 운영자가 되기를 원하지 않을 수 있습니다. EigenLayer 는 이러한 restaker 들이 자신들의 ETH 또는 LST 를 다른 객체에 위임하여 EigenLayer 운영 노드를 실행하는데 도움을 줄 수 있는 경로를 제공합니다. 자신에게 stake 가 위임된 EigenLayer 운영자는 그 위임된 stake 를 예치하여 새로운 이더리움 검증자 노드를 생성하고, 해당 위임된 stake 를 운영자가 참여하는 모듈로부터 처벌받을 수 있습니다. 이러한 운영자는 이더리움 비콘 체인과 EigenLayer 를 통해 참여하는 모듈에서 수수료를 받습니다. 그들은 이 수수료 중 일부를 보유하고 나머지를 위임자에게 전달합니다.

Solo staking: 이 모델에서 native 로 restaking 된 solo staker 는 EigenLayer 에 참여하기 위한 두 가지 선택지를 가집니다: (1) Solo staker 는 EigenLayer 에서 검증 서비스를 직접 제공할 수 있는 AVS 에 옵트인할 수 있습니다. 또는 (2) Solo staker 는 EigenLayer 운영을 다른 객체에 위임하면서 이더리움 검증을 계속할 수 있습니다. 후자의 옵션은 업그레이드 불가능하고 경량화된 설정을 가진 home staker 들이 이더리움의 분산화 및 검열 저항을 계속 기여하면서 (그리고 코어 이더리움 수익을 운영자와 공유하지 않으면서) EigenLayer 를 통해 다른 운영자에게 위임하여 추가 보상을 받을 수 있게 해줍니다. 우리는 EigenLayer 위에 구축될 많은 서비스들이 경량화되어, 자신의 stake 를 다른 운영자에게 위임하고 싶지 않은 home staker 들에게 적합할 것이라고 상상합니다. 예를 들어, 계산적으로 실행이 쉬운 분산형 가격 오라클이 있지만 높은 신뢰를 요구하는 경우를 고려할 수 있습니다.

위임 모델: EigenLayer 의 위임 모델에서는 staker 가 자신의 stake 를 운영자에게 위임해야 합니다. 만약 운영자가 참여하는 EigenLayer 모듈에서 의무를 이행하지 않으면, 위임된 stake 는 처벌을 받을 수 있습니다. 이 운영자에게 stake 를 위임한 restaker 들도 처벌을 받게 됩니다. 따라서, EigenLayer restaker 는 성공적으로 의무를 이행한 실적이 있는 신뢰할 수 있는 운영자에게만 stake 를 위임해야 합니다. EigenLayer 자체에는 위임에 대한 인센티브가 내장되어 있지 않지만, 다른 사람들이 EigenLayer 위에 혁신적인 위임 프레임워크를 구축하는 것은 가능합니다.

<b>수수료 모델:</b> 또한, EigenLayer restaker 는 운영자가 위임자에게 되돌려주는 수수료 비율을 고려해야 합니다. Restaker 가 자신의 ETH 또는 LST 를 위임할 수 있는 많은 운영자가 있을 것이므로, 이는 EigenLayer 에서 restaker 와 운영자 간의 자유 시장을 자연스럽게 형성하게 될 것입니다. 각 EigenLayer 운영자는 이더리움에서 위임 컨트랙트를 제시하며, 이 컨트랙트는 수수료가 어떻게 위임자에게 분배될지 명시하고, 그에 따라 수수료를 라우팅할 것입니다.

## 3. EigenLayer: Slashing Analysis

### 3.1. Slashing Mechanism

암호경제적 보안은 공격자가 프로토콜이 원하는 보안 속성을 잃도록 만들기 위해 부담해야 하는 비용을 정량화한 것입니다. 이를 손상비용(Cost-of-Corruption, CoC)이라고 합니다. CoC 가 잠재적인 손상로 인한 이익(Profit-from-Corruption, PfC)보다 훨씬 클 경우, 우리는 시스템이 견고한 보안을 가지고 있다고 말합니다. 암호경제적 보안은 <b>최소한의 임계값 비율의 운영자가 이타적이며 정직하게 행동할 것이라는 가정 하에만 성립하는 다수 신뢰 보안 보장을 제공하는 시스템</b>과 대조적입니다. EigenLayer 의 핵심 아이디어는 다양한 처벌 메커니즘을 통해 암호경제적 보안을 제공하고, 이를 통해 높은 손상 비용을 부과하는 것입니다.

EigenLayer 스마트 컨트랙트의 핵심 기능 중 하나는 이더리움 Proof-of-Stake (PoS) staker 들의 인출 자격을 보유하는 것입니다. 만약 EigenLayer 에서 resatking 된 staker 가 AVS 에 참여하는 동안 적대적인 행동을 한 것으로 입증되면, 해당 staker 의 ETH 는 처벌을 받게 되고 동결됩니다. 즉, 더 이상 EigenLayer 의 어떤 AVS 에도 참여할 수 없게 됩니다. Staker 의 인출 주소가 EigenLayer 컨트랙트로 설정되어 있기 때문에, staker 가 이더리움 합의에서 EigenLayer 를 통해 ETH 를 인출할 때, 인출된 ETH 는 해당 AVS 의 온체인 처벌 컨트랙트에 따라 처벌을 받게 됩니다.

### 3.2. No Fungible Position on EigenLayer

EigenLayer 는 restaking 된 포지션을 나타내는 대체 가능한 토큰을 발행하지 않습니다. 이는 각 restaker 가 서로 다른 모듈 조합에 참여하여 다양한 처벌 위험을 겪을 수 있기 때문입니다. 이러한 위험이 대체 가능한 포지션 보유자에게 투명하게 전달되도록 보장하는 것은 까다롭고, 대체 가능한 포지션 보유자와 노드를 운영하는 운영자 간의 주인-대리인 문제를 일으킬 수 있습니다. 이는 신중한 관리가 필요하므로, EigenLayer 는 대체 가능한 포지션을 발행할 계획이 없습니다.

### 3.3. Comparison with Merge Mining

EigenLayer 의 restaking 개념은 Bitcoin/Namecoin, Bitcoin/Elastos, Bitcoin/RSK, Litecoin/Dogecoin [10]에서 사용된 병합 채굴 개념과 유사합니다. 작업 증명(PoW) 블록체인의 경우, 검증자에게 가장 큰 비용은 채굴 비용입니다. 그림 6 에서 볼 수 있듯이, 병합 채굴은 동일한 암호학적 PoW 를 사용하여 여러 블록체인에서 동시에 블록을 채굴함으로써 이 비용을 여러 블록체인에 걸쳐 분산시킵니다. 반면, 지분 증명(PoS) 블록체인의 경우 검증자에게 가장 큰 비용은 staking 된 자본의 비용입니다. Restaking 은 이 비용을 여러 모듈 계층에 걸쳐 분산시킵니다.

병합 채굴과 restaking 사이의 유사성은 여기서 끝납니다. 보안 측면에서, restaking 과 병합 채굴은 매우 다릅니다; 병합 채굴은 잠재적인 보안 취약점을 초래할 수 있지만, restaking 은 <b>보안을 강화</b>합니다. 그 이유를 이해하려면 PoW 와 PoS 블록체인을 암호경제학적 관점에서 살펴봐야 합니다. PoW 와 PoS 체인 모두에서, 우리는 메인 체인 검증자 중 일부가 별도의 체인의 검증자로도 활동하는 시나리오를 고려합니다 - PoW 체인의 경우 병합 채굴을 통해, PoS 체인의 경우 restaking 을 통해 이루어집니다. 이 하위 집합이 악의적으로 행동하여 작은 체인을 공격한다고 가정해 봅시다. 예를 들어, 잘못된 상태 전환에 서명하여 체인의 자산을 전송하고 브리징하는 경우입니다. 이제 각 체인 유형에 대한 결과를 살펴보겠습니다:

![Figure6](https://github.com/protocol-diver/paper-translate-kr/blob/main/images/eigen_layer/whitepaper/page_8_1.png)

Figure 6: 병합 채굴 vs restaking.

- <b>Restaking 은 암호경제적 보안을 임의의 하위 집합에 전송합니다.</b> Restaking 을 통해 병합된 PoS 체인의 경우, 다음과 같은 대응이 가능합니다: 잘못된 상태 전환은 메인 체인에서 사기 증명 (fraud-proof)을 통해 처리될 수 있으며, 이 경우 메인 체인에서 악의적인 검증자가 보유한 stake 는 슬래싱 처리됩니다. 이러한 메인 체인으로의 확장은 작은 체인에 restaking 된 양에 비례하는 손상 비용을 발생시킵니다. 우리는 모든 ETH staker 가 EigenLayer 에 참여할 필요 없이 암호경제적 보안을 전송할 수 있다고 주장합니다. 왜냐하면 staker 의 하위 집합만으로도 해당 암호경제적 비용을 부과할 수 있기 때문입니다.

- <b>병합 채굴은 암호경제적 보안을 전송하지 않습니다.</b> PoW 체인의 경우, 메인 체인의 모든 채굴자가 병합 채굴된 체인에 참여하더라도 중요한 암호경제적 보안은 발생하지 않습니다. 첫째, 슬래싱 옵션—즉, 악성 채굴자의 채굴 하드웨어를 비활성화하거나 제거하는 방식—은 사용 불가능합니다. 또한, 더 작은 체인이 토큰 독성으로 인해 가치가 하락하더라도, 즉 병합 채굴된 체인이 더 이상 유용성을 잃어 가치가 떨어지더라도, 채굴자들의 하드웨어 자본은 여전히 주요 체인의 존재로 인해 가치가 유지됩니다. 주요 체인의 토큰은 이와 같은 독성 영향을 경험하지 않기 때문입니다.

### 3.4. Risk Management

우리는 EigenLayer 에서 두 가지 위험 범주를 다룹니다: (1) 많은 운영자가 동시에 일련의 AVS 를 공격하기 위해 공모할 수 있는 위험; (2) EigenLayer 에 구축된 AVS 가 의도치 않은 슬래싱 취약점에 노출될 수 있는 위험 — 이는 정직한 노드가 슬래싱될 위험입니다. 우리는 이 두 가지 위험을 개별적으로 분석합니다.

#### 3.4.1. Operator collusion

우리는 먼저 <b>모든 운영자가 모든 AVS 에 restaking 하는 이상적인 경우</b>를 언급합니다. 이 경우, EigenLayer 에 구축된 AVS 를 손상시키는 비용은 이제 EigenLayer 의 총 staking 금액에 비례합니다. 이는 손상 비용을 최대화하는 측면에서 가장 바람직한 경우입니다. 그러나 실제적인 경우, 일부 운영자만 특정 AVS 에 참여하는 경우, 일부 운영자가 공모하여 AVS 세트에서 자금을 훔치는 복잡한 공격이 발생할 수 있습니다.

$8M 의 restaking 된 ETH 로 보호되는 AVS 가 있고, 총 잠금 가치는 $2M 인 경우를 고려해 봅니다. $2M 의 잠금된 가치를 확보하기 위해 50%의 쿼럼이 필요하므로, 공격이 성공하려면 적어도 공격자의 staking 금액 $4M 이 슬래싱되는 결과가 발생하므로, 애플리케이션은 안전한 것으로 보입니다.

하지만, 만약 동일한 staker 집합이 다른 AVS 에도 restaking 하고 있다면, 상황은 달라질 수 있습니다. 가장 간단한 경우, 정확히 동일한 staker 집합이 10개의 다른 AVS 에 참여하고 있으며, 각각의 AVS 에는 $2M 가 잠겨 있다고 가정합니다. 이 경우, 이 staker 집단을 손상시키는 총 이익은 $20M 이지만, 총 staking 금액은 $8M 에 불과하여 시스템은 암호경제적으로 불안정하게 됩니다.

하나 중요한 사실은 공격이 객관적으로 귀속될 수 있다고 하더라도, 이더리움이 영향을 받은 당사자들에게 보상을 하기 위해 하드 포크를 수행할 것이라고 기대할 수 없다는 점입니다. 특히 ETH staker 의 일부만이 restaking 에 참여하는 경우에는 더욱 그렇습니다. 물론, 많은 ETH staker 들이 악의적인 방식으로 행동하여 중요한 AVS 가 실패하게 만든다면, 커뮤니티는 하드 포크를 조정할 수 있습니다.

하나의 해결책은 특정 AVS 의 PfC 를 제한하는 것입니다. 예를 들어, (1) 브릿지는 슬래싱 기간 내에 가치 흐름을 제한할 수 있고, (2) 오라클은 해당 기간 내에 거래되는 총 가치를 제한할 수 있습니다. 그러나 이 해결책은 해당 AVS 의 설계자에게 의존합니다. 또 다른 해결책은 EigenLayer 가 AVS 를 손상시키는 비용(CoC)을 적극적으로 증가시키는 방식입니다. 여기에서는 어떤 staker 집합도 공모할 수 있다는 점을 고려하여 restaking 보안에 대한 일반화된 분석을 제공합니다. 만약 해당 집합이 특정 AVS 에서 다수결 쿼럼을 형성하면, 그들은 잠재적으로 해당 AVS 에서 PfC 를 추출할 수 있습니다. 우리는 EigenLayer 와 함께 restaking 된 운영자 또는 운영자 집합이 공모를 통해 보안 취약점을 생성할 위험이 있는지 여부를 판단하는 메커니즘을 갖추고 있습니다. 오픈소스 암호경제 대시보드를 생성함으로써, EigenLayer 에 구축된 AVS 는 자신들의 검증 작업에 참여하는 운영자 집합이 다른 많은 AVS 에서 고착화되어 있는지 여부를 모니터링할 수 있게 됩니다. 만약 그렇다면, AVS 는 서비스 컨트랙트에서 오직 적은 수의 AVS 에만 참여하는 EigenLayer 운영자만을 인센티브로 제공하는 규격을 명시할 수 있습니다. 이렇게 되면 EigenLayer 는 탄력적인 보안을 가진 것으로 생각할 수 있습니다. 자세한 암호경제적 위험 분석은 섹션 B 를 참조하십시오.

#### 3.4.2. Unintended slashing

우리는 잘 개발된 프로토콜처럼, EigenLayer 에 구축된 AVS 의 목표는 AVS 가 충분히 시험을 거친 후 점진적으로 경직되는 것임을 주목합니다. AVS 가 경직되면, 의도치 않은 슬래싱의 위험은 최소화될 것이라고 가정합니다. 그러나 AVS 와 그 관련 인프라 및 컨트랙트가 충분히 시험되지 않은 상태에서는 위험 연쇄를 피하기 위해 EigenLayer 에서 다양한 슬래싱 위험을 완화해야 합니다. 하나의 위험은 AVS 가 의도치 않은 슬래싱 취약점(예: 프로그래밍 버그)을 가지고 생성되어 그것이 트리거되어 정직한 사용자에게 자금 손실을 초래하는 경우입니다.

우리는 여기에서 두 가지 방어 수단을 제안합니다: (1) 보안 감사; (2) 슬래싱 이벤트에 대한 거부권. 보안 감사와 관련하여, 우리는 AVS 코드베이스가 스마트 컨트랙트 감사와 마찬가지로 감사되어야 한다고 인식합니다. AVS 코드베이스는 평균적인 스마트 컨트랙트보다 더 복잡할 수 있지만, 스마트 컨트랙트 감사가 일반 대중인 소비자 사용자를 보호하는 데 중점을 두는 것과 달리, AVS 감사는 블록체인 생태계에서 더 숙련된 참여자인 staker 와 운영자를 대상으로 한다는 점에서 차이가 있습니다. 이러한 숙련도와 리스크/보상 프로파일을 고려할 때, 우리는 AVS 가 staker 와 운영자들로부터 옵트인(opt-in)을 받기 위해 적절한 감사가 필요할 것이라고 예상합니다. AVS 가 고착되기 전에 두 번째 방어 수단은 Eigenlayer 의 거버넌스 계층이 있다는 점입니다. 이 계층은 이더리움과 EigenLayer 커뮤니티의 저명한 구성원들로 이루어져 있으며, 멀티시그를 통해 슬래싱 결정을 거부할 수 있는 능력을 가지고 있습니다. 우리는 슬래싱 거부 과정이 결국 제거될 훈련 바퀴와 비슷하다고 생각합니다.

### 3.5. Governance

슬래싱을 거부할 수 있는 권한을 가진 역할임을 고려할 때, 이 멀티시그 거부 위원회를 어떻게 구성할지 신중하게 고려하는 것이 중요합니다. 이 거부 위원회를 선택하는 방법 중 하나는 토큰 기반의 쿼럼을 사용하는 것입니다. 그러나 토큰 기반의 쿼럼을 사용하면 거버넌스가 중앙화된 주체가 토큰의 대부분을 구매하여 자신의 규칙을 지시하거나 더 나아가 시스템을 직접 공격하는 데 취약해질 수 있습니다. 이러한 시나리오를 피하기 위해, 우리는 이더리움과 EigenLayer 커뮤니티의 명망 있는 인물들로 구성된 명성 기반의 위원회를 사용합니다. 이 위원회는 EigenLayer 컨트랙트의 업그레이드를 활성화하고, 슬래싱 이벤트를 검토 및 거부하며, 새로운 AVS 를 슬래싱 검토 과정에 포함시키는 책임을 집니다. 이 거부 위원회는 악의적으로 슬래싱을 트리거할 권한을 가지고 있지 않으며, EigenLayer 컨트랙트의 업그레이드는 시간 지연을 동반한다는 점을 유의해야 합니다.

이 거부 위원회는 새로운 AVS 가 EigenLayer 에 온보딩될 때의 트레이닝 휠로 간주될 수 있습니다. AVS 는 이 거부 위원회를 사용하여 EigenLayer 의 restaker 들에게 악의적인 슬래싱이나 버그로 인한 부정확한 슬래싱에 노출되지 않을 것임을 보장할 수 있습니다. 왜냐하면 슬래싱이 발생하면 항상 거부할 수 있는 위원회가 있기 때문입니다. 한편, AVS 개발자들은 AVS 와 관련된 코드베이스에 대한 실전 테스트를 할 수 있습니다. AVS 가 성숙해지고 restaker 들로부터 충분한 신뢰를 얻으면, 더 이상 거부 위원회를 백업으로 사용할 필요가 없어질 수 있습니다. 거부 위원회를 사용할 때의 신뢰 가정은, AVS 가 거부 위원회가 올바른 슬래싱을 거부하지 않을 것이라고 신뢰해야 하며, EigenLayer 에서 restaking 하는 staker 들은 거부 위원회가 AVS 의 부당한 슬래싱을 반드시 거부할 것이라고 신뢰해야 한다는 것입니다.

EigenLayer 위에 구축하고 거부 위원회를 사용하려는 AVS 는 거부 위원회에 의해 승인되어야 합니다. 온보딩 과정은 보안 감사 및 위원회 구성원들의 기타 실사 절차를 포함할 수 있으며, 여기에는 운영자가 AVS 를 서비스할 수 있는 시스템 요구 사항을 확인하는 작업도 포함됩니다.

### 3.6. Designing Modules for Maximal Security Also Minimizes Centralization Risk

우리는 모든 ETH 가 EigenLayer 에 restaking 되어 특정 AVS 를 보호하는 데 사용될 때 가장 높은 보안을 얻을 수 있다는 점을 주목합니다. 그러나 이를 방해하는 두 가지 요소가 있습니다: (1) AVS 의 예상 수익이 운영자의 운영 비용을 초과하는지 여부; (2) 운영자가 AVS 검증에 참여할 만큼 충분한 컴퓨팅 자원을 가지고 있는지 여부. 우리는 이러한 문제를 완화할 수 있도록 모듈을 설계할 수 있는 두 가지 가능한 패턴을 제안합니다.

1. Hyperscale AVS: Hyperscale AVS 에서는 전체 컴퓨팅 작업 부하가 N 개의 참여하는 운영자 노드에 걸쳐 분배됩니다. 이 기능은 분산 컴퓨팅에서 수평 확장, 스케일 아웃이라고 불립니다. Hyperscale 모듈의 대표적인 예는 hyperscale data availability protocol 으로, 여기서 전체 데이터는 N 개의 청크로 나누어지며, 각 청크의 크기는 원래 데이터의 2/N 크기입니다. 따라서 데이터를 저장하는 총 비용은 마치 2개의 노드만 데이터를 저장하는 것처럼 됩니다. Hyperscale AVS 에서는 각 노드의 처리 요구 사항이 작을 수 있지만, 시스템 자체는 많은 노드들의 성능을 집합적으로 활용하여 높은 처리량을 달성할 수 있다는 점을 주목해야 합니다.
<b>또한, hyperscale AVS 는 기존 블록체인 시스템보다 중앙 집중식 검증에 대한 유인이 적습니다.</b> 이는 기존 블록체인(수평 확장이 이루어지지 않은 시스템)에서는 검증 비용이 중앙 운영자에 의해 완전히 분배될 수 있기 때문입니다. 즉, 한 번 유효하다고 확인된 상태 전환은 모든 운영자 키에서 사용될 수 있습니다. 그러나 수평 확장된 모듈에서는 중앙 집중식 당사자가 각 키에 대해 다른 검증을 수행해야 하므로, 분배 이득이 최소화됩니다.
2. 경량화: 모든 운영자가 중복적으로 수행하는 작업 중 비용이 매우 낮고, 필요한 컴퓨팅 인프라 또한 매우 낮은 예시들도 있습니다. 예를 들어, 라이트 클라이언트를 실행하여 특정 메시지에 대한 증명을 하는 것, 제로 지식 증명을 검증하는 것, 다른 블록체인의 라이트 노드를 실행하는 것, 오라클 가격 피드를 실행하는 것 등은 EigenLayer 에서 실행할 수 있는 경량화된 적극적으로 검증된 서비스들입니다.

우리는 hyperscale 되고 경량화된 AVS 의 모음을 설계하여 EigenLayer 에서 제공되는 대부분의 수익을 창출함으로써, 이더리움의 home 검증자들도 EigenLayer 의 대부분의 경제적 혜택을 얻을 수 있도록 보장할 수 있다고 주목합니다. 이렇게 하면 이더리움 staking 에 대한 중앙 집중화 압박을 최소화할 수 있습니다.

## 4. A World with EigenLayer

### 4.1. EigenLayer Enables New Applications

EigenLayer 에 의해 활성화된 새로운 AVS 의 범위는 상당히 넓으며, 새로운 블록체인, 미들웨어, 그리고 데이터 가용성 계층과 같은 모듈형 블록체인 레이어를 포함합니다. 여기에서 몇 가지 가능성을 나열하며, 이들 중 많은 것이 현재 진행 중이거나 향후 연구에서 주목받는 흥미로운 방향입니다:

![Figure7](https://github.com/protocol-diver/paper-translate-kr/blob/main/images/eigen_layer/whitepaper/page_11_1.png)

그림 7: 블록 공간은 블록체인에서 가장 가치 있는 자원입니다 [11]. 현재 생태계에서는 이 블록 공간의 blocklimit 이 가장 약한 검증자의 인프라에 의해 결정됩니다. 이 제한은 이더리움이 탈중앙화를 중시하는 데서 비롯됩니다. 그러나 EigenLayer 는 추가 자원을 보유한 검증자의 추가 역량을 활용하여, 이를 원하는 애플리케이션에 암호경제적 보안 보장을 제공할 수 있습니다.

1. <b>하이퍼스케일 데이터 가용성 레이어 (Hyperscale AVS):</b> EigenLayer restaking 과 이더리움 커뮤니티에서 개발된 Danksharding 을 포함한 최첨단 데이터 가용성(DA) 기술을 활용하여, 높은 DA 속도와 낮은 비용을 제공하는 하이퍼스케일 DA 레이어를 구축할 수 있습니다.
2. <b>탈중앙화 시퀀서 (경량 / 하이퍼스케일 AVS):</b> 많은 롤업은 MEV 관리 및 검열 저항성을 위해 탈중앙화된 시퀀서를 필요로 합니다. 이러한 시퀀서는 EigenLayer 에서 ETH staker 들의 정족수를 기반으로 구축될 수 있으며, 여러 롤업에 서비스를 제공하는 단일 탈중앙화 시퀀서 정족수가 존재할 수도 있습니다. 또한, 탈중앙화 시퀀서는 반드시 실행을 수행할 필요가 없으며, 상태 증가 문제가 없는 단순한 정렬 계층으로만 작동할 수 있습니다. 따라서 이를 경량화하거나, 합의 노드의 랜덤 하위 집합을 선택하여 서로 다른 트랜잭션 집합을 정렬하도록 함으로써 수평적으로 확장하는 것도 가능합니다.
3. <b>라이트 노드 브릿지 (경량 AVS):</b> EigenLayer 를 사용하면 이더리움과 연결되는 라이트 노드 브릿지를 쉽게 구축할 수 있습니다. 예를 들어, NEAR 와 이더리움을 연결하는 Rainbow Bridge 는 낙관적 패턴을 기반으로 하지만, 검증에 높은 가스 비용이 들기 때문에 지연 시간이 길다는 문제가 있습니다. restaker 들은 오프체인에서 브릿지 입력이 올바른지 검증할 수 있으며, 강력한 암호경제적 정족수가 해당 입력을 승인하면 브릿지 입력이 유효한 것으로 간주됩니다. 만약 누군가 이의를 제기하면, 브릿지 입력을 검증할 수 있으며, 이 과정에서 EigenLayer 의 검증자들은 느린 (비낙관적) 모드에서 슬래시될 수 있습니다.
4. <b>롤업을 위한 패스트 모드 브릿지 (경량 AVS):</b> ZK 롤업의 경우, 이더리움에서 증명 검증 비용이 여전히 높기 때문에 롤업 시퀀서는 이더리움에 데이터를 자주 기록하지 못하며, 이는 구성 가능성 (composability) 에 영향을 미치고 확정성 보장을 지연시킵니다. EigenLayer 에서 대량의 ETH를 restaking 한 운영자 정족수가 오프체인에서 ZK 증명 검증에 참여하고, 온체인에서 증명의 정확성을 인증할 수 있습니다. 만약 패스트 모드 브릿지가 잘못된 주장을 한다면, 더 느린 슬래싱 프로세스를 활성화할 수 있습니다. 낙관적 롤업의 경우, EigenLayer 는 훨씬 더 큰 담보 풀을 활용하여 상태 루트 인증에 참여하도록 유도할 수 있으며, 잘못된 인증 시 슬래싱 위험을 부과할 수 있습니다.
5. <b>오라클 (경량 AVS):</b> 이더리움에 가격 피드를 내장하거나, 유니스왑 토큰 정족수를 활용하여 가격 피드를 제공하는 방안이 제안된 바 있습니다. 이러한 오라클은 EigenLayer 를 통해 구축될 수 있으며, EigenLayer 에 restaking 된 ETH 에 대한 과반수 신뢰만을 요구하는 선택적(opt-in) 계층으로 운영될 수 있습니다.
6. <b>옵트인 이벤트 기반 활성화 (경량 AVS):</b> 청산 및 담보 이전과 같은 이벤트 기반 활성화는 현재 이더리움에서 기본적으로 제공되지 않습니다. 이러한 기능은 별도의 계층인 keeper 네트워크에서 구축될 수 있지만, 블록 공간을 관리하지 않는 keeper 노드는 이벤트 기반 작업의 포함에 대해 강력한 보장을 제공할 수 없습니다.
EigenLayer 에서는 블록 제안자이기도 하며, 이벤트 기반 활성화 AVS 를 위해 EigenLayer 에서 restaking 에 옵트인한 이더리움 검증자들이 이벤트 기반 작업의 포함에 대한 강력한 보장을 제공할 수 있습니다. 단, 이들은 슬래싱 위험을 감수해야 합니다.
7. <b>옵트인 MEV 관리:</b> EigenLayer 에서는 Proposal-Builder Separation [14], MEV smoothing [15], 트랜잭션 포함을 위한 임계값 암호화 [16, 17] 등 다양한 옵트인 MEV 관리 방법들이 실현 가능합니다. 간단한 예로, MEV smoothing은 EigenLayer 위에 restaking 한 그룹이 MEV 를 구성원 간에 고르게 나누기로 결정하여 구축할 수 있습니다. 규정된 MEV smoothing 행동에서 벗어나는 restaker 는 슬래시될 수 있습니다. 블록 제안자만 특정 작업을 수행하면 되기 때문에 자연스럽게 수평적으로 확장 가능합니다.
8. <b>초저지연 결제 체인:</b> 이더리움은 경제적 최종성을 얻는 데 높은 지연 시간이 있으며(최대 12분), 따라서 빠른 결제와 높은 경제적 최종성이 유용할 수 있습니다. EigenLayer 는 ETH restaker 들이 새로운 합의 프로토콜에 참여할 수 있는 restaking 된 사이드체인의 생성을 허용하며, 이들 중 일부는 매우 낮은 지연 시간과 매우 높은 처리량을 자랑합니다. 결제 계층은 상태 성장이 필요하지 않다는 점을 주목할 수 있습니다. ZK 증명을 정산하는 것은 거의 상태가 없는 작업이기 때문입니다(일부 최신 상태 루트는 계약 상태로 유지될 수 있음). 또한, 결제 계층은 많은 ZK 증명을 병렬로 검증할 수 있어 높은 병렬 처리도를 가질 수 있습니다. 이는 경량 AVS 가 될 수 있습니다.
9. <b>싱글 슬롯 최종성 (경량 AVS):</b> EigenLayer 에서 옵트인 메커니즘을 통해 노드들이 블록의 최종성에 서명하는 싱글 슬롯 최종성을 상상할 수 있습니다 [18]. 핵심 아이디어는 restaking 한 노드들이 이제 증언된 블록을 포함하지 않는 체인에서 블록을 생성하지 않을 것임을 증명할 수 있다는 것입니다. 이를 통해 잠재적인 최종성 경로를 만들 수 있습니다. 이를 진정으로 옵트인 방식으로 설계하고, 합의 프로토콜을 깨지 않도록 만드는 것이 중요한 연구 방향입니다.

### 4.2. EigenLayer Leverages Staker Heterogeneity, and Massively Expands Blockspace

모든 블록체인은 네트워크에 허용할 수 있는 가장 약한 노드 인프라를 기준으로 blocklimit 을 결정합니다. 그림 7은 노드 간 자원이 어떻게 크게 달라지는지에 대한 예시를 보여줍니다. 블록체인의 요구 사항보다 더 많은 컴퓨팅 자원을 가진 staker 들은 현재 모든 블록체인에서 일반적인 동질적인 아키텍처 내에서는 초과 자원을 활용할 수 없습니다. EigenLayer 는 높은 용량을 가진 노드만 수행할 수 있는 옵트인 검증 작업을 만들어 컴퓨팅 이질성을 활용할 수 있습니다. 이는 낮은 암호경제적 안전성으로 이어지지만, 이는 위험에 처한 가치를 측정 가능하게 합니다. 탈중앙화에 의존할 수 있는 liveness 는 쉽게 정량화되지 않을 수 있지만, EigenLayer 위에 구축된 시스템은 롤업처럼 이더리움 메인 체인을 백업으로 활용하여 liveness 와 검열 저항을 지원할 수 있다는 점을 주목할 수 있습니다.

Staker 간의 이질성은 컴퓨팅 이질성을 훨씬 넘어서고 있습니다. Staker 는 위험 선호도에 따라 중요한 차이를 보입니다. 예를 들어, 일부 restaker 는 자신들의 오라클 입력이 잘 알려진 거래소의 입력과 다를 경우, 오라클 모듈에 참여하면서 슬래싱을 감수할 의향이 있을 수 있습니다. 반면 다른 restaker 에게는 이 위험이 너무 높다고 여겨질 수 있습니다. 보상 선호도에도 강한 차이가 존재합니다. 일부 restaker 는 탈중앙화된 소셜 네트워크와 같은 특정 분야에 베팅을 하고, 자신이 보유한 초과 컴퓨팅 자원을 그런 소셜 네트워크의 검증 및 부트스트랩을 돕는 데 할당하려 할 것입니다. 이들은 서비스에 대해 비유동적인 토큰 형태로 수수료를 받을 수 있습니다. 반면, 다른 restaker 는 즉시 양도 가능하고 유동적인 보상만 가치를 가진다고 생각할 수 있습니다. 또한, restaker 들은 검증 가능한 자격 증명(Verifiable Credentials), 소울 바운드 토큰(Soul-Bound Tokens) [19] 및 기타 기술과 자산과의 소유 또는 상호작용에 따라 다양한 특징을 가질 수 있습니다. 이러한 식별 가능한 특징은 restaker 들이 특정 모듈 검증 작업을 수행하도록 모집되는 기준이 될 수 있습니다. 따라서 EigenLayer 는 restaker 간의 컴퓨팅 용량, 위험 선호도, 보상 선호도 및 정체성에 따른 이질성을 표현할 수 있도록 하며, 또한 모듈들이 이러한 선호도와 특성의 조합에 따라 restaker 를 모집할 수 있게 합니다.

우리는 DApp 의 성능/보안 파라미터를 조정하는 개념이 탈중앙화 관점에서 EigenLayer 와 이더리움 사이의 철학적인 차이를 나타내지 않는다는 점을 강조합니다. 탈중앙화는 여전히 이더리움에서 liveness 를 보장하는 중요한 부분이며, EigenLayer 와 그 모듈들은 이더리움에 직접적으로 의존합니다. 또한, 앞서 논의한 바와 같이, EigenLayer 에서 많은 수수료 수익을 창출하는 하이퍼스케일 및 경량 검증 작업을 구축함으로써, 심지어 홈 staker 들조차 EigenLayer 에서 제공되는 수익의 상당 부분을 계속해서 누릴 수 있으며, 따라서 네트워크는 중앙집중화 압력을 피할 수 있습니다.

### 4.3. EigenLayer Breaks the Trade-Off between Democracy and Agility

![Figure8](https://github.com/protocol-diver/paper-translate-kr/blob/main/images/eigen_layer/whitepaper/page_13_1.png)

그림 8: 현재 형태의 이더리움은 민주적인 거버넌스를 제공하지만 혁신 속도가 느립니다. EigenLayer 가 추가되면 이더리움은 두 가지 장점을 모두 누릴 수 있습니다: 혁신의 민첩성과 민주적인 거버넌스를 동시에 갖춘 시스템입니다.

이더리움 프로토콜은 강력한 오프체인 거버넌스 방식을 통해 업그레이드되며, 그 결과 필연적으로 느린 속도를 보입니다. 그림 8에서 볼 수 있듯이, Binance Smart Chain(BSC) 과 같은 경쟁 프로토콜들은 더 빠른 의사결정과 업데이트를 달성하기 위해 보다 상향식(top-down)인 기업형 거버넌스를 채택해왔습니다. 이러한 의사결정 방식은 효율적이지만, 민주적이지는 않습니다. 오늘날 모든 프로토콜은 민주적 거버넌스와 혁신 속도 사이에서 일정한 트레이드오프를 하게 됩니다.

EigenLayer 의 출현은 민주적 거버넌스와 민첩한 혁신 사이의 트레이드오프가 존재한다는 개념에 도전합니다. 실제로 EigenLayer 는 이더리움 신뢰 네트워크 위에 민첩한 혁신을 구축할 수 있게 하면서, 이더리움의 핵심은 그대로 두어 신중하고 안정적인 방식으로 계속해서 업그레이드될 수 있도록 합니다. 이렇게 하면 두 가지 장점을 모두 얻을 수 있습니다. 또한, EigenLayer 에서 가능한 무허가 혁신의 속도는 상향식 거버넌스 모델에서 가능한 혁신 속도를 훨씬 초과합니다. 상향식 모델에서는 의사결정이 소수의 엔터티나 개인에게만 제한되기 때문입니다.

이 구성에서는 이더리움 기본 계층이 신중하게 업그레이드되어 장기적인 안정성을 제공하며, 단기적으로 필요한 민첩한 혁신들은 자유 시장을 통해 EigenLayer 를 통해 분배됩니다. 사실, EigenLayer는 이더리움의 스테이징 네트워크 역할을 할 수 있습니다. 여기서 경쟁적인 아이디어들이 무허가 방식으로 구현된 후, 이더리움에 통합되고 흡수됩니다. 우리는 이미 이 스테이징 네트워크에서 Danksharding 기반의 데이터 가용성 계층을 구현 중에 있으며, 이를 통해 얻은 교훈들이 미래의 이더리움 설계에 도움이 될 것입니다.

따라서 이더리움과 EigenLayer 는 <b>이더리움을 위한 신중하고 보수적인 거버넌스와 민첩하고 무허가 혁신</b>을 동시에 가능하게 합니다.

### 4.4. EigenLayer Can Incentivize Ethereum Staker Decentralization

모든 분산 시스템에서는 stake 와 검증자 노드가 중앙집중화되는 자연스러운 경향이 있습니다. 그러나 많은 AVS (어플리케이션 검증 시스템)에서는 stake 와 검증자 노드가 가능한 한 탈중앙화되기를 원합니다. 하나의 예로, 온체인 프라이버시를 위한 임계값 암호화 시스템이 있습니다. 만약 다수의 키가 하나의 중앙 집중화된 엔터티에 의해 보유된다면, 프라이버시는 이 중앙집중화된 엔터티에 의해 규정된 기간 전에 비대상적인 방식으로 파괴될 수 있습니다. 따라서 임계값 키 보유자의 집합은 가능한 한 탈중앙화되어야 하며, 이를 통해 규정된 기간 전에는 아무도 실제 내용을 알 수 없도록 해야 합니다.

EigenLayer 를 사용하면 AVS 는 검증 작업에 참여하는 stake/노드가 탈중앙화된 쿼럼의 일원이어야 한다고 명시적으로 지정할 수 있습니다. 더 구체적으로, AVS 는 EigenLayer 와 통합할 때 계약에서 오직 이더리움 홈 검증자만 해당 작업에 참여할 수 있도록 지정할 수 있으며, 이를 통해 AVS 의 탈중앙화를 유지하는 데 기여합니다. AVS 의 검증 작업에 누가 참여할 수 있는지를 무허가 방식으로 지정할 수 있는 이 능력은 탈중앙화를 프로그래밍하는 것과 유사합니다.

추상적인 관점에서, EigenLayer 는 AVS 가 탈중앙화를 구매할 수 있는 시장을 가능하게 합니다. 점점 더 많은 AVS 가 EigenLayer 에서 오직 홈 검증자만 그들의 작업에 참여할 수 있도록 지정함에 따라, 이는 이더리움에서 홈 검증자 노드를 운영하는 것이 더 많은 수익을 얻을 수 있게 만들고, 결과적으로 탈중앙화를 장려합니다.

### 4.5. Multi-Token Quorums

EigenLayer 는 AVS 가 자신만의 쿼럼을 정의할 수 있는 유연성을 제공합니다. 이 쿼럼은 restaked ETH 로 구성된 쿼럼과 함께 사용되며, 검증 작업에 대한 최종 응답은 각 쿼럼의 다수의 응답을 함수로 처리해야 합니다. 예를 들어, AVS 는 두 개의 쿼럼을 지정할 수 있습니다: 이더리움 restakers 쿼럼과 $AVS 쿼럼(여기서 $AVS 는 AVS 의 토큰입니다). EigenLayer 에서 이 AVS 에 대한 서비스를 제공하려면, 어떤 운영자는 ETH 를 다시 staking 하거나 $AVS 토큰을 staking 해야 합니다. AVS 는 두 쿼럼을 독립적인 쿼럼으로 취급하고, AND 조건을 사용하여 두 쿼럼의 다수 응답을 결합할 수 있습니다.

여러 쿼럼을 정의할 수 있는 이 유연성은 AVS 에게 자사의 토큰을 유틸리티 토큰으로 부트스트랩하고, 프로토콜에 가치를 축적할 수 있는 기회를 제공합니다. 동시에, restaked ETH 쿼럼을 사용하여 자사의 토큰이 가치 하락을 겪는 위험을 헤지할 수 있습니다.

### 4.6. Business Models on EigenLayer

다음은 EigenLayer 위에 AVS 가 구축할 수 있는 비즈니스 모델입니다:

1. 순수 지갑 모델: 이 모델에서는 회사가 상업적 서비스로 EigenLayer 위에 AVS 를 배포합니다. AVS 를 사용하는 사용자는 중립적인 단위로 수수료를 지불해야 합니다: (1) 수수료의 일부는 서비스에 대한 대가로 회사 지갑으로 가며, (2) 나머지 수수료는 EigenLayer 의 ETH restakers 와 EigenLayer 프로토콜로 전달됩니다. 이는 순수한 회사 기반 비즈니스 모델을 가능하게 하고, AVS 가 암호화 공간 내에서 SaaS 경제학을 구축할 수 있게 합니다.
2. 수수료 토큰화 모델: 이 모델에서는 AVS 가 상업적 서비스가 아니라 프로토콜로 운영됩니다. AVS 를 사용하는 사용자는 중립적인 단위로 수수료를 지불해야 합니다: (1) 수수료의 일부는 프로토콜에 의해 지정된 대로 $AVS(AVS 의 네이티브 토큰) 토큰 보유자들의 쿼럼에 전달되며, (2) 나머지 수수료는 EigenLayer 의 ETH restakers와 EigenLayer 프로토콜로 전달됩니다.
3. AVS 의 네이티브 토큰으로 지불하는 모델: 이 모델에서는 AVS 가 프로토콜로 운영되며, 프로토콜의 사용자는 AVS 가 발행한 특정 토큰인 $AVS 의 단위로 수수료를 지불해야 합니다. 이 토큰 $AVS 의 가치는 AVS 가 미래에 지속적으로 수익성 있게 운영될 것이라는 기대에 의존합니다. 수수료는 두 부분으로 구성됩니다: (1) 수수료의 일부는 프로토콜에 의해 지정된 대로 토큰 보유자들의 쿼럼에 전달되며, (2) 나머지 수수료는 EigenLayer 의 ETH restakers 와 EigenLayer 프로토콜로 전달됩니다.
4. Dual staking utility. Under this model, the AVS also issues its own token $AVS and EigenLayer features two quorums: the first quorum is composed of ETH restakers, and the second quorum is composed of $AVS stakers. In the dual quorum model, safety is the better of the two quorums, and liveness is the worst of the two quorums. Now, anyone with either ETH or $AVS can participate in providing security to the AVS via EigenLayer by restaking ETH or staking ($AVS) in their respective quorums. This leads to dual staking in EigenLayer, which can drive utility to the AVS’s token $AVS, while using ETH restaking to hedge against a potential death spiral of $AVS which would compromise the security of the AVS.
4. 이중 staking 유틸리티 모델: 이 모델에서는 AVS 가 자체 토큰인 $AVS 를 발행하고, EigenLayer 는 두 개의 쿼럼을 특징으로 합니다: 첫 번째 쿼럼은 ETH restakers 로 구성되고, 두 번째 쿼럼은 $AVS staker 들로 구성됩니다. 이중 쿼럼 모델에서는 안전성은 두 쿼럼 중 더 좋은 것이며, liveness 는 두 쿼럼 중 더 나쁜 것입니다. 이제 ETH 또는 $AVS 를 보유한 누구든지 EigenLayer 를 통해 AVS 의 보안을 제공하는 데 참여할 수 있습니다. 이를 위해 ETH를 restake하거나 $AVS를 staking 하여 각각의 쿼럼에서 보안을 제공할 수 있습니다. 이로 인해 EigenLayer 에서 이중 staking 이 발생하며, 이는 AVS의 토큰인 $AVS에 유틸리티를 부여하면서, ETH restaking 을 사용하여 $AVS의 가치 하락 위험을 헤지하고 AVS의 보안을 유지할 수 있습니다.

## 5. Summary

1. EigenLayer 는 Ethereum staker 에서 stake 와 검증 서비스가 필요한 모듈로 제공되는 탈중앙화된 신뢰를 위한 자유 시장을 창출합니다.
2. EigenLayer 를 통해 restaking을 하면, Ethereum staker 는 선택한 모듈에 보안 및 검증 서비스를 제공하기 위해 직접 노드를 운영하거나 다른 EigenLayer 운영자에게 위임하는 방식으로 참여할 수 있습니다.
3. 다양한 경량 및 하이퍼스케일 모듈이 EigenLayer 위에 구축될 수 있으며, 이는 독립적인 staker 들이 널리 참여할 수 있도록 설계될 수 있습니다.
4. 모듈은 또한 계산 용량, 리스크/보상 선호도, 신원 등의 차이로 인한 staker 간의 중요한 이질성을 활용할 수 있습니다.
5. EigenLayer 는 블록체인에서 더 민첩하고, 탈중앙화된, 그리고 허가 없는 혁신을 가능하게 하려는 목표를 추구합니다.

## References

[1] Satoshi Nakamoto. Bitcoin: A peer-to-peer electronic cash system. Decentralized business review, page 21260, 2008.

[2] Vitalik Buterin et al. Ethereum white paper.

[3] Ethereum: A secure decentralized generalized transaction ledger. https://ethereum.github.io/yellowpaper/paper.pdf

[4] Rollup-centric ethereum roadmap. https://ethereum-magicians.org/t/a-rollup-centric-ethereum-roadmap/4698

[5] An incomplete guide to rollups. https://vitalik.ca/general/2021/01/05/rollup.html

[6] Lido.fi. https://lido.fi/

[7] Rocketpool. https://rocketpool.net/

[8] Superfluid staking. https://docs.osmosis.zone/osmosis-core/modules/superfluid/

[9] The cryptoeconomics of slashing. https://a16zcrypto.com/the-cryptoeconomics-of-slashing/

[10] Merged mining. https://developers.rsk.co/rsk/architecture/mining/

[11] Blockspace: An introduction with chris dixon. https://www.generalist.com/briefing/ blockspace.

[12] Rainbow bridge. https://near.org/bridge/

[13] Uni should become an oracle token. https://gov.uniswap.org/t/uni-should-become-an-oracle-token/11988

[14] State of research: increasing censorship resistance of transactions under proposer/builder separation (pbs). https://notes.ethereum.org/@vbuterin/pbs_censorship_resistance

[15] Committee-driven mev smoothing. https://ethresear.ch/t/committee-driven-mev-smoothing/10408

[16] Shutter - in-depth explanation of how we prevent front running. https://blog.shutter.network/shutter-in-depth-explanation-of-how-we-prevent-frontrunning/

[17] Removing trusted relays in mev-boost using threshold encryption. https://ethresear.ch/t/removing-trusted-relays-in-mev-boost-using-threshold-encryption/13449

[18] Paths toward single-slot finality. https://notes.ethereum.org/@vbuterin/single_slot_finality

[19] Soulbound. https://vitalik.ca/general/2022/01/26/soulbound.html

# Appendix...

[Appendix](https://docs.eigenlayer.xyz/assets/files/EigenLayer_WhitePaper-88c47923ca0319870c611decd6e562ad.pdf)