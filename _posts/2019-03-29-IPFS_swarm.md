---
layout: post
title: "IPFS - swarm"
categories:
  - IPFS
tags:
  - IPFS
  - swarm
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS swarm

ipfs 네트워크에서 노드에 대한 연결을 모니터하고 유지 보수를 담당하는 구성요소 입니다.
Swarm 명령은 `swarm` 구성 요소를 조작하는 데 사용됩니다.
기본 형식은 다음과 같습니다:

```
ipfs swarm <명령어>
```

다음과 같은 5 개의 하위 명령이 있습니다.

- addrs: 알려진 주소 리스트 (디버깅에 유용)
  - id bool: 주소별로 노드를 나열하며, 기본값은 false

- connect `<address>`: 주소 연결
  - 피어 주소에 대한 새로운 연결
  - 주소 형식은 IPFS 멀티 주소

- disconnect `<address>`: 주소 연결 끊기
  - 피어 주소에 대한 연결을 닫음
  - 주소 형식은 IPFS 멀티 주소
  - 연결 해제는 영구적이지 않고 언제든지 다시 연결할 수 있음

- filters: 작업 주소 필터
  - 현재 필터가 적용된 내용 나열
  - 부속 명령을 사용하여 필터 추가 및 제거
  - 주소 형식은 IPFS 멀티 주소

```
/ip4/19.168.0.0/ipcidr/16
는
192.168.0.0/16
와 같음
```

부속 명령 `add` 및 `rm` 은 필터를 추가 또는 제거하는 데 사용되지만 프로세스가 다시 시작될 때 초기화 됩니다. 필터는 기본적으로 `Swarm.AddrFilters` 에 지정된 필터로 필터링 됩니다.

- peers: 열려있는 피어 연결 리스트
  - v bool: 추가 정보 표시
  - streams bool : 동시에 열려있는 스트림에 대한 각 노드 정보
  - latency bool : 각 피어 대기 시간 정보

```
$ ipfs swarm peers
/ip4/104.131.131.82/tcp/4001/ipfs/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
/ip4/178.62.61.185/tcp/4001/ipfs/QmSoLMeWqB7YGVLJN3pNLQpmmEk35v6wYtsMGLzSr5QBU3
```
