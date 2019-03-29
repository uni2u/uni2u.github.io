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

ipfs 네트워크에서 로컬 노드가 시작된 후 공식적으로 제공되는 기본 노드에 대한 링크를 수신하게 되며 IPFS 에서 제공하는 주 네트워크와의 링크가 생성됩니다.

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

- add `<peer>`: bootstrap 목록에 노드 추가 (포트: 4001)
  - default bool: 기본 bootstrap 노드 추가

```
$ ipfs bootstrap add /ip4/192.168.2.91/tcp/4001/ipfs/QmTvb4UDEqNpV6mizrbZyzjZyw7VWvUxgYbnsdTrFSXKYV
added /ip4/192.168.2.91/tcp/4001/ipfs/QmTvb4UDEqNpV6mizrbZyzjZyw7VWvUxgYbnsdTrFSXKYV
node exit 0
```

- rm `<peer>`: bootstrap 목록에 노드 제거
  - all bool: 모든 bootstrap 노드 삭제

```
$ ipfs bootstrap rm  
/ip6/2400:6180:0:d0::151:6001/tcp/4001/ipfs/QmSoLSafTMBsPKadTEgaXctDQVcqN88CNLHXMkTNwMKPnu
create new datastore
removed /ip6/2400:6180:0:d0::151:6001/tcp/4001/ipfs/QmSoLSafTMBsPKadTEgaXctDQVcqN88CNLHXMkTNwMKPnu
node exit 0
```

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

## 출력 노드 정보

ipfs 네트워크에서 노드는 `Qm` 으로 시작하는 46비트 해시 값으로 식별됩니다. 개발 디버그를 사용하여 링크 노드의 정보를 출력하면 다음 노드 정보를 찾을 수 있습니다.

```
netNotifiee conn:[<peer.ID cJCLDD> <peer.ID QCU2Ec> <peer.ID UuQGw9> <peer.ID SoLPpp>],curconn:<peer.ID SoLPpp>
```

소스에서 peerID 는 6개의 문자열 크기에 불과하며 바이트의 해시 값과 일치하지 않지만 직접적인 연관을 발견했습니다: 소스 코드의 peerID 는 해시 값의 3-8번째 문자를 가져옵니다.

## 사설 IPFS 네트워크 구축

_bootstrap_ 이 노드 정보를 편집하는 매커니즘은 사설 ipfs 네트워크를 구축하기 위한 기반을 제공합니다. 즉, 로컬 노드의 일부는 외부 네트워크에 연결되어 있지 않고 ipfs의 기본 네트워크에 연결되어 있지 않지만 기본적으로 로컬 네트워크의 노드에 연결되어 ipfs 개인 네트워크를 상호 연결할 수 있습니다.

```
$ ipfs bootstrap list
create new datastore
/ip4/192.168.2.91/tcp/4001/ipfs/QmTvb4UDEqNpV6mizrbZyzjZyw7VWvUxgYbnsdTrFSXKYV
node exit 0
```

![ipfs bootstrap 연동](/images/ipfs_bootstrap01.png)

위 그림과 같이 노드는 파일 해시를 통해 네트워크에서 해당 파일을 찾을 수 있고 파란색 노드는 노란색 노드를 통해 빨간색 노드에 저장된 파일을 찾을 수 있습니다. 노란색으로 연결된 노드가 빨간색 네트워크에 대한 연결을 삭제하면 파란색 부분은 다음 그림과 같이 사설 IPFS 네트워크를 형성합니다.

![ipfs bootstrap 노드 삭제](/images/ipfs_bootstrap02.png)

위 그림의 사설 IPFS 네트워크는 _bootstrap_ 을 통해 구현됩니다.
