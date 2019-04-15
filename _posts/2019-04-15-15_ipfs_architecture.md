---
layout: post
title: "IPFS - architecture"
categories:
  - IPFS_Review
tags:
  - IPFS_architecture
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS 기본 아키텍처는 무엇입니까?

IPFS 아키텍처는 아래와 같이 총 8 개의 레이어로 구성됩니다.

![](https://cdn-images-1.medium.com/max/1600/0*3pjaONvymeTFFdRo.jpg)

위 로부터 _Network (ID, Network), Routing, Exchange, merkleDAG (Object, File), Naming, Application_ 으로 구성되며 각 프로토콜 스택은 자체 역할을 가지고 있으며 서로 쌍을 이룹니다. 각 계층은 다음 섹션에서 분석 됩니다.
