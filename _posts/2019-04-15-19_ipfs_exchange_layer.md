---
layout: post
title: "IPFS - exchange"
categories:
  - IPFS_Review
tags:
  - IPFS_exchange
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS 기본 - Exchange 계층

IPFS 의 스위칭 레이어는 **BitSwap 프로토콜**을 통해 피어 노드간에 데이터를 교환합니다. BitTorrent 와 마찬가지로
각 피어 노드는 다운로드하는 동안 다운로드 된 데이터를 다른 피어 노드에 업로드합니다. BitTorrent 프로토콜과 달리 BitSwap 은 시드 파일의 데이터블록에만 국한되지 않습니다.

전체 교환 계층에서 IPFS 의 모든 노드가 마켓을 이루고 있으며 데이터 교환은 Filecoin 이 교환 계층에서 인센티브 계층을 추가합니다. 이 마켓에서 Bitswap 은 두 가지 주요 작업을 수행합니다. 즉, 데이터 블록을 제공하는 피어에 이득 (Filecoin 에서는 코인) 을 주기위한 마켓입니다.

- 네트워크에서 클라이언트 요청 블록을 가져 오기위한 시도
- 소유 블록을 다른 노드로 보냅니다

## BitSwap 프로토콜

BitSwap 은 IPFS 노드의 블록 교환 정보를 기록합니다. 소스 코드 구조는 다음과 같습니다:

``` go
// Additional state kept
type BitSwap struct {
    ledgers map[NodeId]Ledger
    // Ledgers known to this node, inc inactive
    // 노드 명세
    active map[NodeId]Peer
    // currently open connections to other nodes
    // 현재 연결된 피어
    need_list []Multihash
    // checksums of blocks this node needs
    // 노드가 필요한 블록 데이터 검사 목록
    have_list []Multihash
    // checksums of blocks this node has
    // 노드가 가지고 있는 블록 데이터 검사 목록
}
```

노드가 다른 노드로부터 데이터 블록을 요청하거나 다른 노드에 데이터 블록을 제공 할 필요가 있을 때 다음과 같이 _BitSwap_ 메시지를 보내고 이 메시지는 주로 두 부분으로 구성됩니다. 원하는 데이터 블록 목록 (want_list)과 해당 데이터 블록입니다. 전체 메시지는 **Protobuf** 를 사용하여 인코딩 됩니다:

```protobuf
message Message {
  message Wantlist {
    message Entry {
      optional string block = 1; // the block key
      optional int32 priority = 2; // the priority (normalized). default to 1
      // 우선 순위 설정, 기본값 = 1
      optional bool cancel = 3;  // whether this revokes an entry
      // 취소
    }

    repeated Entry entries = 1; // a list of wantlist entries
    optional bool full = 2;     // whether this is the full wantlist. default to false
  }

  optional Wantlist wantlist = 1;
  repeated bytes blocks = 2;
}
```

BitSwap 시스템은 매우 중요한 두 가지 모듈이 있습니다. 요구 사항 관리자 (**Want-Manager**) 와 의사 결정 엔진 (**Decision-Engine**) 이 있습니다. Want-Manager 는 노드가 데이터를 요청할 때 로컬로 해당 결과를 리턴하거나 적합한 요청을 발행하고 후자 (Decision-Engine) 는 다른 노드에 자원을 할당하는 방법을 결정합니다. 노드가 _want_list_ 를 포함하는 메시지를 수신하면 메시지는 의사 결정 엔진으로 전달되고 엔진은 노드의 BitSwap 명세서는 요청을 처리하는 방법을 결정합니다. 전체 프로세스는 다음과 같습니다.

