---
layout: post
title: "NDN - Watched-Prefix-Insertion-Protocol"
categories:
  - NDN
tags:
  - Watched_Prefix_Insertion_Protocol
lang: ko
author: "uni2u"
meta: "Springfield"
---

# Watched Prefix Insertion Protocol

Watched Prefix는 Repo insert 를 위한 새로운 프로토콜입니다. 이 프로토콜을 사용하여 repo 는 동일한 prefix 로 데이터를 요청하도록 _interest_ 를 계속 전송합니다. 데이터 패킷이 수신되면 repo 는 수신된 데이터를 제외하고 새 데이터를 요청하도록 selector (대부분의 경우 exclude selector) 를 업데이트합니다. Repo 는 전송된 _interest_ 의 총량이 특정 숫자 또는 timeout 에 도달하지 못하도록하는 _interest_ 가 표시 될 때까지 prefix 시청을 중단합니다.

## 1. Basic operations

### 1.1 Start watched prefix insertion

Command verb:  **watch start**

name 은 [Repo Command](03_Repo_Command.html) 포멧을 따릅니다.
예를 들어, `<repo prefix>` 를 _/ucla/cs/repo_ 로 지정하면 다음과 같습니다:

```tex
/ucla/cs/repo/watch/start/<RepoCommandParameter>/<timestamp>/<random-value>/<SignatureInfo>/<SignatureValue>
```

이 명령은 다음 parameter 를 사용합니다.

- `Name` (required)
- `InterestLifetime` (optional)
- `MaxInterestNum` (optional)
- `WatchTimeout` (optional)
- `Selectors` (optional)

parameter 에 대한 세부설명:

- **`Name`**: watched prefix
- **`InterestLifetime`**: 전송 interest 와 수신 데이터 간의 최대 대기 시간; 지속시간이 interest 시간 초과보다 크면 동일한 interest 가 다시 전송; interest timeout 내에 데이터가 수신되면 interest selector 가 업데이트되어 새로운 데이터를 요청; 지정하지 않으면 기본값 (4000 밀리 초) 이 설정
- **`MaxInterestNum`**: 전송할 수 있는 총 interest 의 최대 값; interest 의 총 수가이 한도에 도달하면 프로세스 중단; 지정하지 않으면 MaxInterestNum 은 0 으로 설정되고 무한대를 의미
- **`WatchTimeout`**: 프로세스의 지속 시간; Repo 는 시간이 끝날 때까지 prefix 를 계속 watching; WatchTimeout 은 0 으로 설정되며 프로세스는 영원히 계속 실행
- **`Selectors`**: 수신된 데이터를 제외하고 새 데이터를 요청하는 데 사용

### 1.2 Watch status check

Command verb:  **watch check**

watched prefix 진행 도중 requester 는 진행 상태를 확인하기 위해 watch status check command 를 보낼 수 있습니다. watch check command 는 [Repo Command](03_Repo_Command.html) 입니다. watch check command 의 의미는 다음과 같습니다:

```tex
/ucla/cs/repo/watch/check/<RepoCommandParameter>/<timestamp>/<random-value>/<SignatureInfo>/<SignatureValue>
```

이 command 는 다음 parameter 를 사용합니다:

- `Name` (required)

parameter 에 대한 자세한 설명:

- **`Name`** 은 watched prefix 입니다.

### 1.3 Stop watched prefix insertion

Command verb:  **watch stop**

watched prefix 진행중에 requester 는 watch stop prefix insert 를 중지하기 위해 watch stop command 를 보낼 수 있습니다. stop command 은 [Repo Command](03_Repo_Command.html) 입니다.

```tex
/ucla/cs/repo/watch/stop/<RepoCommandParameter>/<timestamp>/<random-value>/<SignatureInfo>/<SignatureValue>
```

command 는 다음 parameter 를 사용합니다:

- `Name` (required)

parameter 에 대한 자세한 설명:

- **Name**: watched prefix

## 2. RepoCommandResponse

이 watch status 데이터 객체는 watch 명령과 watchCheck 명령의 응답 데이터 객체가 될 수 있습니다. repo command 응답 포멧을 따릅니다.

