---
layout: post
title: "IPFS - merkle-link"
categories:
  - IPFS_Review
tags:
  - IPFS_merkel_link
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS IPLD - merkle-link

merkle-link 는 두 객체를 연결하는 방법입니다. 대상 객체와 원본 객체는 암호화 된 해시의 내용을 사용하여 주소 지정됩니다. 동시에 대상 객체의 해시도 소스 객체에 포함됩니다. merkle-link 를 통한 콘텐츠 주소 지정은 다음을 수행 할 수 있습니다.

- 암호화 무결성 검사: Resolver 된 링크의 값은 해시로 테스트 할 수 있습니다. 이렇게하면 해시를 통해 링크되지 않은 데이터를 다른 사용자에게 줄 수 없기 때문에 git 또는 bittorrent 와 같이 광범위하고 안전한 신뢰할 수없는 데이터 교환이 가능합니다.
- 변경 불가능한 데이터 구조: merkle 링크가 있는 데이터 구조는 변경 될 수 없으며 분산 시스템의 중요한 속성입니다. 이는 분산 변수 상태 (예 : CRDT) 및 장기 아카이브를 의미하는 버전 제어에 유용합니다.

merkle-link 는 다음 IPLD 객체 모델로 표현됩니다: "link value" 포함/매핑, 예:

링크는 json 에서 "link object" 로 표현 될 수 있습니다.

```go
{ 
   "/" : "/ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k"
}
// "/"는 링크 키입니다.
// "/ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k" 링크 값입니다.
```

foo/baz 에 링크가 있는 객체: 

```go
{
  "foo": {
      "bar": "/ipfs/QmUmg7BZC1YP1ca66rRtWKxp77WgVHrnv263JtDuvs2k", // 링크가 아닙니다.
      "baz": {"/": "/ipfs/QmUmg7BZC11ca66rRtWKxpXp77WgVHrnv263JtDuvs2k"} // 링크
    }
}
```

다음 구조에서는 files/cat.jpg 에 또 다른 가상 "link object" 가 있으며 실제 링크는 files/cat.jpg/link 에 있습니다.

```go
{
  "files": {
    "cat.jpg": { // 링크 된 속성은 다른 객체에 포함됩니다.
      "link": {
      "/": "/ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k"}, // 링크
      "mode": 0755,
      "owner": "jbenet"
    }
  }
}
```

링크가 수정되어 링크 경로가 유효하지 않으면 맵 자체가 오브젝트로 바뀝니다.

이 링크는 멀티 허브 일 수 있습니다. 즉, 링크는 /ipfs 레벨 또는 객체의 절대 경로에 있다고 가정합니다. 그러나 현재 /ipfs 계층 경로만 사용할 수 있습니다.

응용 프로그램이 / 를 사용하여 다른 내용을 나타내야 할 경우 응용 프로그램 자체에서 해상도가 충돌하지 않도록해야 합니다.
