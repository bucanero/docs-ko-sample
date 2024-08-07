---
sidebar_label: "인덱서 소개"
---

# 인덱서 소개

여기에서 인덱서 개념에 익숙해지고 자신만의 인덱서를 구축하기 위해 알아야 할 모든 것을 찾을 수 있습니다.

:::info Disclaimer

이 페이지의 설명은 당신이 블록체인 기술에 대해 어느 정도 이해하고 있다고 가정합니다.

:::


### 블록체인과 그 성질

블록체인 데이터는 체인이 생성될 때, 한 번에 한 블록씩 직렬화된 형태의 **쓰기**에 최적화되어 있습니다. 특정 블록이나 계정에 대한 데이터를 블록체인에 쿼리하는 것은 매우 간단한 형태이거나 "좁은" 쿼리입니다. 그러나 여러 단일 블록 쿼리의 결과를 집계해야 할 때 여러 블록에서 데이터를 각각 쿼리하는 것은 번거로울 수 있습니다. 따라서 *"넓은" 쿼리* 를 고려할 수 있습니다.

블록체인 자체가 분산 데이터베이스이고, 스마트 컨트랙트(탈중앙화 애플리케이션, dApp)는 블록체인 내부의 가상 머신에서 실행되는 애플리케이션이라는 사실을 감안할 때, 스마트 컨트랙트 를 "백엔드"로 간주 해서는 *안 된다는* 점을 이해해야 합니다. 일부 애플리케이션은 스마트 컨트랙트만으로 구성될 수 있지만, 스마트 컨트랙트만으로 dApp을 구축하는 것은 대부분의 경우 불가능합니다.

스마트 컨트랙트는 상호 작용 측면에서 제한적입니다. "상호 작용"이란 사용자 알림, 타 애플리케이션과의 통합 등과 같이 현실 세계에서 매우 일반적인 것을 의미합니다.

그러나 블록체인의 특성은 *결정론적이어야* 한다는 것입니다. 블록체인의 중요한 특징은, 블록체인은 주어진 시점에서의 상태를 알고 있으며 블록체인의 경우 그 시간의 단위가 블록이라는 것입니다. 그것들을 스냅샷이라고 생각해보세요. 블록체인은 모든 블록의 상태에 대한 스냅샷을 수행합니다. 우리는 사용자로서 특정 블록에 대한 스마트 컨트랙트를 호출할 수 있으며, 블록체인은 호출을 받을 때마다 동일한 블록에 대해 항상 동일한 결과를 생성하도록 보장합니다.

블록체인의 결정론적 특성은 외부(오프체인) 변수로부터 블록체인을 닫습니다. 스마트 컨트랙트 내에서 API 호출을 수행하는 것은 완전히 불가능합니다. 블록체인과 스마트 컨트랙트는 외부(오프체인) 세계와 차단됩니다.

![Blockchain closed from outer world](/docs/intro/blockchain.png)

블록체인은 요청된 변경 사항을 분산된 방식으로 상태에 적용하는 방법을 제공하는 데 탁월합니다. 그러나 변경 사항을 관찰하려면, 네트워크로부터 정보를 적극적으로 가져와야 합니다.

추상적인 설명 대신 예를 들어 보겠습니다.

:::note dApp 예시

예를 들어 전자책을 판매하는 스마트 컨트랙트가 있습니다. 사용자가 책을 구입하면 이메일을 통해 사본을 보내려고 합니다.

:::


dApp에는 오프체인 어딘가에 배포된 헬퍼가 있으며 이 헬퍼에는 전자책 사본과 함께 이메일을 보낼 수 있는 코드가 있습니다. 하지만 이 헬퍼를 어떻게 트리거할까요?

### 외부 세계로부터 블록체인으로 데이터 가져오기

