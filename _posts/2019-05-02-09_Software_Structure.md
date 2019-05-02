---
layout: post
title: "NDN - Repo-Software-Structure"
categories:
  - NDN
tags:
  - Repo_Software_Structure
lang: ko
author: "uni2u"
meta: "Springfield"
---

# Software Structure

## Major Modules

- Database Handle: read, insert, update, delete 를 포함한 데이터베이스 관련 작업 처리를 위한 기본 인터페이스; _interest_ 의 selector 에 따라 NDN 패킷에서 작동; 이 모듈은 storage-handle directory 에 위치
- Command Echo: 각 echo module 은 서로 다른 기능의 _interest_ 및 command 를 별도로 처리할 수 있음; read-echo 를 통한 read, write-echo 를 통한 insert 및 insert progress check 그리고 delete-echo 를 통한 삭제
- Helpers: repo command parameter, response 및 repo TLV 포멧 정의
- Server: starting repo 프로세스는 reading configuration file, initiating database, registration prefix 를 포함하여 정의; repo 의 주요 기능 포함
- Test: Unit Test

## Module Relation Graph

```
--------------------------------
|                              |
|         Repo Server          |
|                              |
--------------------------------
              ||
              || contains
              \/
--------------------------------
|                              |
|  Interest and Command Echo   | 
|  Read, Insert and Delete     |
|                              |
--------------------------------
              ||
              || uses 
              \/ 
--------------------------------
|                              |
|       Database Handle        |
|                              |
--------------------------------
```
