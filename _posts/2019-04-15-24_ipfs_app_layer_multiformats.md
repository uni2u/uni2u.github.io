---
layout: post
title: "IPFS - multiformats"
categories:
  - IPFS_Review
tags:
  - IPFS_multiformats
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS Application 계층 - Multiformats

## 1. Multiformats 이란 무엇인가?

Multiformats 프로젝트는 IPFS 프로토콜용으로 특별히 제작되었습니다. 프로토콜의 상호 운용성, 유연성 유지, 확장성, 업그레이드 지원 시스템을 만드는 것입니다. 현재 IPFS 및 libp2p 모듈에서 사용되는 IPFS 시스템은 주로 ID 암호화 및 데이터 자가 기술을 담당합니다. 미래의 보안 시스템을 위한 프로토콜로 자가 기술 형식 값을 강화하여 구현됩니다. 자가 기술 형식은 시스템이 공동 작업하고 업그레이드 할 수 있게 합니다.

계약의 자가 기술 측면에는 아래 조항이 있습니다:

- 문맥으로부터 판단되는 값이 아닌 특정 값을 지정해야합니다.
- 확장성을 증가시켜야 합니다.
- 컴팩트해야하며 2 진 형식으로 표현 될 수 있습니다.
- 사람이 읽을 수 있는 표현이 있어야 합니다.

## 2. Multiformats 구성

Multiformats 의 구성 요소는 다음과 같습니다.

- multihash: 자가 기술 해시 값
- multiaddr: 자가 기술 네트워크 주소
- multibase: 자가 기술 코드 값
- multicodec: 자가 기술 직렬화 값
- multistream: 자가 기술 네트워크 전송 스트림
- multigram (WIP): 자가 기술 패킷 네트워크 프로토콜

