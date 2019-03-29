---
layout: post
title: "IPFS - bootstrap"
categories:
  - IPFS
tags:
  - IPFS
  - bootstrap
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS bootstrap

ipfs 네트워크에서 다른 노드에 연결하려고 할 때 다음과 같은 프로세스가 발생합니다.

- "신뢰할 수 있는" 노드 중 일부에 연결
- "신뢰할 수 있는" 노드를 통해 점차적으로 대상 노드에 연결

"신뢰할 수 있는" 노드를 **bootstrap** 이라고 부릅니다. `bootstrap` 명령은 이러한 _bootstrap_ 을 작동시키는 것입니다.

_bootstrap_ 노드를 추가 및 제거하는 것은 매우 신중해야 합니다. 즉, 반드시 믿을 수 있는 노드가 되어야 합니다. 그렇지 않으면 악성 노드에 연결하거나 아일랜드 노드로 만들 수 있습니다.

명령의 기본 형식은 다음과 같습니다.

```
 ipfs bootstrap <명령어>
```

다음 세 가지 하위 명령이 있습니다:

- add `<peer>`: bootstrap 목록에 노드 추가
  - default bool: 기본 bootstrap 노드 추가

- rm `<peer>`: bootstrap 목록에 노드 제거
  - all bool: 모든 bootstrap 노드 삭제

- `list`: 현재 부트 스트랩 노드 리스트

```
$ ipfs bootstrap list  
/dnsaddr/bootstrap.libp2p.io/ipfs/QmNnooDu7bfjPFoTZYxMNLWUQJyrVwtbZg5gBMjTezGAJN  
/dnsaddr/bootstrap.libp2p.io/ipfs/QmQCU2EcMqAqQPR2i9bChDtGNJchTbq5TbXJJ16u19uLTa  
/dnsaddr/bootstrap.libp2p.io/ipfs/QmbLHAnMoJPWSCR5Zhtx6BHJX9KiKNN6tpvbUcqanj75Nb  
/dnsaddr/bootstrap.libp2p.io/ipfs/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt  
/ip4/104.131.131.82/tcp/4001/ipfs/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ  
/ip4/104.236.179.241/tcp/4001/ipfs/QmSoLPppuBtQSGwKDZT2M73ULpjvfd3aZ6ha4oFGL1KrGM  
/ip4/104.236.76.40/tcp/4001/ipfs/QmSoLV4Bbm51jM9C4gDYZQ9Cy3U6aXMJDAbzgu2fzaDs64  
/ip4/128.199.219.111/tcp/4001/ipfs/QmSoLSafTMBsPKadTEgaXctDQVcqN88CNLHXMkTNwMKPnu  
/ip4/178.62.158.247/tcp/4001/ipfs/QmSoLer265NRgSp2LA3dPaeykiS1J6DifTC88f5uVQKNAd  
/ip6/2400:6180:0:d0::151:6001/tcp/4001/ipfs/QmSoLSafTMBsPKadTEgaXctDQVcqN88CNLHXMkTNwMKPnu  
/ip6/2604:a880:1:20::203:d001/tcp/4001/ipfs/QmSoLPppuBtQSGwKDZT2M73ULpjvfd3aZ6ha4oFGL1KrGM  
/ip6/2604:a880:800:10::4a:5001/tcp/4001/ipfs/QmSoLV4Bbm51jM9C4gDYZQ9Cy3U6aXMJDAbzgu2fzaDs64  
/ip6/2a03:b0c0:0:1010::23:1001/tcp/4001/ipfs/QmSoLer265NRgSp2LA3dPaeykiS1J6DifTC88f5uVQKNAd
```
