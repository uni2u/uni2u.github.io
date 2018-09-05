---
layout: post
title: "IPFS (InterPlanetary File System)"
categories: 
  - ipfs
  - Distributed File System
tags:
  - ipfs
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS

IPFS (InterPlanetary File System) 의 영문만 해석을 하면 _행성간 파일 시스템_ 입니다. 제목만 보았을때 '무슨 이런 황당한' 이라는 생각이 들었습니다.

일단 주요 링크는 다음과 같습니다.

- IPFS 공식 사이트: <https://ipfs.io/>
- IPFS 공식 도큐먼트: <https://docs.ipfs.io/>
- IPFS 논문: <https://github.com/ipfs/papers/raw/master/ipfs-cap2pfs/ipfs-p2p-file-system.pdf>
- IPFS Demo Youtube: <https://youtu.be/8CMxDNuuAiQ>

앞으로 수많은 어려움이 있을 듯 싶지만 하나씩 알아보고자 합니다.

## IPFS 는 무엇일까?

IPFS 를 최초로 소개한 논문의 Abstract를 살펴보면 다음과 같습니다. 

> IPFS는 'InterPlanetary File System' 의 약자로 모든 컴퓨팅 장치를 동일한 파일 시스템과 연결하려는 P2P 분산 파일 시스템입니다. 어떤면에서 IPFS는 웹과 비슷하지만 IPFS는 하나의 Git 저장소 내에서 오브젝트를 교환하는 단일 BitTorrent swarm 으로 볼 수 있습니다. 즉, IPFS는 content addressed 하이퍼 링크를 사용하여 높은 처리량의 content addressed 블록 스토리지 모델을 제공합니다. 이는 버전화 된 파일 시스템, 블록 체인 및 심지어 영구 웹을 구축 할 수 있는 데이터 구조인 일반화 된 Merkle DAG 를 형성합니다. IPFS 는 분산 해시 테이블, 인센티브화 된 블록 교환 및 자체 인증 네임 스페이스를 결합합니다. IPFS는 단일 장애 지점 (SOF) 이 없으며 노드는 서로를 신뢰 할 필요가 없습니다.

IPFS 를 최초로 설계한 [Juan Benet](https://www.linkedin.com/in/jbenetcs/) 은 다음과 같이 이야기 했습니다.

> When you have IPFS, you can start looking at everything else in one specific way and you realize that you can replace it all -- Juan Benet

나에게는 '모든 것을 다르게 보고 모든 것을 바꾸어보자!' 는 하나의 슬로건 처럼 보였습니다.

IPFS는 파일을 가져 와서 관리하고 버전을 저장하고 버전을 추적 할 수 있는 버전으로 관리하는 파일 시스템입니다. 또한 이러한 파일이 네트워크를 통해 이동하는 방식을 설명하므로 분산 파일 시스템이기도 합니다.

IPFS는 본질적으로 BitTorrent 와 유사한 네트워크에서 데이터와 컨텐트가 어떻게 이동하는지에 대한 규칙을 가지고 있습니다. 이 파일 시스템 계층은 다음과 같은 매우 흥미로운 속성을 제공합니다:

- 완전히 배포 된 웹 사이트
- 원본 서버가없는 웹 사이트
- 클라이언트 측 브라우저에서 완전히 실행할 수있는 웹 사이트
- 대화 할 서버가 없는 웹 사이트

### Content Addressing

IPFS는 서버에 저장되는 객체(사진, 기사, 비디오)를 참조하는 대신 파일의 해시를 기준으로 모든 것을 참조합니다. 브라우저에서 특정 페이지에 액세스하려는 경우 IPFS는 전체 네트워크에 "이 해시에 해당하는 파일을 가지고 있습니까?" 라는 질문을 하고 IPFS에 있는 노드는 해당 파일을 반환할 수 있습니다.

IPFS는 HTTP 계층에서 content addressing 을 사용합니다. 위치별로 문제를 해결하는 식별자를 만드는 대신 콘텐츠 자체를 표현하여 문제를 해결하려고 합니다. 즉, 콘텐츠가 주소를 결정합니다. 이 메커니즘은 파일을 가져와서 암호화된 방식으로 해시하여 매우 작고 안전한 파일 표현으로 끝나도록 하는 것입니다. 따라서 다른 사람이 단순히 같은 해시를 가지고 있는 다른 파일을 만들어서 주소로 사용할 수 없습니다. IPFS에있는 파일의 주소는 보통 루트 객체를 식별하는 해시로 시작한 다음 경로를 따라 내려갑니다. 서버 대신에 특정 객체와 통신하고 그 객체 내의 경로를 확인합니다.

### HTTP vs. IPFS to find and retrieve a file

HTTP에는 식별자가 위치하기 때문에 파일을 호스팅하고있는 컴퓨터를 쉽게 찾을 수있는 멋진 속성이 있습니다. 이것은 유용하며 일반적으로 잘 작동하지만 오프라인 사례 또는 네트워크에서 로드를 최소화하려는 대규모 분산 시나리오에서는 작동하지 않습니다.

IPFS에서는 단계를 두 부분으로 나눕니다:

- content addressing 을 사용하여 파일 식별
- Go and find it - hash가 생겼을 때 네트워크에 누가 '이 콘텐츠를 가지고 있는가? (hash)' 를 물어 보고 해당 노드에 연결하여 다운로드

그 결과 매우 빠른 라우팅이 가능한 peer-to-peer 오버레이가 생성됩니다.

더 많은 정보는 [Alpha Video](https://www.youtube.com/watch?v=8CMxDNuuAiQ) 를 참조하기 바랍니다.
