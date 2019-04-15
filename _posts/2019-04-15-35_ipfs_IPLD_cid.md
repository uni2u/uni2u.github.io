---
layout: post
title: "IPFS - ipld-cid"
categories:
  - IPFS_Review
tags:
  - IPFS_ipld_cid
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS IPLD - IPLD-CID

## 왜 CID가 필요한가?

CID 는 IPFS 분산 파일 시스템에서와 유사한 표준 파일 주소 지정 형식입니다. 콘텐츠 주소 지정, 암호화 해싱 알고리즘 및 자체 설명 형식을 결합합니다. IPFS 및 IPLD 의 중요한 내부 식별자 입니다.

![](https://dominicsteil.files.wordpress.com/2017/02/2017-02-21-06_09_27-juan-benet_-enter-the-merkle-forest-youtube.png?w=648)

CID 관련 토론은 다음을 참조하십시오: https://github.com/ipfs/specs/issues/130 (첫 번째 게시물은 여기를 참조하십시오)

## 프로토콜 설명

CID 는 콘텐츠 주소 지정을위한 자가 기술 식별자 입니다. 콘텐츠의 주소를 가져 오기 위해서는 암호화 해시 함수를 사용해야합니다. 많은 multiformats 를 사용하여 유연한 자체 기술을 구현합니다. 즉 해쉬 값을 얻기 위해 multihash 를 사용하고 콘텐츠 유형을 기술하기 위해 multicodec-packed 으로 포장하고, CID 자체를 문자열로 인코딩하는 multibase 를 사용합니다.

![](https://image.slidesharecdn.com/20181120-distributedweb-181120212721/95/2018-11-20-distributed-web-23-638.jpg?cb=1542749305)

현재 버전: CIDv1

CIDv1 은 네 부분으로 구성됩니다.

```go
<cidv1> ::= <mb><version><mcp><mh>
# or, expanded:
<cidv1> ::= <multibase-prefix><cid-version><multicodec-packed-content-type><multihash-content-address>
```

- <_multibase-prefix_> 멀티베이스 인코딩 (1 ~ 2 바이트) 입니다. CID를 다른 형식으로 인코딩하는 것이 편리합니다.
- <_cid-version_> 향후 업그레이드를 용이하게 하기 위해 CID 버전을 나타내는 변수입니다.
- <_multicodec-packed-content-type_> 콘텐츠 또는 데이터의 유형을 나타 내기 위해 다중 코드로 인코딩 된 형식을 사용하는 형식입니다.
- <_multihash-content-address_> 다중 해시 값은 콘텐츠의 암호화 해시 해시를 나타냅니다. Multihash 는 CID 가 향후 업그레이드 및 수정을 위해 다른 암호화 해시 기능을 사용할 수있게 합니다.

## 디자인 컨셉

CID 는 IPFS 를 구축 할 때 여러 가지 장단점을 염두하여 고안되었습니다. 이것은 여러 형식을 지원하는 많은 프로젝트와 관련이 있습니다.

_압축성_: CID 바이너리 기능으로 압축하는 것이 매우 효율적이며 CID 가 URL 의 일부가 될 수도 있습니다.
_운송 편의성_: "copy-pastability". CID 는 멀티베이스 인코딩으로 쉽게 전송됩니다. 예) base58 btc 로 인코딩 된 CID 의 길이는 더 짧아지고 해시 값은 쉽게 복사되고 붙여 넣어집니다.
_가변성_: CID 는 모든 형식의 결과, 모든 해시 함수를 나타낼 수 있습니다.
_콘텐츠 잠금 방지_: CID 는 이전 콘텐츠에서 보호해야합니다.
_업그레이드 가능성_: CID 의 인코딩 된 버전을 업그레이드 할 수 있어야 합니다.

## 읽을 수 있는 CID 값

더 나은 디버깅과 해석을 위해서는 의미 있고 읽기 쉬운 CID 의 내용이 필요합니다. 일반 CID 는 다음과 같이 "사용자가 읽을 수 있는 CID" 로 변환 할 수 있습니다.

`<hr-cid> ::= <hr-mbc> "-" <hr-cid-version> "-" <hr-mcp> "-" <hr-mh>`

각 부분은 독자적인 읽을 수있는 내용을 나타냅니다.

- <_hr-mbc_> 는 사용자가 읽을 수있는 multibase 인코딩 (예: base58btc)
- <_hr-cid-version_> 은 cidv# 의 버전 (예: cidv1 또는 cidv2)
- <_hr-mcp_> 는 사용자가 읽을 수있는 multicodec-packed 된 인코딩 (예: cbor)
- <_hr-mh_> 은 사용자가 읽을 수 있는 multihash (예: sha2-256-256-abcdef0123456789 ...)

예를들어

```go
# TODO example
# example CID
# corresponding human readable CID
```

## 버전

### CIDv0

CIDv0 은 이전 버전과 호환되는 버전입니다.

- multibase 는 항상 base58btc
- multicodec 는 항상 protobuf-mdag
- cid-version 은 항상 cidv0
- multihash 는 다음과 같이 표현됩니다. cidv0 :: = <_multihash-content-address_>

### CIDv1

참조: https://github.com/ipld/cid#how-does-it-work-protocol-description

```go
<cidv1> ::= <multibase-prefix><cid-version><multicodec-packed-content-type><multihash-content-address>
```

## 기존 구현

- go-cid
- java-cid
- js-cid
- rust-cid
- py-cid
