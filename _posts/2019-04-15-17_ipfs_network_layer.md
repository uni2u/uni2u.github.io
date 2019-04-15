---
layout: post
title: "IPFS - network"
categories:
  - IPFS_Review
tags:
  - IPFS_network
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS 기본 - Network 계층

네트워크 계층은 IPFS 의 libp2p 모듈입니다. 모듈에는 다음이 포함됩니다:

- Transports: 전송 계층
- Discovery: 네트워크 검색 레이어
- Peer Routing: 노드 라우팅
- NAT Traversal: NAT 레이어
- Content Routing: 콘텐츠 기반 주소 지정

![](https://cecs.wright.edu/~pmateti/Research/IPFS/Figures/libp2p.003.jpg)

네트워크 계층의 역할은 IPFS 노드가 다양한 기본 네트워크 프로토콜 (구성 가능) 을 사용하여 다른 노드에 대한 연결을 관리하고 이들이 정기적으로 통신하는지 확인하는 것입니다.

이 계층은 전체 IPFS 에 대한 기본 네트워크 장비 또는 네트워크 기능을 제공할 뿐만 아니라 암호화 전송, 네트워크 침투, 다중 링크 혼합 및 기타 기술을 추가합니다.

IPFS 노드가 다른 노드에 연결 되면 WAN 을 통과하고 IPFS 네트워크 스택의 기능은 다음과 같습니다:

1. 전송 : IPFS 는 브라우저에서 사용하는 WebRTC 데이터 채널, LED BLAT (Low Latency UTP) 전송 프로토콜 등에 가장 적합한 기존 주류 전송 프로토콜과 호환됩니다.
2. 신뢰성: uTP 및 SCTP를 사용하여 두 프로토콜이 네트워크 상태를 동적으로 조정할 수 있도록 보장 합니다.
3. 연결성: ICE 와 같은 NAT 교차 기술을 사용하여 광역 네트워크 연결성을 달성 합니다.
4. 무결성: 해시 검사를 사용하여 데이터 무결성을 검사합니다 . IPFS 의 모든 데이터 블록에는 고유 해시가 있습니다.
5. 검증 가능성: 데이터 송신자의 공개 키 및 HMAC 메시지 인증 코드를 사용하여 메시지의 신뢰성을 검사합니다.

IPFS 는 모든 네트워크를 사용할 수 있으며 IP 에 의존하지 않습니다. 이를 통해 IPFS 를 사용하여 전체 네트워크를 포괄 할 수 있습니다. IPFS 는 향후 나타날 수 있는 다른 네트워크 프로토콜과 호환 및 확장에 사용되는 대상 주소와 프로토콜을 나타내는 **multiaddr** 형식을 사용합니다.

```
# an SCTP/IPv4 connection
/ip4/10.20.30.40/sctp/1234/

# an SCTP/IPv4 connection proxied over
TCP/IPv4/ip4/5.6.7.8/tcp/5678/ip4/1.2.3.4/sctp/1234/
```
