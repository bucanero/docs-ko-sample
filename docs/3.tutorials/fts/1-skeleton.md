---
id: skeleton
title: 뼈대 및 Rust 아키텍처
sidebar_label: 컨트랙트 아키텍처
---

import {Github} from "@site/src/components/codetabs"

In this article, you'll learn about the basic architecture behind the FT contract that you'll develop while following this _"Zero to Hero"_ series. You'll discover the contract's layout and you'll see how the Rust files are structured in order to build a feature-complete smart contract.

:::info New to Rust?
If you are new to Rust and want to dive into smart contract development, our [Quick-start guide](../../2.build/2.smart-contracts/quickstart.md) is a great place to start.
:::

---

## 소개

This tutorial presents the code skeleton for the FT smart contract and its file structure. You'll find how all the functions are laid out as well as the missing Rust code that needs to be filled in. Once every file and function has been covered, you'll go through the process of building the mock-up contract to confirm that your Rust toolchain works as expected.

---

## Files structure

The repository comes with many different folders. Each folder represents a different milestone of this tutorial starting with the skeleton folder and ending with the finished contract folder. If you step into any of these folders, you'll find that they each follow a regular [Rust](https://www.rust-lang.org/) project. The file structure for these smart contracts have:

- `Cargo.toml`: 코드 의존성(dependency)을 정의하기 위한 파일(Javascript나 node 프로젝트에서의 `package.json`과 유사)
- `src`: 모든 Rust 소스 파일이 저장되는 폴더입니다.
- `target` folder where the compiled `wasm` will output to.

<hr className="subsection" />

### 소스 파일

