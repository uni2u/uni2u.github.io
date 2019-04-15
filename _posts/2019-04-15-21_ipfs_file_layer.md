---
layout: post
title: "IPFS - file"
categories:
  - IPFS_Review
tags:
  - IPFS_file
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS 기본 - File 계층

IPFS 는 MerkleDAG 의 버전 관리 파일 시스템을 모델링합니다. 이 모델은 Git 과 비슷합니다:

1. block: 가변 크기 블록
2. list: 블록 또는 다른 목록의 모음
3. tree: 블록, 목록 또는 다른 트리 모음
4. commit: 버전 히스토리에 있는 트리의 스냅 샷

## 파일 객체: blob

파일 객체가 저장되면 일반적으로 데이터 분할이 필요하며 각 블록의 크기는 256kb 이지만 작은 파일 (<256kb) 인 경우 분할 할 필요가 없으며 직접 데이터 단위로 저장됩니다. 저장소의 유형은 _blob_ 객체입니다. 

각 _blob_ 객체는 다음과 같이 주소 지정 데이터 단위를 포함합니다.

```go
{
    "data":"some data here", 
    // blobs links 없음
}
```

## 파일 객체: list

IPFS 에 256kb 보다 큰 파일이 저장되면 데이터 분할이 필요합니다. 목록 객체는 여러 개의 _blob_ 객체로 구성되며 내부에 _blob_ 이 중복 될 수 있습니다. _lists_ 객체는 정렬된 _blob_ 및 _list_ 객체를 포함합니다. 이러한 정렬된 _blob_ 및 _list_ 에는 자체 _link_ 가 있습니다. _link_ 는 데이터 정보에 해당합니다.

```go
{
    "data":["blob","list","blob"],  // list 데이터 형식 객체 유형
    "links":[
        {
            "hash":"XLYkgq61DYaq8Nhkcqy7LcnSA7dSHQ78x",
            "size":189458
        },
        {
            "hash":"XLHBNsgoepUDKL8dkd9Hesa5io9sdxi7n",
            "size":19442
        },
        {
            "hash":"XLWVQKJII8v7dggkfdhHSFlkaw9yjs7dj",
            "size":5286
        } // links 에서 lists 는 이름이 없음
    ]
}
```

## 파일 객체: tree

IPFS 에서 Tree 객체는 디렉토리를 나타내는 Git 트리와 비슷하거나 해시에 이름을 매핑하는 테이블이며 해시 값은 _blob_, _list_, _tree_ 또는 _commit_ 을 나타내며 구조는 다음과 같습니다.

```go
{
    "data":["blob","list","blob"], // tree 데이터 객체 유형 배열
    "links":[
        {
            "hash":"XLYkgq61DYaq8Nhkcqy7LcnSA7dSHQ78x",
            "name":"less",
            "size":189458
        },
        {
            "hash":"XLHBNsgoepUDKL8dkd9Hesa5io9sdxi7n",
            "name":"script",
            "size":19442
        },
        {
            "hash":"XLWVQKJII8v7dggkfdhHSFlkaw9yjs7dj",
            "name":"template",
            "size":5286
        } // trees 는 이름이 없음
    ]
}
```

## 파일 객체: commit

IPFS 에서 _commit_ 객체는 버전 히스토리에 있는 객체의 스냅샷을 나타내며 Git 의 커밋과 매우 비슷하지만 모든 객체 유형을 가리킬 수 있습니다.

```go
{
  "data": {
    "type": "tree",
    "date": "2014-09-20 12:44:06Z",
    "message": "This is a commit message."
  }, "links": [
  { "hash": "XLa1qMBKiSEEDhojb9FFZ4tEvLf7FEQdhdU",
  "name": "parent", "size": 25309 },
  { "hash": "XLGw74KAy9junbh28x7ccWov9inu1Vo7pnX",
  "name": "object", "size": 5198 },
  { "hash": "XLF2ipQ4jD3UdeX5xp1KBgeHRhemUtaA8Vm",
  "name": "author", "size": 109 }
  ]
}
```
```
> ipfs file-cat <ccc111-hash> --json
{
  "data": {
    "type": "tree",
    "date": "2014-09-20 12:44:06Z",
    "message": "This is a commit message."
}, "links": [
    { "hash": "<ccc000-hash>",
      "name": "parent", "size": 25309 },
    { "hash": "<ttt111-hash>",
      "name": "object", "size": 5198 },
    { "hash": "<aaa111-hash>",
      "name": "author", "size": 109 }
  ]
}

> ipfs file-cat <ttt111-hash> --json
{
  "data": ["tree", "tree", "blob"],
  "links": [
    { "hash": "<ttt222-hash>",
      "name": "ttt222-name", "size": 1234 },
    { "hash": "<ttt333-hash>",
      "name": "ttt333-name", "size": 3456 },
    { "hash": "<bbb222-hash>",
      "name": "bbb222-name", "size": 22 }
 ] 
}

> ipfs file-cat <bbb222-hash> --json
{ "data": "blob222 data", "links": [] }
```

