---
layout: post
title: "IPFS - merkle-path"
categories:
  - IPFS_Review
tags:
  - IPFS_merkel_path
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS IPLD - merkle-path

## 1. merkle-path 란 무엇입니까?

merkle-path 는 유닉스 스타일 경로 (예: /a/b/c/d) 를 사용하여 merkle-link 를 통해 모든 객체를 가져옵니다.

공통 파일 시스템은 IPFS 위에 객체 모델로 설계되어 데이터 객체 작업 및 쿼리를 구현하는 특정 알고리즘을 설계 할 수 있습니다.

## 2. merkle-path 의 작동 원리는 무엇입니까?

merkle-path 는 유닉스 스타일의 경로로 경로를 점진적으로 내용을 파싱합니다. 내용을 파싱하는 것은 merkle-link 의 내용을 가져 와서 더 파싱하는 것을 의미합니다.

예) 다음과 같은 merkle-path 가 있다고 가정합니다.

/ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k/a/b/c/d

의미:

- ipfs 는 컴퓨터가 무엇을 해야할지 구분할 수 있게 해주는 프로토콜 네임 스페이스 입니다.
- QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k 는 암호화 해시 값입니다.
- a/b/c/d 는 Unix 경로입니다.

/로 표현되는 통과 가능한 경로는 두 종류의 링크를 나타낼 수 있습니다.

- 동일한 대상 내부에서 데이터를 쿼리합니다.
- 객체 간의 정보 탐색은 merkle-link 에 따라 구현됩니다.

예를 들어 다음 데이터 집합이 있다고 가정합니다.

```protobuf
$ ipfs object cat --fmt=yaml QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k
---
a:
  b:
    link:
      /: QmV76pUdAAukxEHt9Wp2xwyTpiCmzJCvjnMxyQBreaUeKT
    c: "d"
    foo:
      /: QmQmkZPNPoRkPd7wj2xUJe5v5DsY6MX33MFaGhZKB2pRSE

$ ipfs object cat --fmt=yaml QmV76pUdAAukxEHt9Wp2xwyTpiCmzJCvjnMxyQBreaUeKT
---
c: "e"
d:
  e: "f"
foo:
  name: "second foo"

$ ipfs object cat --fmt=yaml QmQmkZPNPoRkPd7wj2xUJe5v5DsY6MX33MFaGhZKB2pRSE
---
name: "third foo"
```

다음과 같은 path 가 있다고 가정:

- /ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k/a/b/c
  - 첫 번째 객체만 탐색하고 문자열 d 를 가져옵니다.
- /ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k/a/b/link/c
  - 두 객체를 두루 거치고 문자열 e 를 얻습니다.
- /ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k/a/b/link/d/e
  - 두 객체를 두루 거치고 문자열 f 를 얻습니다.
- /ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k/a/b/link/foo/name
  - 첫 번째, 두 번째 대상을 두루 거쳐 문자열 second foo 를 얻게 됩니다.
- /ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k/a/b/foo/name
  - 첫 번째, 두 번째 대상을 두루 거쳐 문자열 third foo 를 얻게 됩니다.
