---
layout: post
title: "IPFS - id"
categories:
  - IPFS
tags:
  - IPFS
  - id
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS id, ping

네트워크, ID 및 ping 과 관련된 비교적 간단한 두 명령에 대해 설명합니다.

## Id

```tex
ipfs id <peerID>: ipfs 노드 ID 정보 표시
```

`-f` 옵션에는 다음과 같은 다섯 가지 값이 있습니다.

- `<id>`: 노드 ID
- `<aver>`: 클라이언트 버전
- `<pver>`: 프로토콜 버전
- `<pubkey>`: 공개 키
- `<addr>`: 주소

```tex
$ ipfs id QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ -f ="<aver>"
go-ipfs/0.4.15/
```

## Ping

```tex
ipfs ping <peerID>: 노드에 ping 패킷 보냄
```

라우팅 시스템을 통해 노드를 발견하고 ping 메시지를 전송하며 왕복 지연 정보를 확인합니다.
`-n` 옵션을 사용하여 int 형식의 보낼 핑 메시지 수를 설정할 수 있습니다. (기본값 10)

```tex
ipfs ping QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
Looking up peer QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
PING QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ.
Pong reveived: time=224.02 ms
Pong reveived: time=223.66 ms
Pong reveived: time=224.12 ms
Pong reveived: time=213.82 ms
Pong reveived: time=212.02 ms
Pong reveived: time=223.95 ms
Pong reveived: time=225.61 ms
Pong reveived: time=225.01 ms
Pong reveived: time=224.70 ms
Pong reveived: time=224.33 ms
Average latency: 222.12ms
```
