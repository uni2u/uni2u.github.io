---
layout: post
title: "IPFS - identity"
categories:
  - IPFS_Review
tags:
  - IPFS_identity
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS 기본 - ID 계층

각 IPFS 노드에는 고유 한 ID 가 있으며 ID 는 NodeID 로 표시됩니다. IPFS 네트워크에 가입하기 전에 노드는 먼저 자신의 ID 를 생성해야 합니다. 공개 키는 S/Kademlia 정적 암호화 알고리즘에 의해 생성되며 해시 연산에 의해 NodeID 를 생성 합니다. C ++ 에서 NodeID 의 생성 과정은 다음과 같습니다:

``` c++
// 난이도 (leading factor)를 0으로 설정
difficulty =< integer parameter >
// 노드 초기화
n= Node{}
// 루프 연산, 조건부 해시 (NodeID) 연산의 값에 대 한 파일럿 0 수가 ≥ 0 자리로 설정 될 때까지
do{
  // 공개-개인 키 쌍 생성
  n.PubKey, n.PrivKey = PKI.genKeyPair() 
  // 검증되지 않은 NodeID 를 얻기위해 공개 키에서 해시 작업을 수행
  n.NodeId = hash(n.PubKey)
  // NodeID 의 적법성 확인, 검증 후 다음 루프 종료
  p = count_preceding_zero_bits(hash(n.NodeId))
} while(p<difficulty)
```

노드는 공개 키의 해시 연산 값이 요구 사항을 충족 할 때까지 계속해서 공개-개인 키 쌍을 생성합니다. 최종 공개 키 해시 작업의 값은 IPFS 노드와 같은 시스템의 NodeID 입니다.

터미널에서 `ipfs id` 를 실행하면 다음과 같이 시스템의 IPFS 노드 정보가 표시됩니다.

```
$ ipfs id
{
  "ID": "QmXdSpUBx9Ut6q8LF8Wyt1Wi2wxmob6qnnT11uV3SmvUP3",
  "PublicKey": "CAASpgIwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCh7fHOV1X0LEsqxxlY+wRRuMGZ+E7sMAWXfMj8NLenv3KpIX8pHy0lk/H6VCCjKB+t4e2Rb+Px9Uwh2YjRlLaaMkBYN27COVBdtL9sH5ZW2BZ6x00Deg+gWoO0xs5rzadtk45vV44RzxYDuGCT0GW79WgTBUi0s1O00LE6p+wOrZdk6UGYjUEzL3nD6CRMPoQbVtV8WGBWFksoyM+bnlyyCcFhw0suN5Yf8OicwGIDznsjniSOrku9QpoFU1B96SKMmfqiXZC+KyzFfKp/U1lFmfp7wlYObUYpMkcWI0/bRzdursQGga7xXyKwQJ5ZGouSANt6cClTMnOUHGLCFzzNAgMBAAE=",
  "Addresses": null,
  "AgentVersion": "go-ipfs/0.4.18/",
  "ProtocolVersion": "ipfs/0.1.0"
}
```

사용자는 시스템의 IPFS 노드 정보 파일을 삭제 한 다음 초기화 (명령: `ipfs init`) 하여 새 ID 를 재설정 할 수 있습니다. 하지만 이렇게하면 이전 ID 의 모든 네트워크 혜택을 잃게됩니다.

IPFS 네트워크의 노드가 처음으로 연결될 때 자신의 공개 키를 서로 교환 한 다음 상대방의 공개 키, 즉 해시 (`other.PubKey`)를 해시하고 얻은 값이 상대방의 NodeID 와 같으면 합법적 노드로 인식하고 연결됩니다.  (값이 다르면 'fake' 노드로 인식)

여기에서 해시 함수에 의해 얻어진 값은 QmXdSpubx9Ut6q8LF8Wyt1Wi2wxmob6qnnT11uV3SmvUP3 과 같은 값이며 이 값은 Base58 인코딩 후의 값입니다. (모든 해시가 "Qm" 으로 시작)
이것은 실제로 multihash 이기 때문에 처음 2 바이트는 해시 함수와 해시 길이를 지정하는 데 사용됩니다. 16 진수의 처음 두 바이트는 1220이며 12는 SHA256 해시 함수임을 나타내고 20은 32 바이트 길이 계산을 선택하는 해시 함수를 나타냅니다.
