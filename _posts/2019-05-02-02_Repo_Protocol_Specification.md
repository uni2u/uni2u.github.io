---
layout: post
title: "NDN - Repo Protocol Specification"
categories:
  - NDN
tags:
  - Repo_Protocol_Specification
lang: ko
author: "uni2u"
meta: "Springfield"
---

# Repo Protocol Specification

Repo 는 콘텐츠를 보존하고 보유한 콘텐츠를 요청하는 _interest_ 에 응답하는 것으로 네트워크를 지원합니다. 어떤 노드에서도 Repo 가 존재할 수 있는데 특히 해당 노드의 응용 프로그램이 데이터를 보존해야 하는 경우 권장됩니다. NDN repo protocol 이란 repo 에서 데이터 객체 (Data Object) 에 대한 read, insert, delete 사양 (specification) 입니다. 

Repo 의미는 NDN 의 name 끝에 서명된 객체 (signed components) [_Signed Interests_](https://redmine.named-data.net/projects/ndn-cxx/wiki/SignedInterest) 와 basic common semantic 을 기반으로 합니다.

데이터 객체의 insert 및 delete 를 포함하는 repo 의 일부 동작이 요구 될 때 _command interest_ 가 전송됩니다. _command interest_ 는 insert 및 delete 명령으로서의 _interest_ 이며 액세스 제어를 위한 _command interest_ 형식으로 서명됩니다. repo 는 데이터 객체를 사용하여 command 에 응답합니다. 

repo protocol 은 데이터 패킷 검색 및 관리의 두 부분으로 분류할 수 있습니다. Repo-ng 는 다양항 방식으로 데이터를 insert 하고 delete 하는 일련의 repo 관리 프로토콜 (management protocol) 을 구현하고 있습니다.

## Repo Management Protocols

아래 내용은 **[repo-ng](01_repo_ng.html)** 포스팅의 Repo 관리 프로토콜의 내용입니다. 

- **[Repo Command](03_Repo_Command.html)** 는 repo 에 데이터를 insert 하거나 remove 할 수 있는 요청 및 응답에 대한 포멧, 이러한 command 를 서명하고 인증하는 방법을 정의합니다.
- **[Basic Repo Insertion Protocol](04_Basic_Repo_Insertion_Protocol.html)** 은 데이터 패킷의 insert 형식을 정의합니다. repo-ng 는 기본 프로토콜 외에도 몇가지 다른 insert 프로토콜을 구현할 수 있습니다.
  - [Watched Prefix Insertion Protocol](05_Watched_Prefix_Insertion_Protocol.html) : 지속적으로 생성된 데이터를 insert 하는 프로토콜을 정의합니다.
  - [Tcp Bulk Insert Repo Insertion Protocol](06_Tcp_Bulk_Insert_Repo_Insertion_Protocol.html) : 대량의 데이터 패킷 (예: 동일 호스트에서 생성된) 을 insert 하는 간단한 TCP 기반 프로토콜을 정의합니다.
- **[Repo Deletion Protocol](07_Repo_Deletion_Protocol.html)** 은 특정 prefix 아래에서 데이터 패킷에 대한 delete 포멧을 정의합니다.

## Data packet retrieval from repo

Repo 는 보유하고있는 데이터 객체의 prefix 를 NDN forwarding daemon 에 등록하고 Repo 는 해당 prefix 로 응답합니다.

_standard interest_ 는 repo 에서 콘텐츠를 가지고 오는데 사용됩니다. NFD 에 등록된 prefix 와 _interest_ 의 name 이 일치하면 Repo 가 응답합니다. repo 의 콘텐츠가 _interest_ 와 일치하면 데이터 객체로 응답합니다. _interest_ 가 일치하지 않으면 응답하지 않습니다.

프로토콜은 다음과 같습니다.

일치하는 데이터 객체가 있는 경우:

```
Requester                     Repo
    |                           |
    |                           |
    |         Interest          |
 t1 |-------------------------->|
    |                           |
    |        Data Object        |
 t2 |<==========================|
    |                           |
    |                           |
    |                           |
```

일치하는 데이터 객체가 없는 경우:

```
Requester                     Repo
    |                           |
    |                           |
    |         Interest          |
 t1 |-------------------------->|
    |                           |
    |                           |
    |                           |
```

### About Freshness

repo 로 freshness (신선도?) 를 처리하는 솔루션은 명확하게 정의되지 않았으므로 생산자 (producer) 는 repo 에 콘텐츠를 담을 때 freshness 를 관리 (쓸모없는 콘텐츠 삭제) 하여야 합니다. MustBeFresh selector 는 repo 에서 콘텐츠를 가지고 오거나 (fetching), repo command 를 처리할 때 무시 (ignored) 됩니다.
