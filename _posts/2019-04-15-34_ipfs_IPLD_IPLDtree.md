---
layout: post
title: "IPFS - ipld-tree"
categories:
  - IPFS_Review
tags:
  - IPFS_ipld_tree
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS IPLD - IPLD-Tree

## 1. IPLD 데이터 모델이란 무엇입니까?

IPLD 데이터 모델은 모든 merkle-dag 에 대해 간단한 JSON 기반 구조를 정의합니다. 또한 일련의 인코딩 된 형식 구조를 정의합니다.

## 2. 제한 및 비전

다음 제한 사항이 있습니다.

- IPLD 경로는 모호하지 않아야하며 주어진 경로가 통과하는 방식은 일정해야합니다. (예: 링크 이름 충돌을 피하십시오)
- IPLD 경로는 전역이어야 하며 다른 언어도 지원해야 합니다. (예: ASCII 대신 UTF-8 사용)
- IPLD 경로는 UNIX 및 웹 위의 레벨에 있어야 합니다. (/ 를 사용하면 ASCII 시스템 내의 전환이 결정됩니다)
- 많은 시스템이 JSON 인터페이스를 지원합니다. IPLD에는 JSON 형식을 지원하는 가져 오기 및 내보내기 기능이 있어야합니다.
- JSON 데이터 모델도 간단하고 사용하기 쉽습니다. IPLD 또한 사용하기 쉬워야합니다.
- 이를 통해 데이터를 쉽게 정의 할 수 있습니다. IPLD 위에 새로운 데이터 구조를 정의 할 때 많은 배경 지식이 필요하지 않습니다.
- IPLD는 JSON 데이터 모델을 기반으로 하므로 JSON-LD 를 통해 RDF 및 링크 된 데이터 표준과 호환되어야 합니다.
- IPLD 직렬화 형식 (디스크, 전송 중)은 빠르고 공간 효율적이어야합니다. (JSON 형식으로 저장할 수 없지만 CBOR 또는 다른 형식이어야 함)
- IPLD 암호화 해시 해시는 업그레이드 가능해야합니다. (멀티 해시 사용)

다음과 같은 특징이 있습니다.

- IPLD 는 잘못된 데이터 (예: 불완전한 JSON 저장) 가 없어야 합니다.
- IPLD를 업그레이드 할 수 있어야 합니다. 예) 디스크에 저장된 더 나은 형식이 표시되면 시스템을 약간의 비용으로 업그레이드 할 수 있습니다.
- IPLD 객체는 merkle-link 뿐만 아니라 속성을 구문 분석 할 수 있어야 합니다.
- IPLD 에 의해 정의된 형식은 구현 및 변환이 용이해야 합니다.
- IPLD 에 의해 정의된 형식은 전체 객체를 얻지 않고도 검색 가능해야 합니다. (CBOR 및 Protobuf 는 이미 그것을 할 수 있습니다)

## 3. 형식 정의

(JSON과 YML 모두 형식의 형식을 표시하는 데 사용됩니다. 객체의 동등성을 보여주기 위해 둘 다 사용합니다.)

IPLD 데이터 모델의 핵심은 그냥 JSON 입니다. 즉, (a) 일부 기본 유형을 가진 트리 구조 파일, (b) JSON 과 일대일 매핑을 (c)  json 에서 사용될 수 있음을 의미합니다

다른 한편으로 (a) 일부 오류를 수정하고, (b) 효율적인 직렬화를 수행하며 (c) 새로운 기술을 기다리는 단일 전송 형식으로 제한되지 않기 때문에 "JSON"이 아닙니다

### 3.1 Basic Node 단일 노드

다음은 JSON 으로 대표되는 IPLD 객체입니다:

```json
{
  "name": "Vannevar Bush"
}
```

multihash 값이 QmAAA ... AAA 라고 가정합니다. 링크가 없고 문자열만 있습니다. 그러나 사용자는 여전히 keyname 으로 구문 분석 할 수 있습니다.

```
$ ipld cat --json QmAAA...AAA
{
  "name": "Vannevar Bush"
}

$ ipld cat --json QmAAA...AAA/name
"Vannevar Bush"
```

또한, yml 구조체에서:

```yml
$ipld cat --yml QmAAA...AAA
---
name: Vannevar Bush

$ipld cat --xml QmAAA...AAA
<!xml> <!-- todo -->
<node>
  <name>Vannevar Bush</name>
</node>
```

### 3.2 Linking Between Nodes (노드 간 링크)

노드 간의 merkle-link 가 IPLD 의 존재 이유입니다. IPLD 내의 링크는 노드 내에 포함된 특별한 형식일 뿐입니다.

```json
{
  "title": "As We May Think",
  "author": {
    "/": "QmAAA...AAA" // 이전 예제의 데이터에 연결
  }
}
```

위 데이터의 다중 해시 값이 QmBBB ... BBB 라고 가정합니다. 노드에는 QmAAA ... AAA 에 대한 subpath 링크 작성자가 있으므로 다음을 수행할 수 있습니다.

