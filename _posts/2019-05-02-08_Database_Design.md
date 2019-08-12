---
layout: post
title: "NDN - Repo-Database-Design"
categories:
  - NDN
tags:
  - Repo_Database_Design
lang: ko
author: "uni2u"
meta: "Springfield"
---

# Database Design

## Lists of Tables

- NDN_REPO

## Specifics of Tables

### :: NDN_REPO

#### :::: Columns

- name: TLV 데이터 패킷의 블록 이름 (BLOB PRIMARY KEY)
- data: 전체 TLV 포멧의 데이터 패킷 (BLOB)
- parentName: name length 가 ‘name - 1’ 인 TLV 포멧의 prefix (BLOB)
  - 예) name 이 /a/b/c/ 이면 parentName 은 /a/b
  - root name 이 / 인 경우 parentName 은 null
- nChildren: name 의 direct children 수 (INTEGER)
  - 예) raw name: /a/b, /a/b/c, /a/b/d, /a/b/c/d (/a/b 의 children: 2, /a/b/c 의 children: 1, /a/b/d 및 /a/b/c/d 의 children: 0)

#### :::: Note

name 및 pname 을 사용하여 테이블은 prefix tree 를 작성합니다.
