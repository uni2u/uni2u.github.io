---
layout: post
title: "NDN - Repo-Command"
categories:
  - NDN
tags:
  - Repo_Command
lang: ko
author: "uni2u"
meta: "Springfield"
---

# Repo Command

repo 의 insert, delete 및 기타 작업의 Command 는 [_Signed Interests_](https://redmine.named-data.net/projects/ndn-cxx/wiki/SignedInterest) 형식으로 인코딩됩니다.

```protobuf
/<repo-prefix>/<command-verb>/................./.........................................
                              \______  _______/ \__________________  ___________________/
                                     \/                            \/
                             RepoCommandParameter   Signed Interest additional components
```

_repo command interest_ 의 의미는 다음과 같습니다:

name 의미 (semantics) 는 다음과 같은 컴포넌트로 정의됩니다:

- `<repo prefix>`: 특정 prefix 인 repo 가 수신대기 (listening) 를 나타냅니다.
- `<command verb>`: command name 을 나타냅니다.
- `<RepoCommandParameter>`: repo command 의 parameters 를 나타냅니다.

다음 컴포넌트는 액세스 제어를 위한 _singed interest_ 컴포넌트 입니다.

- `<timestamp>`
- `<random-value>`
- `<SignatureInfo>`
- `<SignatureValue>`

repo 의 prefix 인 _/ucla/cs/repo/_ 는 다음과 같이 정의됩니다:

```protobuf
/ucla/cs/repo/<command verb>/<RepoCommandParameter>/<timestamp>/<random-value>/<SignatureInfo>/<SignatureValue>
```

## 1. RepoCommandParameter

```tex
RepoCommandParameter ::= REPOCOMMANDPARAMETER-TYPE TLV-LENGTH
                           Name?
                           StartBlockId?
                           EndBlockId?
                           ProcessId?
                           InterestLifetime?
                           ForwardingHint?


Name                  ::= NAME-TYPE TLV-LENGTH NameComponent*
NameComponent         ::= NAME-COMPONENT-TYPE TLV-LENGTH BYTE+

StartBlockId          ::= STARTBLOCKID-TYPE TLV-LENGTH
                           nonNegativeInteger

EndBlockId            ::= ENDBLOCKID-TYPE TLV-LENGTH
                           nonNegativeInteger

ProcessId             ::= PROCESSID-TYPE TLV-LENGTH
                           nonNegativeInteger

InterestLifetime      ::= INTEREST-LIFETIME-TYPE TLV-LENGTH
                           nonNegativeInteger

ForwardingHint       ::= FORWARDING-HINT-TYPE TLV-LENGTH
                           Delegation+
```

### 1.1 StartBlockId, EndBlockId

StartBlockId 및 EndBlockId 는 세그먼트 데이터를 처리하는 데 사용됩니다. StartBlockId 는 첫 번째 세그먼트 번호를 나타내고 EndBlockId 는 마지막 세그먼트 번호를 나타냅니다. Repo 는 StartBlockId 와 EndBlockId 사이의 세그먼트 ID를 가진 세그먼트 데이터를 처리합니다. StartBlockId 가 없으면 repo 프로세스의 첫 번째 세그먼트 ID 는 0 입니다. EndBlockId 가 없는 시나리오는 Repo Insertion Command 섹션과 Repo Deletion Command 섹션의 특정 프로세스에 설명되어 있습니다.

### 1.2 Conflict of Selectors and StartBlockId, EndBlockId

Repo 는 RepoCommandParameter 에서 Selector 및 StartBlockId, EndBlockId 와 함께 명령을 처리 할 수 없습니다. RepoCommandParameter 가 둘 다 전달되면 Repo 는 이 명령을 무시하고 오류 코드 405를 반환합니다.

### 1.3 ProcessId

ProcessId 는 insert 및 delect 확인 명령에 사용되어 특정 insert 및 delete 프로세스를 나타냅니다. ProcessId 는 insert 및 delete 명령의 repo 명령 응답에 의해 가져오기 (fetch) 됩니다.

### 1.4 InterestLifetime

InterestLifetime 은 전송 된 _interest_ 와 수신 된 데이터 간의 최대 대기 시간입니다. InterestLifetime 후에 수신 된 데이터가 없으면 _interest_ 시간이 초과됩니다. InterestLifetime 은 선택 사항이며 지정되지 않으면 기본값이 설정됩니다.

### 1.5 ForwardingHint

ForwardingHint 요소는 Link Object 섹션에 정의된 name 위임 (delegation) 목록을 포함합니다. 각 위임은 요청 된 데이터 패킷이 위임 경로를 따라 _interest_ 를 전달함으로써 검색 될 수 있음을 의미합니다. ForwardingHint 의 _interest_ 에 대한 forwarding logic 의 특성은 별도의 문서로 정의됩니다.

## 2. Repo Command Response

Repo command 의 응답 (response) 은 _Repo command interest_ 에 대한 응답 데이터 패킷입니다. 응답에는 명령 프로세스 및 기타 정보의 상태를 나타내는 상태 코드가 포함됩니다. RepoCommandResponse 라는 TLV 인코딩 블록은 데이터 패킷의 내용으로 인코딩됩니다.

```tex
RepoCommandResponse   ::= INSERTSTATUS-TYPE TLV-LENGTH
                           ProcessId?
                           StatusCode
                           StartBlockId?
                           EndBlockId?
                           InsertNum?
                           DeleteNum?

ProcessId            ::= PROCESSID-TYPE TLV-LENGTH
                            nonNegativeInteger 

StatusCode            ::= STATUSCODE-TYPE TLV-LENGTH
                            nonNegativeInteger    

StartBlockId          ::= STARTBLOCKID-TYPE TLV-LENGTH
                            nonNegativeInteger

EndBlockId            ::= ENDBLOCKID-TYPE TLV-LENGTH
                            nonNegativeInteger

InsertNum             ::= INSERTNUM-TYPE TLV-LENGTH
                            nonNegativeInteger

DeleteNum             ::= DELETENUM-TYPE TLV-LENGTH
                            nonNegativeInteger
```

### 2.1 Name

name 은 repo command 의 RepoCommandResponse 에 있는 name 을 나타냅니다.

### 2.2 ProcessId

ProcessId 는 command 프로세스 번호를 나타내기 위해 repo 가 생성하는 난수입니다. 클라이언트는 이 ProcessId 를 사용하여 특정 command 의 상태를 확인할 수 있습니다.

### 2.3 StatusCode

StatusCode 는 repo command 프로세스의 상태를 나타냅니다.

### 2.4 StartBlockId, EndBlockId

StartBlockId 및 EndBlockId 는 RepoCommandParameter 와 동일합니다. RepoCommandParameter 중 하나가 누락 된 경우 repo 는 현재 알려진 Id 로 설정합니다. 예를 들어 RepoCommandParameter 에 StartBlockId 가 없으면 응답의 StartBlockId 가 0 으로 설정됩니다. RepoCommandParameter에 EndBlockId 가 없으면 Repo 가 데이터 패킷에 FinalBlockId 를 가져올 때까지 EndBlockId 가 null 로 설정됩니다. 반환 된 데이터 패킷의 FinalBlockId 가 EndBlockId 보다 작으면 EndBlockId 가 FinalBlockId 로 설정됩니다.

### 2.5 InsertNum, DeleteNum

InsertNum 은 얼마나 많은 데이터 패킷이 repo 에 성공적으로 insert 되었는지 나타내는 지표이며 insertion status check 의 응답에 사용됩니다. DeleteNum 은 delete 명령 및 deletion check 명령에 대한 응답으로 사용됩니다. DeleteNum 은 얼마나 많은 데이터 패킷이 repo 에서 성공적으로 삭제되었는지 나타냅니다.

## 3. Repo TLV Type Encoding Number

| Type | Number |
| --- | --- |
|RepoCommandParameter|201|
|StartBlockId|204|
|EndBlockId|205|
|ProcessId|206|
|RepoCommandResponse|207|
|StatusCode|208|
|InsertNum|209|
|DeleteNum|210|
|InterestLifetime|214|

## 4. Repo Trust Model

repo 의 신뢰 모델은 PKI 와 같은 repo 서비스를 배포하는 사람들에 따라 다릅니다. Repo 는 자체 verification policies 를 지정할 수 있으며 데이터 소비자는 자체 신뢰 앵커를 지정할 수 있습니다. NDN [FAQ](http://named-data.net/project/faq/#How_does_NDN8217s_8220trust_management8221_work) 는 어떻게 NDN trust managment 가 작동하는지 보여줍니다.
