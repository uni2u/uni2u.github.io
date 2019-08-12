---
layout: post
title: "NDN - Basic-Repo-Insertion-Protocol"
categories:
  - NDN
tags:
  - Basic_Repo_Insertion_Protocol
lang: ko
author: "uni2u"
meta: "Springfield"
---

# Basic Repo Insertion Protocol

기본 Repo insert 프로토콜은 [Repo Command](03_Repo_Command.html) 를 사용합니다.

Repo insert command 는 Repo 가 내용을 검색 (retrieve) 하고 저장 (store) 하도록 요청합니다. 이 command interest 는 _signed interest_ 이며 repo 에 의해 정의된 액세스 제어 정책으로 유효성 (validate) 이 검사됩니다. _Interest_ 유효성이 확인되고 데이터의 name 이 repo 에 존재하지 않는 경우 저장소는 OK status 가 포함된 데이터 객체로 응답하고 insert 데이터를 가져올 _interest_ 를 보내기 시작합니다.

세그먼트 데이터 insert 는 insert 프로토콜에서 지원됩니다. 분할 정보는 RepoCommandParameter 에 정의됩니다. 콘텐츠가 세그먼트화 되면 최종 세그먼트 ID 가 블록에 인코딩 됩니다.

Forwarding hint 는 대규모 NDN 네트워크에서 mobility 지원을 위한 내용입니다. Repo 는 insert 프로토콜에서 Forwarding hint 메소드를 지원합니다. 정기적인 _interest_ 가 라우팅 불가 (unroutable) 면 사용자는 목적지 범위 (destination region) 가 포함된 링크 객체를 사용하여 _interest_ 를 보냅니다. _interest_ 가 사용자 범위 (region) 에지 (edge) 에 도달하면 라우터는 위임 정보를 선택합니다. 중계 라우터 (Intermediate router) 는 이 위임을 통해 _interest_ 를 전달합니다. _interest_ 가 생산자 (producer) 범위 (region) 에지 (edge) 에 도달하면 라우터는 원래 name 으로 _interest_ 를 전달합니다.

## 1. Basic operations

### 1.1 Insert data

Command verb: **insert**

name semantics 는 `insert` 라는 repo command 포멧을 따릅니다.
예를들어 `<repo prefix>` 를 _/ucla/cs/repo_ 로 지정하면 다음과 같습니다:

```protobuf
/ucla/cs/repo/insert/<RepoCommandParameter>/<timestamp>/<random-value>/<SignatureInfo>/<SignatureValue>
```

### 1.2 Insertion status check

Command verb: **insert check**

insert 진행 중에 요청자 (requester) 는 insert 진행 상태를 확인하기 위해 insert status check command 를 보낼 수 있습니다. 이 status check command 는 _signed interest_ 형식입니다. insert status check 명령의 의미는 다음과 같습니다:

insert check 와 같습니다. 예:

```protobuf
/ucla/cs/repo/insert check/<RepoCommandParameter>/<timestamp>/<random-value>/<SignatureInfo>/<SignatureValue>
```

## 2. Formats

### 2.1 RepoCommandParameter

insert 및 insert check command 의 RepoCommandParameter 는 Repo Command 페이지에 있습니다. Name, StartBlockId, EndBlockId 는 insert 에 사용됩니다. Name과 ProcessId 는 insert check command 에 사용됩니다.

insert command 에서 Name 은 repo 가 가져올 (fetch) 데이터의 name 또는 prefix 를 나타냅니다. StartBlockId 또는 EndBlockId 가 설정된 경우 Repo 는 StartBlockId 와 EndBlockId 사이의 세그먼트 번호가 있는 세그먼트 데이터를 검색합니다.

insert check command 에서 Name 은 가져올 (fetch) Repo 에 대한 데이터의 name 또는 prefix 를 나타냅니다. ProcessId 는 지정된 프로세스를 나타내기 위해 RepoCommandResponse 에 의해 설정됩니다.

### 2.2 Insertion status response

