---
layout: post
title: "IPFS - ipld-resolver"
categories:
  - IPFS_Review
tags:
  - IPFS_ipld_resolver
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS IPLD - IPLD-Resolvers

IPLD resolver 는 내부 DAG API 모델인 IPLD 데이터 구조 resolver 입니다:

- put(node, options, callback): IPLD 데이터 구조를 저장하는 노드
  - node: IPLD 구조 노드
  - options: 다음 중 하나를 포함해야 하는 객체
    - 1) cid - 노드의 CID
    - 2) [hashAlg], [version] 및 format
      - 노드의 CID 를 만드는데 사용
      - 기본 hashAlg 및 version 은 기본 형식에 해당 
    - 3) callback: 시그니처가있는 콜백 함수
      - function (err, cid) {}, err은 함수가 실행될 때 반환 될 수 있는 오류
      - cid는 저장 장치 객체의 CID

- get(cid [, path] [, options], callback): 지정된 노드 CID 또는 경로로 해당 노드를 검색
  - options: 선택 객체

localResolve 는 bool 유형이며 true 이면 로컬 경로만 해석됩니다.

- callback 은 서명 function (err, result) 가 있는 콜백 함수이며 result 는 다음 내용이 포함된 객체
  - 1) value:  get 에 의해 얻어진 데이터
  - 2) remainderPath: 전체 경로가 해석되는지 또는 localResolve 가 선택되는지 여부
  - 3) cid: 마지막으로 발견된 노드 탐색

- getMany(cids, callback): 동시에 여러 노드 검색
  - callback: 서명된 콜백 function (err, result) 이며 결과는 CID에 해당하는 노드의 배열

- getStream(cid [, path] [, options]): get과 같지만 노드 (인수)를 전달하는 데 사용되는 소스 pull-stream 을 반환
- treeStream(cid [, path] [, options]): cid + path 에서 모든 경로는 pull-stream 에 의해 반환
  - options: recursive 부울 유형-lingks 순회
- remove(cid, callback): 지정된 cid로 노드를 삭제
- support.add(multicodec, formatResolver, formatUtil): IPLD 자료 구조에 일부 지원 추가
- support.rm(multicodec): IPLD 에 대한 일부 지원 제거
