---
layout: post
title: "IPFS - Identities"
categories:
  - IPFS
tags:
  - IPFS
  - identity
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS 디자인 - Identities

IPFS는 다양한 기능을 담당하는 하위 프로토콜을 조합하여 구성된다.

- Identities
- Network
- Routing
- Exchange
- Discovery
- Objects
- Files
- Naming

IPFS 를 구성하는 많은 모듈들은 별도의 프로젝트로 진행되고 있다.

-   [IPFS - Content Addressed, Versioned, P2P File System (DRAFT 3)](https://github.com/ipfs/papers/raw/master/ipfs-cap2pfs/ipfs-p2p-file-system.pdf)
-   [github.com/ipfs/specs](https://github.com/ipfs/specs)
-   [github.com/libp2p/specs](https://github.com/libp2p/specs)
-   [github.com/ipld/specs](https://github.com/ipld/specs)

차분히 하나씩 알아보는 시간을 가지고자 한다.

# 1. Identities

IPFS 의 Node (peer) 는 분산되어 있기 때문에 각 Node 를 식별하기 위해 **NodeID** 를 발행한다. NodeID 는 공개키를 _Cryptographic Hash_ 함수를 사용하여 얻은 Digest 를 이용한다.

![공개키를 활용한 NodeID 생성](/images/ipfs_id01.png)

```go
type NodeId Multihash
type Multihash []byte
// self-describing cryptographic hash digest

type PublicKey []byte
type PrivateKey []byte
// self-describing keys

type Node struct {
  NodeId NodeID
  PubKey PublicKey
  PriKey PrivateKey
}

difficulty = << integer parameter >>;
n = Node{};
for {
  n.PubKey, n.PrivKey = PKI.genKeyPair()
  n.NodeId = hash(n.PubKey)
  p = count_preceding_zero_bits(hash(n.NodeId))

  if p < difficulty {
    break
  }
}
```

Node 간 연결은 서로의 공개 키를 교환하고 `hash(n.PubKey) == NodeID` 검사를 실시하여 _Fake_ 노드를 판단한다. (NodeID 값이 공개키 Hash 값과 같아야 함)

## 1.1 multihash (self-describing hash format)

자가 기술형 Hash 포멧 (self-describing hash format) 이라고 할 수 있다. Hash Digest 전에 자신의 사용하는 Hash 함수의 _'타입, 길이'_ 등 **메타 데이터** 정보를 포함하는 것이다.

```tex
<function_code><digest_length><digest_bytes>
```

자가 기술형 Hash 포멧의 장점은 Hash 함수가 다른 Node Hash 함수와 통신이 가능한 것이다. Hash 함수가 버전업 했을 경우에도 과거의 Hash 값을 다시 생성할 필요는 없다.

IPFS 에서는 multihash 라는 방식이 취해지고 있다.

[_github.com/multiformats/multihash_](https://github.com/multiformats/multihash)

```tex
<varint_hash_function_code><varint_digest_size_in_bytes><hash_function_output>
```

```tex
// Binary
  hash    digest                     hash
function   size                     digest
--------  --------  ------------------------------------
00010001  00000100  101101100 11111000 01011100 10110101
sha1      4 bytes   4 byte sha1 digest
```

![multihash 를 활용한 Node 간 통신](/images/ipfs_id02.png)

현재, IPFS 상에서는 아래의 Hash 함수 타입을 사용할 수 있다.

- sha2-256
- sha2-512
- sha3

`count_preceding_zero_bits` 함수가 반환하는 값이 일정한 값 이하가 아닌경우 생성되지 않는다.  
마치 BitCoin 마이닝의 Nonce 를 찾는 작업과 같다.
시빌 공격 (대량의 ID 를 생성해 네트워크를 빼앗는 공격) 에 대한 대응이라고 생각할 수 있다.

## 1.2 모듈

### 1.2.1 키생성

Node.js의 경우는 [_js-libp2p-crypto_](https://github.com/libp2p/js-libp2p-crypto) 가 제공되고 있다. 브라우저용으로 WebCrypt API에 대한 대응도 하고 있다.  

Go의 경우는 [_go-libp2p-crypto_](https://github.com/libp2p/go-libp2p-crypto) 가 제공되고 있다.

### 1.2.2 PeerID 생성

PeerID 를 생성하기 전에, 생성한 키의 회전을 높일 수 있는 연구가 필요하다.  
예를들어 RSA키를 생성할 경우 공개 키를 RSA DER 형식의 인코딩을 하고 키 정보를 저장하는 protobuf 에서 정의되는 구조체에 포함하여 protobuf 에서 시리얼라이즈한 것을 Base64 에 걸친 문자열로 구성하는 것이다.

![RSA, protobuf, Base64 를 활용한 PeerID 생성](/images/ipfs_id03.png)

Node.js의 경우 [_js-peer-id_](https://github.com/libp2p/js-peer-id) 가 생성되어 있다. 실제로는 [_js-peer-info_](https://github.com/libp2p/js-peer-info) 가 Peer 추상모델을 작성할 때 호출된다.  

Go의 경우는 [_go-libp2p-peer_](https://github.com/libp2p/go-libp2p-peer) 가 담당한다.

### 1.2.3 Peer 및 키 정보 보존

교환된 Peer 정보를 저장하기 위한 모듈도 구성되어 있다.

Node.js 의 경우는 [_js-peer-book_](https://github.com/libp2p/js-peer-book) 이 담당한다.  
`Key=PeerID/Value=PeerInfo` 라는 Map 을 On-Memory 에 유지하면서 액세서를 공개하고 있다.  

Go의 경우 [_go-libp2p-peerstore_](https://github.com/libp2p/go-libp2p-peerstore) 에서 Interface 를 제공한다.

Peer 정보와 Key 정보의 보존 외에 PeerMetadeta (Peer 마다 추가 정보) 를 보존할 수 있도록 되어 있다. 그 외,

- 저장정보 CRUD 스레드 세이프  
- Peer 마다 TTL 설정  
- Latency 감시

등의 기능을 가지고 있다.  
go-libp2p-peerstore 의 구현은 다음과 같다.

- [go-libp2p-peerstore/pstoremem](https://github.com/libp2p/go-libp2p-peerstore/pstoremem): On-Memory
- [go-libp2p-peerstore/pstoreds](https://github.com/libp2p/go-libp2p-peerstore/pstoreds): 아래와 같은 다양한 Database 추상 레이어 {[go-datastore](https://github.com/ipfs/go-datastore)} 대응
  - [go-ds-leveldb](https://github.com/ipfs/go-ds-leveldb)
  - [go-ds-bolt](https://github.com/ipfs/go-ds-bolt)
  - [go-ds-redis](https://github.com/ipfs/go-ds-redis)
  - [go-ds-s3](https://github.com/ipfs/go-ds-s3)
  - [go-ds-badger](https://github.com/ipfs/go-ds-badger)
  - [go-ds-flatfs](https://github.com/ipfs/go-ds-flatfs)
  - [go-ds-swift](https://github.com/ipfs/go-ds-swift)

외부 스토리지나 분산 스토리지에 Peer 정보를 보존할 수 있는 기능을 제공하고 있음을 알 수 있다.