이 insert status 데이터 오브젝트는 insert command 와 insert check command 모두의 응답 데이터 오브젝트 일 수 있습니다. repo command response 포멧을 따릅니다.

StatusCode 는 insert status 를 나타냅니다. InsertNum 은 repo  interest 의 데이터 수를 나타냅니다. StartBlockId 및 EndBlockId 는 insert 된 데이터의 시작 및 끝 세그먼트 ID 입니다. InsertNum 은 insert 된 데이터 세그먼트의 수 입니다. ProcessId 는 프로세스의 ID 로 repo 에 의해 생성된 임의의 번호를 나타냅니다.

insert command 의 경우 아래 정의에 따라 상태 코드가 설정되며 insert command 에 따라 StartBlockId 및 EndBlockId 가 설정됩니다. RepoCommandParameter 의 StartBlockId 가 누락되면 응답으로 0 이 설정됩니다. EndBlockId 가 누락되면 설정되지 않습니다.

insert check command 의 경우 아래의 정의에 따라 상태 코드가 설정되며 StartBlockId 와 EndBlockId 는 Repo 가 사용하는 StartBlockId 와 EndBlockId 에 따라 설정되고 InsertNum 은 insert 진행에 따라 설정됩니다. ProcessId 는 확인된 프로세스의 ID 에 따라 설정됩니다. EndBlockId 가 결정되지 않은 경우이 EndBlockId 는 응답으로 설정되지 않습니다.

StatusCode 정의:

|StatusCode|Description|
|---|---|
|100|command 는 OK, 데이터를 가져올 (fetch) 수 있습니다|
|200|모든 데이터가 insert 되었습니다|
|300|insert 작업이 진행 중입니다|
|401|insert command 또는 insert check command 가 무효화되었습니다|
|403|잘못된 (Malformed) command 입니다|
|404|insert 작업이 진행 중입니다|
|405|EndBlockId Missing Timeout|

### 2.3 EndBlockId Missing Timeout

StartBlockId 가 있지만 EndBlockId 가 없고 반환 데이터 패킷에 FinalBlockId 가 없으면 Repo 는 데이터를 계속 가져옵 (fetch) 니다. 이 경우를 막기 위해 EndBlockId missing timeout 이 설정됩니다. Repo 는 StartBlockId 가 있지만 EndBlockId 가없는 경우 타이머를 시작합니다. Timeout 이 발생하면 Repo 는 데이터를 가져 (fetch) 와서 insert 프로세스를 저장 (store) 하고 종료합니다. insert 프로세스 중에 insert check command 가 도착하면 타이머 시간은 0 으로 설정됩니다. FinalBlockId 가 포함 된 데이터 패킷이 도착하면 timeout 이해제됩니다.

## 3. Protocol Process