## 버전 관리

IPFS 버전 제어는 Git 위에 구현되며, Git 버전 제어는 이전 장에서 다루었습니다. _commit_ 객체는 기록 버전의 객체 스냅 샷을 나타냅니다. 서로 다른 두 _commit_ 간에 객체 데이터를 비교하면 두 버전의 파일 시스템 간 차이가 나타납니다. IPFS 는 Git 버전 제어 도구의 모든 기능을 구현하며 Git 과도 호환됩니다.

호환성의 이유는 다음과 같습니다.

- Git 도구 버전을 사용하여 마이그레이션된 IPFS
- IPFS 는 FUSE 파일 시스템을 마운트합니다. Git 저장소로 IPFS tree 마운트하고 Git 파일 시스템을 IPFS 형식으로 변환

## 파일 시스템 경로

MerkleDAG 에서 문자열 경로 API 를 사용하여 IPFS 객체를 순회 할 수 있음을 알 수 있습니다. IPFS 파일 객체는 IPFS 를 UNIX 파일 시스템에 쉽게 마운트 할 수 있도록 고안되었습니다.

## 파일을 list 와 blob 으로 분할

대용량 파일의 버전 관리 및 배포의 주된 과제는 파일을 별도의 블록으로 분리하는 올바른 방법을 찾는 것입니다. IPFS 가 각기 다른 유형의 파일에 대해 올바른 분리 방법을 제공하기 위해 다음 옵션을 제공합니다.

Rabin Fingerprints 알고리즘을 사용하여 적절한 블록 경계를 정의합니다. rsync 및 rolling-checksum 알고리즘을 사용하여 버전 간 블록 변경을 감지합니다. 사용자가 파일 크기를 설정하고 데이터 블록의 분할 전략을 조정할 수 있습니다.

## 경로 찾기 성능

경로 기반 액세스는 전체 개체 그래프를 탐색해야하며 각 개체를 검색하려면 DHT 에서 key 값을 찾고 피어에 연결하고 해당 데이터 블록을 검색해야 합니다. 조회되는 경로에 여러 경로가있는 경우 특히 상당한 성능 오버 헤드가 발생합니다. IPFS 는 이것을 고려하여 다음과 같은 방법으로 완화 할 수 있습니다.

- tree chche: 모든 객체는 해시 주소가 지정되므로 무기한으로 캐시 될 수 있으며 tree 는 일반적으로 작기 때문에 blob 과 IPFS 는 tree 를 먼저 캐시합니다.
- flattened trees: 특정 tree 의 경우 모든 오브젝트에 액세스 할 수 있는 링크 된 목록을 작성할 수 있습니다. flat tree 에서 name 은 원래 tree 에서 분리 된 경로로 구분됩니다.

예) 위의 ttt111 에 대한 flat tree 는 다음과 같습니다:

```go
{
  "data":["tree","blob","tree","list","blob","blob"],
  "links":[
    {"hash":"<ttt222-hash>","size":1234,"name":"ttt222-name"},
    {"hash":"<bbb111-hash>","size":123,"name":"ttt222-name/bbb111-name"},
    {"hash":"<ttt333-hash>","size":3456,"name":"ttt333-name"},
    {"hash":"<lll111-hash>","size":578,"name":"ttt333-name/lll111-name"},
    {"hash":"<bbb222-hash>","size":22,"name":"ttt333-name/lll111-name/bbb222-name"},
    {"hash":"<bbb222-hash>","size":22,"name":"bbb222-name"},
  ]
}
```