```
$ ipld cat --json QmBBB...BBB
{
  "title": "As We May Think",
  "author": {
    "/": "QmAAA...AAA" // links to the node above.
  }
}

$ ipld cat --json QmBBB...BBB/author
{
  "name": "Vannevar Bush"
}

$ ipld cat --yml QmBBB...BBB/author
---
name: "Vannevar Bush"

$ ipld cat --json QmBBB...BBB/author/name
"Vannevar Bush"
```

### 3.3 Link Properties Convention (링크 속성 컨벤션)

링크를 사용함으로써 IPLD 사용자는 복잡한 데이터 구조를 구축 할 수 있습니다. 따라서 관계 구조를 표시하거나 보조 데이터를 추가하는 것과 같은 정보를 쉽게 인코딩 할 수 있습니다. 그러나 그것은 다음에 논의 될 "대상 링크 협약" 과는 다릅니다. 때로는 다른 객체를 만들지 않고도 링크에 데이터를 추가하기를 원할 때가 있습니다. 실제 IPLD 링크를 다른 객체에 중첩시키고 추가 속성을 사용하면 됩니다.

참고: 링크 속성은 travesal 모호성으로 인해 링크 객체에서 직접 허용되지 않습니다.

예) 파일 시스템이 있고 노드 간의 권한 제어 또는 소유권에 대한 메타 정보를 분배하려고한다고 가정합니다. 해시 값이 QmCCC...CCC 인 객체 디렉토리가 있다고 가정합니다:

```json
{
  "foo": { // link wrapper with more properties
    "link": {"/": "QmCCC...111"} // the link
    "mode": "0755",
    "owner": "jbenet"
  },
  "cat.jpg": {
    "link": {"/": "QmCCC...222"},
    "mode": "0644",
    "owner": "jbenet"
  },
  "doge.jpg": {
    "link": {"/": "QmCCC...333"},
    "mode": "0644",
    "owner": "jbenet"
  }
}
```

YML 표현:

```yml
---
foo:
  link:
    /: QmCCC...111
  mode: 0755
  owner: jbenet
cat.jpg:
  link:
    /: QmCCC...222
  mode: 0644
  owner: jbenet
doge.jpg:
  link:
    /: QmCCC...333
  mode: 0644
  owner: jbenet
```

데이터 구조에 고유 속성에 대한 설명을 추가하더라도 여전히 구문 분석 할 수 있습니다:

```
$ ipld cat --json QmCCC...CCC/cat.jpg
{
  "data": "\u0008\u0002\u0012��\u0008����\u0000\u0010JFIF\u0000\u0001\u0001\u0001\u0000H\u0000H..."
}

$ ipld cat --json QmCCC...CCC/doge.jpg
{
  "subfiles": [
    {
      "/": "QmPHPs1P3JaWi53q5qqiNauPhiTqa3S1mbszcVPHKGNWRh"
    },
    {
      "/": "QmPCuqUTNb21VDqtp5b8VsNzKEMtUsZCCVsEUBrjhERRSR"
    },
    {
      "/": "QmS7zrNSHEt5GpcaKrwdbnv1nckBreUxWnLaV4qivjaNr3"
    }
  ]
}

$ ipld cat --yml QmCCC...CCC/doge.jpg
---
subfiles:
  - /: QmPHPs1P3JaWi53q5qqiNauPhiTqa3S1mbszcVPHKGNWRh
  - /: QmPCuqUTNb21VDqtp5b8VsNzKEMtUsZCCVsEUBrjhERRSR
  - /: QmS7zrNSHEt5GpcaKrwdbnv1nckBreUxWnLaV4qivjaNr3

$ ipld cat --json QmCCC...CCC/doge.jpg/subfiles/1/
{
  "data": "\u0008\u0002\u0012��\u0008����\u0000\u0010JFIF\u0000\u0001\u0001\u0001\u0000H\u0000H..."
}
```

그러나 link 의 파싱은 순회 (traversal) 에 의해 여전히 얻어 져야 합니다.

### 3.4 Duplicate property keys (중복 속성 키)

현재 파서의 경우에도 중복 이름이 있는 속성이 없다는 점은 주목할 가치가 있습니다. 따라서 충돌을 피하기 위해 구문 분석 중에 항목의 첫 번째 항목만 읽습니다. 예) 다음과 같은 객체가 있습니다.

```json
{
  "name": "J.C.R. Licklider",
  "name": "Hans Moravec"
}
```

이 순서대로 표준 형식으로 Canonical Format (json이 아닌 cbor) 이 표준 형식으로 나타나며 해당 해시 값은 QmDDD ... DDD 입니다. 오직 다음을 얻을 수 있습니다:

```
$ ipld cat --json QmDDD...DDD
{
  "name": "J.C.R. Licklider",
  "name": "Hans Moravec"
}

$ ipld cat --json QmDDD...DDD/name
"J.C.R. Licklider"
```

### 3.5 Path Restrictions (경로 제한)