1. command 의 authorize 시작; 승인 실패가 나지 않으면 3 번으로 이동
2. 승인 실패인 경우 authorization failure 를 나타내는 응답을 보내고 요청을 중단하고 insert process (StatusCode: 401) 적용
3. StartBlockId 및 EndBlockId 가 모두 누락된 경우 7 번으로 이동
4. StartBlockId 와 EndBlockId 가 모두 있고 StartBlockId 가 EndBlockId 보다 작거나 같으면 7 번으로 이동
5. malformed command 인 응답을 보내고 단계를 중단하고 insert process (StatusCode: 403) 적용
6. 승인 완료까지 대기
7. 인증이 실패하면 2 번 (StatusCode : 401) 으로 이동
8. insert 진행중임을 나타내는 응답 전송 (StatusCode : 200)
9. StartBlockId 또는 EndBlockId 가 있는 경우 16 번으로 이동
10. insert command 에서 Name 검색 (retrieve) 시작
11. 검색 완료까지 대기
12. 검색에 실패하면 26 번으로 이동
13. 검색된 데이터 패킷 저장
14. 단계를 중단하고 insert 프로세스 종료
15.  StartBlockId 가 없으면 StartBlockId 를 0으로 설정; EndBlockId 가 누락 된 경우 데이터 패킷에 FinalBlockId 가 표시되지 않으면 EndBlockId 가 누락; EndBlockId Missing Timeout timer 시작
16. Name 에 StartBlockId 추가
17. Name 검색 (retrieve) 시작
18. 검색 완료까지 대기
19. 검색에 실패하면 26 번으로 이동
20. 검색된 (retrieved) 데이터 패킷 저장 (store)
21. 데이터 패킷에 FinalBlockId 가 포함되어 있고 FinalBlockId 가 EndBlockId 보다 작거나 EndBlockId 가 없는 경우 EndBlockId 를 FinalBlockId 로 설정
22. Name 의 마지막 구성 요소가 EndBlockId 보다 크거나 같으면이 단계를 중단하고  insert 프로세스 종료
23. Name의 마지막 컴포넌트를 증가
24. 17 번으로 이동
25. 이 데이터로 2 번 중복하여 데이터 검색; 이러한 2 검색 모두 실패하면 단계를 중단; 성공하면 21 번으로 이동
26. 이 데이터로 2 번 중복하여 데이터 검색; 이러한 2 검색 모두 실패하면 단계를 중단; 성공하면 14 번으로 이동

EndBlockId Missing Timeout 타이머가 시작되면 Repo 는 17 ~ 26 단계에서 타이머를 모니터링합니다. Timeout 이 발생하면 즉시 insert command process 를 중단합니다.

### 3.1 Repo insert check command progress

구현은 insert 진행 상황에 대한 상태 알림 (notification) 을 게시 (publish) 할 수 있습니다. status check 프로세스는 다음과 같습니다.

1. insert status command 에 대한 authorize 시작; 실패하면 2번으로, 성공하면 3 번으로 이동
2. 승인 실패인 경우 authorization failure 를 나타내는 응답을 보내고 요청을 중단하고 insert process (StatusCode: 401) 적용
3. command 에서 데이터 이름으로 insert 진행 점검; 4 번 또는 5 번으로 이동
4. status code 가 있는 response status; 검사 (check) 프로세스 종료 (StatusCode: 404)
5. insert 상태 확인; insert 진행 상태 반환; EndBlockId Missing Timeout 타이머가 실행중인 경우 타이머를 0 으로 설정; 검사 (check) 프로세스 중단 (StatusCode : 300)

### 3.2 Protocol diagram:

```tex
Requester                     Repo                          Data producer
    |                           |                                 |
    |                           |                                 |
  +---+  Insert command       +---+                               |
  |   | --------------------> |   |                               |
  +---+                       |   |                               |
    |                         |   |                               |
  +---+   Confirm start       |   |                               |
  |   | <==================== |   |                               |
  +---+   Reject command      +---+                               |
    |     (with status code)    |                                 |
    |                         +---+     Interest for Data       +---+
    |                         |   | --------------------------> |   |
    |                         +---+                             |   |
    |                           |                               |   |
    |                         +---+       Data segment          |   |
    |                         |   | <========================== |   |
    |                         +---+                             +---+
    |                           |                                 |
    |                           ~                                 ~
    |                           ~                                 ~
    |                           |                                 |
    |                         +---+     Interest for Data       +---+
    |                         |   | --------------------------> |   |
    |                         +---+                             |   |
    |                           |                               |   |
    |                         +---+       Data segment          |   |
    |                         |   | <========================== |   |
    |                         +---+                             +---+
    |                           |                                 |
    |                           |                                 |
    |                           ~                                 ~
    |                           ~                                 ~
    |                           |                                 |
    |                           |                                 |
    |                           |                                 |
  +---+   Status interest     +---+                               |
  |   | --------------------> |   |                               |
  +---+                       |   |                               |
    |                         |   |                               |
  +---+    Status response    |   |                               |
  |   | <==================== |   |                               |
  +---+                       +---+                               |
    |                           |                                 |
    |                           |                                 |
```