Response 는 두부분이 있습니다: InsertNumber 는 watched prefix 아래의 데이터 패킷이 Repo 에 몇 개 insert 되었는지 나타냅니다;
StatusCode 는 프로세스 상태를 나타냅니다.

StatusCode 정의:

|StatusCode|Description|
|---|---|
|100|command 는 OK; watch the prefix 시작|
|101|Watched Prefix insert 중지|
|300|watched prefix Insert 진행중임|
|401|watch 명령 또는 watchCheck 명령이 무효화 (invalidated)|
|402|BlockId 가 있음. BlockId 는 프로토콜에서 지원되지 않음|
|403|Malformed Command|
|404|프로세스가 진행되고 있지 않음|

## 3. Protocol Process

### 3.1 Repo watch start command process:

1. command 유효성 검사 (validate) 시작; 유효성이 확인되면 3 단계로 이동, 그렇지 않으면 2 단계로 이동
2. validation failure response; 이 단계를 중단하면 프로세스가 종료. (StatusCode: 401)
3. Check parameters; interest 추출 할 수 없는 경우 부정적 응답 (StatusCode: 403) 을 보내고 프로세스 중지
4. BlockId 가 있으면 부정적 응답 (StatusCode: 402) 을 전송하고 프로세스 중지
5. interest 를 구성하려면 interest command 의 parameter 를 사용; Rightmost Child Selector 설정
6. interest 보내고 timer 시작; 전송된 interest + 1 (초기 값은 0)
7. 데이터를 받은 경우 8 단계로 이동, 시간이 초과되면 17 단계로 이동
8. 데이터의 유효성이 확인되면 9 단계로 이동, 그렇지 않으면 15 단계로 이동
9. watched prefix insert 실행 중인지 확인; 그렇지 않으면 프로세스 종료
10. process times out(exceed watch timeout) 되거나 전송된 interest 의 총 수가 제한에 도달하면 11 단계로 이동, 그렇지 않으면 12 단계로 이동
11. 모든 variables (전송된 interest 수, watch timeout, interest lifetime) 를 지우고 (단계를 중단) 프로세스 종료
12. 데이터를 repo 에 store
13. Update selector 및 수신된 항목을 min 에서 제외하고 업데이트된 selector 를 사용하여 새로운 interest 를 생성
14. 신규 interest 와 전송된 interest 수에 1 을 더한 다음 7 단계로 이동
15. 9-11 단계를 반복하고 12 단계를 건너뜀
16. selector 를 업데이트 하고 수신 된 데이터만 제외 (name 이 작은 다른 데이터가 충족 될 수 있으므로); 14 단계로 이동
17. 9-11 단계를 반복하고 12 단계를 건너뜀
18. interest 를 보내고 전송된 interest 수에 1 을 더한 다음 7 단계로 이동

### 3.2 Repo watch check command process:

1. check command validate 시작; 유효성이 확인되면 3 단계로 이동하고, 그렇지 않으면 2 단계로 이동
2. validation failure 를 나타내는 응답을 보내고 이 단계를 중단하면 프로세스 종료 (StatusCode: 401)
3. Check parameter; interest 에서 추출 할 수 없거나 checked 할 name 이 없는 경우 부정적인 응답 (StatusCode: 403) 을 보냄
4. 해당 프로세스가 존재하는지 확인; 그렇다면 5 단계로 이동, 그렇지 않은 경우 부정 응답을 보냄 (StatusCode: 404)
5. processId 를 사용하여 응답을 찾고 응답을 되돌려 보냄

### 3.3 Repo watch stop command process:

1. stop command validate 시작; 유효성이 확인되면 3 단계로 이동, 그렇지 않으면 2 단계로 이동
2. validation failure 를 나타내는 응답을 보내고 이 단계를 중단하면 프로세스 종료 (StatusCode: 401)
3. Check parameter; interest 추출 할 수 없는 경우 부정 응답을 보냄 (StatusCode: 403)
4. 프로세스를 중지하고 부정적인 응답을 보냄 (StatusCode: 101)