| 파일                         | 설명                                                                                                                           |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| [ft_core.rs](#corers)      | 대체 가능한 토큰(FT)의 액세스 및 전송을 제어하는 로직이 존재합니다. 이 파일은 [핵심](https://nomicon.io/Standards/Tokens/FungibleToken/Core) 표준의 구현을 나타냅니다. | |
| [lib.rs](#librs)           | 공개된 스마트 컨트랙트 함수들을 가지고 있고, 어떤 정보가 온체인에 저장될지 지시합니다.                                                                            |
| [metadata.rs](#metadatars) | 토큰 및 메타데이터 구조를 정의합니다. 이 파일은 표준의 [메타데이터](https://nomicon.io/Standards/Tokens/FungibleToken/Metadata) 확장 구현을 나타냅니다.            |
| [storage.rs](#storagers)   | 등록 및 저장을 위한 로직을 포함합니다.  이 파일은 [스토리지 관리](https://nomicon.io/Standards/StorageManagement) 표준의 구현을 나타냅니다.                       |

```
skeleton
├── Cargo.lock
├── Cargo.toml
└── src
    ├── ft_core.rs
    ├── lib.rs
    ├── metadata.rs
    └── storage.rs
```

:::tip
[GitHub 레퍼지토리](https://github.com/near-examples/ft-tutorial/tree/main/1.skeleton)에서 코드를 살펴보세요.
:::

---

## `ft_core.rs`

사용자 간에 FT를 전송하고 중요한 정보를 쿼리할 수 있는 핵심 로직입니다.

| 메서드                       | 설명                                                                                                                                                                                                                                                                           |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ft_transfer**           | 지정된 양의 FT를 수신자 ID로 전송합니다.                                                                                                                                                                                                                                                    |
| **ft_transfer_call**    | 지정된 양의 FT를 수신자에게 전송하고, 수신자의 컨트랙트에서 교차 컨트랙트 호출(cross-contract call)을 수행하여 `ft_on_transfer` 메서드를 실행하려고 시도합니다. 이 `ft_on_transfer` 메서드의 구현은 컨트랙트 작성자에게 달려 있습니다. 이 튜토리얼의 마켓플레이스 섹션에서 예제 구현을 볼 수 있습니다. `ft_on_transfer`의 실행이 끝나면, `ft_resolve_transfer`는 호출되어서 성공적으로 수행되었는지 확인됩니다. |
| **ft_total_supply**     | 컨트랙트에서 유통되는 대체 가능한 토큰의 총량을 반환합니다.                                                                                                                                                                                                                                            |
| **ft_balance_of**       | 특정 사용자가 소유한 대체 가능한 토큰 수를 반환합니다.                                                                                                                                                                                                                                              |
| **ft_on_transfer**      | 수신자의 컨트랙트에 따라 진행되는 메서드입니다. `ft_transfer_call` 메서드를 통해 FT가 수신자의 컨트랙트 계정으로 전송될 때 호출됩니다. 이는 보낸 사람에게 다시 환불해야 하는 FT 수를 반환합니다.                                                                                                                                                     |
| **ft_resolve_transfer** | `ft_on_transfer` 실행이 완료된 후 호출됩니다. 이 함수는 수신자 컨트랙트에서 사용하지 않은 FT를 환불하고 환불 후 수신자에게 전송된 순 FT 수를 반환합니다(있는 경우).                                                                                                                                                                     |

<Github language="rust" start="61" end="167" url="https://github.com/near-examples/ft-tutorial/blob/main/1.skeleton/src/ft_core.rs" />

You'll learn more about these functions in the [circulating supply](/tutorials/fts/circulating-supply) and [transfers](/tutorials/fts/transfers) sections of the tutorial series.

---

## `lib.rs`

이 파일은 컨트랙트가 저장하고 추적하는 정보를 간략하게 설명합니다.

| 메서드                    | 설명                                                                                                                              |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **new_default_meta** | 기본 `metadata`를 이용해 컨트랙트를 초기화해, 사용자가 입력값을 제공하지 않아도 되도록 합니다. In addition, a total supply is passed in which is sent to the owner. |
| **new**                | 사용자가 제공한 `metadata`와 총 공급량으로 컨트랙트를 초기화합니다.                                                                                      |

:::info Keep in mind
The initialization functions (`new`, `new_default_meta`) can only be called once.
:::

<Github language="rust" start="36" end="60" url="https://github.com/near-examples/ft-tutorial/blob/main/1.skeleton/src/lib.rs" />

튜토리얼 시리즈의 [토큰 정의](2-define-a-token.md)에서 이러한 기능에 대해 자세히 알아볼 것입니다.

---

## `metadata.rs`

이 파일은 토큰 및 메타데이터에 대해 저장할 정보를 추적하는 데 사용됩니다. 또한 표준 메타데이터 확장의 일부인 컨트랙트의 [메타데이터](https://nomicon.io/Standards/Tokens/FungibleToken/Metadata)를 보는 기능을 정의할 수 있습니다.

| 이름                        | 설명                                                            |
| ------------------------- | ------------------------------------------------------------- |
| **FungibleTokenMetadata** | 이 구조는 대체 가능한 토큰의 메타데이터를 정의합니다.                                |
| **ft_metadata**           | This function allows users to query for the token's metadata. |

<Github language="rust" start="11" end="30" url="https://github.com/near-examples/ft-tutorial/blob/main/1.skeleton/src/metadata.rs" />

튜토리얼 시리즈의 [토큰 정의](2-define-a-token.md)에서 이러한 기능에 대해 자세히 알아볼 것입니다.

---

## `storage.rs`

[스토리지 관리](https://nomicon.io/Standards/StorageManagement) 표준에 따른 등록 로직을 포함합니다.

| 메서드                          | 설명                                                                                                                       |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **storage_deposit**          | 주어진 계정에 대해 Ⓝ의 입금을 받는 지급 메서드입니다. 이렇게 하면 컨트랙트 사용자가 등록됩니다.                                                                  |
| **storage_balance_bounds** | 컨트랙트와 상호 작용하는 데 필요한 최소 및 최대 허용 스토리지 금액을 반환합니다. FT 컨트랙트의 경우 최소값 = 최대값입니다.                                                 |
| **storage_balance_of**     | 지정된 사용자가 지불한 총 스토리지 및 사용 가능한 스토리지를 반환합니다. FT 컨트랙트의 경우 등록을 위해 컨트랙트에서 사용하고 스토리지에 대해 초과 지불할 수 없기 때문에, 사용 가능한 스토리지는 항상 0입니다. |

<Github language="rust" start="81" end="108" url="https://github.com/near-examples/ft-tutorial/blob/main/1.skeleton/src/storage.rs" />

:::tip
튜토리얼 시리즈의 [스토리지](4.storage.md)에서 이러한 기능에 대해 자세히 알아볼 것입니다.
:::

## 뼈대 만들기

아직 기본 레퍼지토리를 복제하지 않은 경우 터미널을 열고 다음을 실행합니다.

```sh
git clone https://github.com/near-examples/ft-tutorial/
```

다음으로 `1.skeleton/build.sh` 파일 내에 있는 빌드 스크립트를 사용하여 뼈대 컨트랙트를 빌드합니다.

```sh
cd ft-tutorial/1.skeleton
cargo near build
```

이 소스는 뼈대일 뿐이므로 다음과 같이 사용하지 않는 코드에 대한 많은 경고를 받게 됩니다.

```
   = note: `#[warn(dead_code)]` on by default

warning: constant is never used: `GAS_FOR_RESOLVE_TRANSFER`
 --> src/ft_core.rs:5:1
  |
5 | const GAS_FOR_RESOLVE_TRANSFER: Gas = Gas(5_000_000_000_000);
  | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

warning: constant is never used: `GAS_FOR_FT_TRANSFER_CALL`
 --> src/ft_core.rs:6:1
  |
6 | const GAS_FOR_FT_TRANSFER_CALL: Gas = Gas(25_000_000_000_000 + GAS_FOR_RESOLVE_TRANSFER.0);
  | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

warning: `fungible-token` (lib) generated 25 warnings
    Finished release [optimized] target(s) in 1.93s
✨  Done in 2.03s.
```

이러한 경고에 대해 걱정하지 마세요. 아직 이 컨트랙트를 배포하지 않을 것입니다. 뼈대 구축하는 것은 Rust 툴체인이 제대로 작동하는지 확인하는 것이고, 이는 다음 튜토리얼에서 이 NFT 컨트랙트의 개선된 버전을 컴파일할 수 있는지 확인하는 데 유용합니다.

---

## 결론

이 NFT 스마트 컨트랙트의 레이아웃과 다양한 소스 파일에 모든 함수가 어떻게 배치되어 있는지 확인했습니다. `yarn`을 사용하여 컨트랙트를 컴파일할 수 있었고, 튜토리얼의 다음 [섹션](/tutorials/fts/circulating-supply)에서 이 뼈대를 구체화하기 시작할 것입니다.

---

:::note 문서의 버전 관리
이 글을 쓰는 시점에서 이 예제는 다음 버전에서 작동합니다.

- rustc: `1.77.1`
- near-sdk-rs: `5.1.0` (with enabled `legacy` feature)
- cargo-near: `0.6.1`
:::
