---
layout: post
title: "NDN - Tcp-Bulk-Insert-Repo-Insertion-Protocol"
categories:
  - NDN
tags:
  - Tcp_Bulk_Insert_Repo_Insertion_Protocol
lang: ko
author: "uni2u"
meta: "Springfield"
---

# Tcp Bulk Insert Repo Insertion Protocol

Tcp Bulk Insert Repo Insert 프로토콜은 repo Insert 프로토콜의 특별한 부분입니다.

Repo 와 producer 가 동일한 호스트에서 실행될 때 TCP bulk 를 사용하여 Repo storage 에 직접 데이터를 insert 할 수 있습니다.

## 1. Basic operations

### 1.1 Listen

Listen 은 초기 repo operation 입니다. Repo는 먼저 listen() 을 invoke 하여 TCP 소켓에서 listen 합니다. 호스트 및 포트 번호 정보는 parameter 로 전달됩니다. 전체 insert 는 비동기 프로세스이며 메인 스레드는 다른 자식 스레드에 작업을 listening 및 할당 (assigning) 합니다.

### 1.2 Handle

Handle 은 다음 2 가지 operations 이 포함됩니다: handleAccept, handleReceive. HandleAccept() 는 listen() 의 콜백 함수입니다. 소켓에서 연결을 수락하고 프로세스 후 다음 연결을 수락할 준비를 합니다. 연결이 끝나면 데이터 패키지를 분석하기 위해 HandleReceive 가 호출됩니다.

### 1.3 StartReceive

handleReceive 함수를 시작하고 비동기 수신을 준비하십시오.

ProcessId 는 클라이언트가 삭제 프로세스를 진행하기 위해 생성한 난수입니다. Repo 는  ProcessId 를 삭제 프로세스와 일치시킵니다.