UNIX 와 WEB 에서는 경로 설명에 몇 가지 중요한 문제가 있습니다. (관련 토론도 있습니다)
유닉스와 웹의 모델과 호환되기 위해서 IPLD 는 특정 경로 구성 요소를 가진 경로를 명시적으로 비활성화 합니다. 데이터 자체에는 여전히 이러한 속성이 포함될 수 있습니다. (누군가가이를 수행하고 합법적인 용도로 사용함)
따라서 경로 해석기만 이러한 경로를 통해 구문 분석 할 수 없습니다. 이러한 제한 사항은 일반적인 Unix 및 UTF-8 경로 시스템과 동일합니다.

### 3.6 Integers in JSON

IPLD 는 JSON 과 직접 호환되며 JSON 의 성공을 최대한 활용할 수 있지만 JSON 오류와 관련이 없습니다. 형식 기본 설정을 따를 수 있지만 항상 잘 정의된 1:1 매핑이 있는지 주의해야 합니다. 정수와 관련하여 JSON 에는 EJSON 과 같이 정수를 나타내는 다양한 문자열이 있습니다. 이들을 사용할 수 있으며 다른 형식의 변환은 자연스럽게 이루어져야 합니다. 즉 JSON 을 CBOR 로 변환 할 때 EJSON 정수는 문자열 값으로 나타내는 대신 자연스럽게 적절한 CBOR 정수로 변환되어야 합니다.

### 3.7 데이터 구조 예

IPLD 는 사용자가 신규 또는 가져온 이전 데이터 파일을 정의하는 것을 방해하지 않는 간단하고 유연한 형식입니다. 이를 위해 아래에 몇 가지 샘플 데이터 구조를 보겠습니다.

Unix 파일 시스템

A small File (작은 파일)

```json
{
  "data": "hello world",
  "size": "11"
}
```

A Chunked File 일부

```json
{
  "size": "1424119",
  "subfiles": [
    {
      "link": {"/": "QmAAA..."},
      "size": "100324"
    },
    {
      "link": {"/": "QmAA1..."},
      "size": "120345",
      "repeat": "10"
    },
    {
      "link": {"/": "QmAA1..."},
      "size": "120345"
    },
  ]
}
```

A Directory 

```json
{
  "foo": {
    "link": {"/": "QmCCC...111"},
    "mode": "0755",
    "owner": "jbenet"
  },
  "cat.jpg": {
    "link": {"/": "QmCCC...222"},
    "mode": "0644",
    "owner": "jbenet"
  },
  "doge.jpg": {
    "link": {"/": "QmCCC...333"},
    "mode": "0644",
    "owner": "jbenet"
  }
}
```

git blob 데이터 블록

```json
{
  "data": "hello world"
}
```

git tree 구조

```json
{
  "foo": {
    "link": {"/": "QmCCC...111"},
    "mode": "0755"
  },
  "cat.jpg": {
    "link": {"/": "QmCCC...222"},
    "mode": "0644"
  },
  "doge.jpg": {
    "link": {"/": "QmCCC...333"},
    "mode": "0644"
  }
}
```

git commit 구조

```json
{
  "tree": {"/": "e4647147e940e2fab134e7f3d8a40c2022cb36f3"},
  "parents": [
    {"/": "b7d3ead1d80086940409206f5bd1a7a858ab6c95"},
    {"/": "ba8fbf7bc07818fa2892bd1a302081214b452afb"}
  ],
  "author": {
    "name": "Juan Batiz-Benet",
    "email": "juan@benet.ai",
    "time": "1435398707 -0700"
  },
  "committer": {
    "name": "Juan Batiz-Benet",
    "email": "juan@benet.ai",
    "time": "1435398707 -0700"
  },
  "message": "Merge pull request #7 from ipfs/iprs\n\n(WIP) records + merkledag specs"
}
```

Bitcoin Block

```json
{
  "parent": {"/": "Qm000000002CPGAzmfdYPghgrFtYFB6pf1BqMvqfiPDam8"},
  "transactions": {"/": "QmTgzctfxxE8ZwBNGn744rL5R826EtZWzKvv2TF2dAcd9n"},
  "nonce": "UJPTFZnR2CPGAzmfdYPghgrFtYFB6pf1BqMvqfiPDam8"
}
```

Bitcoin Transaction

```
---
inputs:
  - input: {/: Qmes5e1x9YEku2Y4kDgT6pjf91TPGsE2nJAaAKgwnUqR82}
    amount: 100
outputs:
  - output: {/: Qmes5e1x9YEku2Y4kDgT6pjf91TPGsE2nJAaAKgwnUqR82}
    amount: 50
  - output: {/: QmbcfRVZqMNVRcarRN3JjEJCHhQBcUeqzZfa3zoWMaSrTW}
    amount: 30
  - output: {/: QmV9PkR2gXcmUgNH7s7zMg9dsk7Hy7bLS18S9SHK96m7zV}
    amount: 15
  - output: {/: QmP8r8fLUnEywGnRRUrHB28nnBKwmshMLiYeg8udzYg7TK}
    amount: 5
script: OP_VERIFY
```
