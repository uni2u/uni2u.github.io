---
layout: post
title: "IPFS - ipld"
categories:
  - IPFS_Review
tags:
  - IPFS_ipld
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS Application 계층 - IPLD

![](https://d2w9rnfcy7mm78.cloudfront.net/1460904/display_a17b47ccd1506a4a2db5cf8183f706a6?1511808471)

많은 대중적인 시스템 (git, bittorrent, ipfs, tahoe-lafs, sfsro) 이 merkle-tree 와 해시 링크 관련된 데이터 구조를 사용합니다. IPLD (Inter Planetary Linked Data) 는 다음과 같은 개념을 정의합니다:

- merkle-links: merkle-graph 의 핵심 단위
- merkle-dag: merkle-links 의 지도
- merkle-paths: 유닉스 스타일 경로는 merkle-dag 를 쉽게 탐색 할 수 있음
- IPLD Data Model: merkle-dag 를 표현하기위한 유연한 JSON 기반 데이터 모델
- IPLD Serialized Formats: JSON, CBOR, CSON, YAML, Protobuf, XML, RDF 등과 같이 IPLD 객체가 사용할 수 있는 형식의 목록
- IPLD 신뢰형식: 데이터에 대해 동일한 해석 논리를 보장하는 시퀀스 형식에 대한 명확한 설명으로 merkle-link 및 기타 암호화 응용 프로그램에 중요

![](http://image.chaindesk.cn/IPLD%E7%BB%84%E4%BB%B6.jpg/mark)

![](https://pocket-image-cache.com/direct?url=http%3A%2F%2Fimage.chaindesk.cn%2FIPLD%25E7%25BB%2593%25E6%259E%2584.jpg%2Fmark&resize=w1408)

요약하면: IPLD 는 merkle-link 가 통과 할 수있는 JSON 파일 객체입니다.

IPLD 의 구성 요소는 다음 그림과 같이 구성됩니다. CID, IPLD 트리, IPLD Resolver.

![](https://i.warosu.org/data/biz/img/0120/19/1544045573897.png)