![](https://raw.githubusercontent.com/ipfs/js-ipfs-bitswap/HEAD/img/architecture.png)

위의 프로토콜 흐름도를 통해 BitSwap 데이터 교환의 전체 과정과 peer-to-peer 연결의 수명주기를 볼 수 있습니다. 이 라이프 사이클에서 피어 노드는 일반적으로 네 가지 상태를 거칩니다:

- Open: 전송할 BitSwap 명세 상태는 연결이 설정 될 때까지 피어 노드간에 열립니다.
- Sending: want_list 및 데이터 블록은 피어 노드간에 전송됩니다.
- Close: 피어 노드는 데이터를 보낸 후에 연결을 끊습니다.
- Ignored: 피어 노드는 타임 아웃, 자동 및 낮은 크레딧과 같은 요인으로 인해 무시됩니다.

피어 노드의 소스 구조를 결합하여 IPFS 노드가 서로를 찾는 방식을 분석합니다:

``` go
type Peer struct {
    nodeid NodeId
    ledger Ledger
    // Ledger between the node and this peer
    // 노드와 피어 노드 간의 청구 명세서
    last_seen Timestamp
    // timestamp of last received message
    // 마지막으로받은 메시지의 타임 스탬프
    want_list []Multihash
    // checksums of all blocks wanted by peer
    // includes blocks wanted by peer's peers
    // 모든 블록 검사 필요
}

// Protocol interface:
interface Peer {
    open (nodeid :NodeId, ledger :Ledger);
    send_want_list (want_list :WantList);
    send_block (block :Block) -> (complete :Bool);
    close (final :Bool);
}
```
###  :: Peer.open(NodeID,Ledger)

노드가 연결을 설정하면 보낸 노드는 _BitSwap_ 명세서를 초기화 합니다. 피어에 대한 명세서를 보관할 수 있습니다. 새 명세서를 새로 만들 수도 있습니다. 이는 노드 청구 일관성 문제에 따라 다릅니다. 송신 노드는 수신 노드에 통지하기 위해 청구서를 담고있는 공개 메시지를 전송할 것입니다. 수신 노드가 _Open_ 메시지를 수신 한 후 이 연결 요청을 수락할지 여부를 선택할 수 있습니다.

수신자는 송신자의 로컬 명세 데이터를 체크하는데 신뢰할 수 없는 노드를 판명합니다. 즉 전송 타임 아웃, 매우 낮은 신용 점수 및 큰 부채 비율에 기초하여 신뢰할 수 없는 노드인 것으로 판명되면 수신기는 ignore_cooldown 을 통해 이 요청을 무시하여 연결을 끊습니다. 이 목적은 부정 행위를 방지하는 것입니다.

연결이 성공하면 수신자는 로컬 명세서를 사용하여 피어 개체를 초기화하고 last_seen 타임 스탬프를 업데이트 한 다음 수신 된 명세서를 자체 명세서와 비교합니다.

두 명세서가 정확히 같으면 연결이 Open 하고 명세가 정확히 동일하지 않은 경우 노드는 새로운 명세서를 생성하고 이 명세서에 동기화를 전송합니다. 이는 앞에서 언급한 송신자 노드와 수신자 노드의 명세 일관성 문제를 보장합니다.

### :: Peer.send_want_list(WantList)

연결이 이미 Open 상태에 있으면 송신자 노드는 want_list 를 연결된 모든 수신 노드에 브로드캐스트합니다. 동시에 want_list 를 수신 한 후 수신 노드는 수신자가 원하는 데이터 블록이 있는지 여부를 확인한 다음 _BitSwap_ 정책을 사용하여 데이터 블록을 보내고 전송합니다.

### :: Peer.send_block(Block)

블록을 보내는 방법 로직은 매우 간단합니다. 기본적으로 송신 노드는 데이터 블록만 전송합니다. 모든 데이터를 수신 한 후 수신 노드는 Multihash 를 계산하여 예상되는 것과 일치하는지 확인한 다음 수신 확인을 반환합니다. 블록 전송을 완료 한 후, 수신 노드는 블록 정보를 need_list 에서 have_list 로 이동시키고 수신 노드와 송신 노드 모두 명세서 목록을 동시에 갱신한다. 전송 확인이 실패하면 전송 노드가 오작동하거나 의도적으로 수신 노드의 행동을 공격 할 수 있으며 수신 노드는 추가 거래를 거부 할 수 있습니다.

### :: Peer.close(Bool)

peer-to-peer 연결은 두 가지 경우에 닫아야 합니다.

- **silent_want** (시간초과): 상대방으로 부터 메시지를 받지 못했을 때 노드는 `Peer.close (false)` 를 발행
- **_BitSwap_ 종료** (노드 종료): 노드는 `Peer.close (true) ` 발행

P2P 네트워크의 경우 '모든 사람들이 자신의 데이터를 공유하도록 동기를 부여하는 방법' 및 'P2P 소프트웨어의 자체 데이터 공유 전략을 이용하는 방법' 이 있습니다. IPFS 에서도 마찬가지입니다. 그 중 _BitSwap_ 전략 시스템은 '신용', '전략' 및 '명세서' 의 세 부분으로 구성됩니다.

## BitSwap 신뢰 시스템

_BitSwap_ 프로토콜은 노드가 데이터를 공유하도록 동기를 부여 할 수 있어야 합니다. IPFS 는 노드 간의 데이터 전송 및 수신을 기반으로 신용 시스템을 구축합니다.

- 다른 노드로 데이터를 보내면 신용도가 증가합니다.
- 다른 노드로부터 데이터를 수신하면 신용도가 낮아집니다.

노드가 데이터만 수신하고 데이터를 업로드하지 않으면 신용도는 다른 노드에 의해 낮추어 집니다. 이것은 사이버 공격을 효과적으로 방지 할 수 있습니다.

## BitSwap 정책

_BitSwap_ 신용 시스템으로 서로 다른 정책으로 구현할 수 있습니다. 각 정책은 시스템의 전반적인 성능에 다른 영향을줍니다. 정책의 목표는 다음과 같습니다:

- 노드 데이터 교환의 전반적인 성능과 효율성이 가장 높습니다.
- 데이터를 다운로드만 하고 업로드하지 않는 현상을 방지합니다.
- 일부 공격을 효과적으로 방지합니다.
- 신뢰할 수 있는 노드에 대해 느슨한 메커니즘을 설정합니다.

IPFS 는 whitepaper 에서 몇 가지 참조 정책 메커니즘을 제공합니다. 각 노드는 다른 노드가 주고받는 데이터를 기반으로 신용 및 부채 비율 (_rb_) 을 계산합니다: `r = bytes_sent / bytes_recv + 1`

데이터 전송 속도 (_P_):
`P (send | r) = 1 - (1 / (1 + exp (6-3r)))`

위 합의에 따르면 r 이 2 보다 크면 전송률 _P (send | r)_ 가 작아 지므로 다른 노드는 데이터를 계속 보내지 않습니다.

## BitSwap 명세

_BitSwap_ 노드는 다른 노드와 통신하는 명세서 (데이터 송수신 레코드) 를 기록합니다. 명세 데이터 구조는 다음과 같습니다.

```go
type Ledger struct {
    owner      NodeId
    partner    NodeId
    bytes_sent int
    bytes_recv int
    timestamp  Timestamp
}
```

이를 통해 노드는 변조를 피하기 위해 히스토리를 추적 할 수 있습니다. 두 노드간에 연결이 이루어지면 _BitSwap_ 은 서로 결제 정보를 교환하고 원장이 일치하지 않으면 원장은 지워지고 다시 예약되며 악의적 노드는 이 원장을 잃어 버리고 채무를 청산 할 것으로 예상됩니다. 다른 노드는 이것을 기록 할 것이며 파트너 노드는 위법 행위로 취급하여 거래를 거부 할 수 있습니다.
