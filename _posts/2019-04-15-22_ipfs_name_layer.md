---
layout: post
title: "IPFS - naming"
categories:
  - IPFS_Review
tags:
  - IPFS_naming
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS 기본 - Naming 계층

## 1. IPNS: 네이밍 및 변수 상태

IPFS 는 해당 파일 내용을 검색 할 수 있는 파일 또는 디렉토리에 대해 주소 지정 가능한 해시 값을 생성 할 수 있습니다. 그러나 파일의 내용이 변경되면 이전 해시 값은 최신 내용을 검색 할 수 없으며 이전 내용만 검색 할 수 있습니다. 이러한 방식으로 우리가 다양한 콘텐츠를 게시 할 때 이를 검색하는 것은 매우 불편합니다.

따라서 이 문제를 해결하기 위해 IPNS 는 특정 값을 가변 컨텐츠의 최신 상태로 고정시킵니다. 즉, 파일 내용을 다시 편집 할 때 잠긴 IPNS 값으로 최신 컨텐츠에 액세스 할 수 있습니다.

## 2. 자가 인증 네이밍

SFS 자체 인증 네이밍 시스템은 IPFS 에 암호화 된 배포의 전역 네임 스페이스에서 가변적인 자체 인증 네이밍을 구축하는 방법을 제공합니다. IPFS 네이밍 체계는 다음과 같습니다.

- IPFS 노드 정보는 `NodeId = hash (node.PubKey)` 에 의해 생성
- 각 사용자에게는 변수 네임 스페이스가 할당되고 이전에 생성 된 노드 ID 정보가 주소 이름으로 사용 (_/ipns/_)
- 사용자는 이 경로 아래에 자체 개인 키로 서명 된 객체를 게시 (_/ipns/XLF2ip4ii9x0wejs23HD2swlddVmas8kd0Ax/_)
- 다른 사용자가 객체를 획득하면 서명이 공개 키 및 노드 정보와 일치하는지 여부를 검출함으로써 사용자의 공개 객체의 신뢰성을 검증하고 가변 상태 획득

예) IPNS 링크 : _https://ipfs.io/ipns/QmdKXkeEWcuRw9oqBwopKUa8CgK1iBktPGYaMoJ4UNt1MP_

앞 절인 _https://ipfs.io/ipns/_ 는 네임 스페이스이고 그 뒤에 노드의 ID 인 _NodeID_ 가 옵니다. 모든 노드의 네임 스페이스는 _IPNS_ 입니다.

이 블록의 동적 변화 내용은 라우팅 함수를 설정함으로 제어됩니다. 아래 소스 코드를 통해 네임 스페이스가 바운드 NodeId 의 형태로 마운트 된 이유를 알 수 있습니다: `routing.setValue ( NodeId, <ns-object-hash>)`

네임 스페이스에서 게시 된 데이터 객체 경로 이름은 하위 이름으로 처리 될 수 있습니다:

```tex
/ipns/XLF2ipQ4jD3UdeX5xp1KBgeHRhemUtaA8Vm/
/ipns/XLF2ipQ4jD3UdeX5xp1KBgeHRhemUtaA8Vm/docs
/ipns/XLF2ipQ4jD3UdeX5xp1KBgeHRhemUtaA8Vm/docs/ipfs
```

## 3. Human Read Naming

IPNS 는 노드의 NodeID 를 잠금 해시 값으로 사용하며 사용자는 이러한 긴 문자열을 기억하기 어렵습니다. 따라서 도메인 이름 izone.com 과 같은 정책을 통해 도메인 이름을 설정해야 합니다. ipfs.io/ipns/izone.com 을 사용하여 데이터 컨텐츠에 액세스 할 수 있습니다.

[1] peer 노드 링크

SFS (Self-Validating File System)의 디자인 개념에 따라 사용자는 다른 사용자 노드의 개체를 자신의 네임 스페이스에 직접 연결할 수 있으므로보다 신뢰할 수있는 네트워크를 만들 수 있습니다.

```tex
# Alice 가 Bob 과 링크
ipfs link //friends/bob/

# Eve 는 Alice 와 연결
ipfs link //friends/alice /

# Eve 는 Bob 에 액세스 할 수 있음
//friends/alice/friends/bob

# Verisign 인증 도메인 방문
//foo.com
```

[2] DNS TXT IPNS 레코드

기존 DNS 시스템에 TXT 레코드를 추가하면 도메인 이름을 통해 IPFS 네트워크의 파일 객체에 액세스 할 수 있습니다.

```tex
# DNS TXT 레코드
ipfs.benet.ai. TXT "ipfs=XLF2ip4jD3U..."
# 상징 링크로 표현
ln -s /ipns/XLF2ip4jD3U... /ipns/fs.benet.ai

# 또한 IPFS 는 다음과 같이 이진 인코딩을 읽을 수있는 파일로 변환하는 읽기 가능 식별자 인 Proquint 를 지원
# proquint 성명
/ipns/dahih-dolij-sozuk-vosah-luvar-fuluh
# 아래의 해당 양식으로 분해
/ipns/KhWwNprxYVxKqpDZ
```

또한 IPFS 는 짧은 주소 지정 서비스를 제공합니다. 이 서비스는 지금 볼 수있는 DNS 및 WebURL 링크와 유사합니다.

```tex
# 사용자는 아래에서 link 를 얻을 수 있음
/ipns/shorten.er/foobar

# 자신의 네임 스페이스에 적용
/ipns/XLF2ipQ4JD3Udex6xKbgeHrhemUtaA9Vm
```
