---
layout: post
title: "IPFS - multicodec"
categories:
  - IPFS_Review
tags:
  - IPFS_multicodec
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS Multiformats - multicodec


multicodec 은 자가 기술 형식을 포함하는 자가 기술 코덱입니다. multicodec 식별자는 varint 입니다. 실제로 base58 btc 인코딩의 경우 z, protobuf 의 경우 0x50 등과 같이 데이터 내용의 형식을 결정하기 위한 1-2 바이트가 있는 테이블입니다.

이는 https://github.com/multiformats/multicodec/blob/master/table.csv 에서 볼 수 있습니다.

multicodec 으로 식별되는 많은 양의 데이터는 다음과 같습니다.

```tex
<multicodec><encoded-data> 

＃ 생략할 수 있음：
<mc><data>
```

또 다른 유용한 시나리오는 multicodec 을 키 액세스의 일부로 사용하는 것입니다. 예를 들면 다음과 같습니다.

```tex
# suppose we have a value and a key to retrieve it
"<key>" -> <value>

# we can use multicodec with the key to know what codec the value is in
"<mc><key>" -> <value>
```

multicodec 은 multihash 및 multiaddr 과 잘 작동하며 여러 코드를 사용하여 이 값 앞에 접두어를 붙이면 자신이 무엇인지 알 수 있습니다.
