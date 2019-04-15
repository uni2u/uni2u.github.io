---
layout: post
title: "IPFS - routing"
categories:
  - IPFS_Review
tags:
  - IPFS_routing
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS 기본 - Routing 계층

이 계층에서는 이전에 살짝 이야기 된 분산 해시 테이블인 DHT 를 알아보고자 합니다. 각 노드에는 NodeID 라는 고유 ID 가 있습니다. 네트워크 거리 및 대기 시간을 기반으로 라우팅 테이블을 생성합니다. DHT 분산 해쉬 테이블은 피어 노드가 소유 한 데이터에 따라 생성되고 테이블의 key 는 파일 내용의 해시값이며 value 는 데이터에 해당하는 노드 NodeID 입니다.

노드가 IPFS 네트워크에 참여하면 시드 노드를 통해 다른 많은 피어 노드 정보를 수신하고 호스트는 이러한 노드를 연결하고 _ping_ 시간 및 노드 거리에 따라 라우팅 테이블을 구성합니다. 라우팅 테이블은 여러 개의 k-버킷으로 구성되며 k-버킷은 로컬 노드의 NodeID 수에 따라 분산됩니다. 
예) 내 노드가 01011이고 k-버킷의 값이 5 (k = 5) 이면 가까운 버켓과 먼 버킷의 k-버킷 분포는 다음과 같습니다:

- k1 : 01010
- K2 : 01000, 01001
- K3 : 01110, 01111, 01100, 01101
- k4 : 00 으로 시작하는 노드

k-버킷의 노드는 ping 지연 시간에 의해 3 개의 계층으로 더 나누어진다. 3 계층은 각각 20 밀리 초, 60 밀리 초, 그 이상의 지연이다.

노드가 데이터를 저장할 때 데이터는 로컬 노드 IPFS 저장소에 직접 저장됩니다. 설정 데이터의 해시값은 key 이며 key 근처의 다른 노드로 브로드캐스팅되어 key 데이터를 저장했다는 사실을 알려줍니다. 다른 노드는 이 정보를 저장하고 DHT 테이블을 구성하며 내용은 key/value 저장소이고 key 는 데이터 해시값이며 value 는 정보를 저장하는 노드입니다.

노드가 키 데이터를 검색 할 때, 먼저 자신의 blockstore 를 검색합니다. 키에 가장 가까운 k-버킷에 쿼리가 없으면 내부 노드는 자체 DHT 테이블에서 쿼리합니다. key 에 대한 value 가 있으면 피드백이 반환되고 그렇지 않으면 key 에 가장 가까운 노드가 제공됩니다.

IPFS 의 DSHT 구조는 저장된 데이터의 크기에 따라 차별화 됩니다. 작은 값 (1KB 이하) 은 DHT 에 직접 저장되며 큰 값의 경우 DHT 는 인덱스만 저장합니다. 이 인덱스는 피어 노드의 NodeId 이며 피어 노드는 유형값에 대한 특정 서비스를 제공 할 수 있습니다. DSHT 의 인터페이스는 다음의 libP2P 모듈에 있습니다:

``` cpp
type IPFSRouting interface {
  FindPeer (node NodeId)
  // gets a particular peer’s network address 
  // 특정 노드 네트워크 주소 가져 오기
  SetValue (key[]bytes, value []bytes)
  // stores a small meta data value in DHT 
  // DHT 테이블을 통해 더 작은 메타 파일 저장
  GetValue (key[]bytes)
  // retrieves small meta data value from DHT 
  // DHT 테이블에서 메타 파일 검색
  ProvideValue (key Multihash)
  // announces this node can serve a large value
  // 노드를 브로드캐스트하여 메타 파일에 해당하는 데이터 제공
  FindValuePeers (key Multihash, min int)
  // gets a number of peers serving a large value 
  // 데이터 제공 노드 정보 얻기
}
```
