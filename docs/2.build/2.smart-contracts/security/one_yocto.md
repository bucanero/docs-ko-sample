---
id: one-yocto
title: 사용자가 맞는지 확인 (1yⓃ)
---

NEAR uses a system of [Access Keys](../../../1.concepts/protocol/access-keys.md) to simplify handling accounts.
`전체 액세스 키`는 계정에 대한 완전한 제어권(즉, 모든 [Action](../anatomy/actions.md)을 수행할 수 있음)을 가지고 있고, `함수 호출 키`는 Ⓝ를 보증금으로 첨부하지 _않는_ 지정된 스마트 컨트랙트 내 메서드를 호출할 수 있는 권한만 있는 키입니다.

사용자가 컨트랙트와 상호작용하기 위해 [웹사이트에 로그인](../../integrate/frontend.md#user-sign-in)하면, 실제로 발생하는 일은 `함수 호출` 키가 생성되어 웹사이트에 저장되는 것입니다. 웹 사이트는 `함수 호출` 키에 액세스할 수 있으므로, 이를 통해 원하는 대로 승인된 메서드를 호출하는 데 사용할 수 있습니다. 이것은 대부분의 경우 사용자에게 매우 친숙하지만, [NFT](../../5.primitives/nft.md) 또는 [FT](../../5.primitives/ft.md)와 같은 귀중한 자산의 전송과 관련된 시나리오에서는 주의하는 것이 중요합니다. 이러한 경우 자산 이전을 요청하는 사람이 **실제로 사용자인지** 확인해야 합니다 .

사용자가 호출한 사람인지 확인하는 직접적이고 저렴한 방법 중 하나는, `1 yⓃ`를 첨부하도록 요구하는 것입니다. 이 경우, 사용자는 지갑으로 리디렉션되고 트랜잭션을 수락하라는 메시지가 표시됩니다. 그 이유는, 다시 한 번 이야기하지만 NEAR를 보내는 작업은 `전체 액세스` 키를 가진 사람만이 할 수 있기 때문입니다.
`전체 액세스` 키는 사용자의 지갑에만 있기 때문에, `1 yⓃ`를 포함한 트랜잭션은 사용자에 의해 이루어졌다고 신뢰할 수 있습니다.
