---
layout: post
title: "IPFS - merkle-dag"
categories:
  - IPFS_Review
tags:
  - IPFS_merkel_dag
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS IPLD - merkle-dag

merkle-link 가 있는 객체는 방향성 지도 (Merkle-graph) 를 형성하며 암호화되지 않은 상태로 계산 될 수 있습니다. (암호화 된 해시 함수의 속성이 변경되지 않은 경우, 즉 merkle-dag)
그러므로 merkle-linking (merkle-graph) 을 사용하는 모든 그래프는 또한 비순환 그래프 (DAG) 를 지향해야하므로 merkle-dag 가 되어야합니다.