NEAR 블록체인은 모든 사람이 블록체인과 상호 작용할 수 있도록 [JSON-RPC 엔드포인트](https://docs.near.org/api/rpc/introduction)을 구현합니다. JSON-RPC API를 통해 사용자는 주어진 매개변수로 스마트 컨트랙트를 호출할 수 있습니다. 또한 사용자는 블록체인의 데이터도 볼 수 있습니다.

따라서 예제를 계속 진행하면 헬퍼가 매초마다 [블록](https://docs.near.org/api/rpc/block-chunk#block)을 풀링한 다음, 모든 [청크](https://docs.near.org/api/rpc/block-chunk#chunk)를 풀링 하고 블록에 포함된 트랜잭션을 분석하여 "전자책 구매" 함수 호출이 있는 스마트 컨트랙트에 대한 트랜잭션이 있는지 확인할 수 있습니다. 이러한 트랜잭션을 관찰하면, 해당 트랜잭션이 성공적인지 확인하여 "전자책 구매" 트랜잭션이 실패한 사용자에게는 전자책을 보내지 않아야 합니다.

프로세스가 완료되면 헬퍼 코드를 트리거하여 사용자가 구매한 전자책이 포함된 이메일을 보낼 수 있습니다.

이 접근 방식은 데이터를 가져오는 *풀(pull) 모델* 입니다. 이 접근 방식에는 잘못된 것이 없지만, 때때로 가장 편안하거나 신뢰할 수 있는 접근 방식이 아니라는 것을 알 수 있습니다.

또한 모든 데이터가 JSON-RPC를 통해 제공되는 것은 아닙니다. 예를 들어 *로컬 Receipt* 는 NEAR 노드의 내부 데이터베이스에 저장되지 않기 때문에 JSON-RPC를 통해 사용할 수 없습니다.

### 인덱서

블록체인 인덱서는 데이터를 가져오는 *푸시(push) 모델* 을 구현한 것입니다. 소스에서 데이터를 적극적으로 가져오는 대신 헬퍼는 데이터가 헬퍼에 전송될 때까지 기다립니다. 전송된 데이터는 완전하므로, 헬퍼는 즉시 분석을 시작할 수 있습니다. 이상적으로는, 더 자세한 정보를 얻기 위해 추가 풀링을 피할 수 있을 만큼 전송된 데이터는 완전합니다.

우리의 예로 돌아가서 헬퍼는 **청크**, **트랜잭션 상태** 등과 함께 모든 **블록**을 수신 하는 **인덱서**가 됩니다. 같은 방식으로 헬퍼는 데이터를 분석하고 코드를 트리거하여 사용자에게 그들이 산 전자책이 포함된 이메일을 보냅니다.

![Indexer is streaming the data from the blockchain](/docs/intro/indexer.png)

:::info 인덱서 개념

인덱서는 *체인에 기록되는 데이터 스트림을* 수신 한 다음 즉시 필터링 및 처리하여 흥미로운 이벤트나 패턴을 감지할 수 있습니다.

:::


## 인덱서와 "넓은" 쿼리

이 문서의 시작 부분에서 *"넓은" 쿼리* 라는 용어 를 언급했습니다. 이에 대한 요약은 다음과 같습니다.

:::note "넓은" 쿼리 정의

여러 블록에서 데이터를 쿼리하려면 여러 단일 블록 쿼리에 대한 결과 집계가 필요합니다. 이러한 집계는 *"넓은" 쿼리* 에서 오는 것으로 간주할 수 있습니다.

:::

인덱서는 블록체인에서 *데이터 스트림을* 수신하고 정의된 요구 사항에 따라 데이터를 즉시 필터링하고 처리할 수 있으므로, "넓은" 쿼리 실행을 단순화하는 데 사용할 수 있습니다. 예를 들어, SQL과 같은 편리한 쿼리 언어를 사용하여 나중에 데이터를 분석하기 위해 데이터 스트림을 영구 데이터베이스에 기록할 수 있습니다.

"넓은 쿼리"가 필요한 또 다른 예시는 시드 구문을 사용하여 하나 이상의 계정을 복구하는 경우입니다. 시드 문구는 본질적으로 서명 키 쌍을 나타내므로, 복구는 연결된 공개 키를 공유하는 모든 계정에 대한 것입니다. 따라서 [NEAR 지갑](https://wallet.near.org)을 통해 계정을 복구하기 위해 시드 문구를 사용할 때, 일치하는 공개 키가 있는 모든 계정을 쿼리해서 복구해야 합니다. Utilizing [Near Lake Framework](https://github.com/near/near-lake-framework-rs) can be used to store this data in a permanent database and this allows [NEAR Wallet](https://wallet.near.org) to perform such "wide queries". 이 작업은 JSON-RPC만으로는 불가능합니다.

## 요약

이 문서가 인덱서 개념을 이해하는 데 도움이 되기를 바랍니다. 또한 이제 애플리케이션에 인덱서가 필요한지 여부를 쉽게 결정할 수 있기를 바랍니다.

## 다음은 무엇인가요?
We encourage you to learn more about the [Lake Indexer project](/build/data-infrastructure/lake-framework/near-lake). Please, proceed to [Tutorials](/build/data-infrastructure/lake-framework/near-lake-state-changes-indexer) section to learn how to build an indexer on practice.

대신, NEAR 생태계와 단단히 통합되어 있는 몇 개의 다른 써드 파티 인덱서들이 있습니다. [데이터 도구](/concepts/data-flow/data-storage#data-tools)에서 모든 데이터 소싱 옵션(The Graph, Pagoda, Pipespeak, 그리고 SubQuery 포함) 을 확인할 수 있습니다.
