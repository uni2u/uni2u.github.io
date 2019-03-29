---
layout: post
title: "IPFS - object"
categories:
  - IPFS
tags:
  - IPFS
  - object
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS object

ipfs 네트워크에서 업로드한 파일 하나하나가 하나의 object 이며, 일종의 DAG 데이터 유형을 가지고 있습니다.
object 명령은 ipfs 에서 DAG 객체와 상호 작용하는 데 사용됩니다.
기본 형식은 다음과 같습니다:

```
ipfs object <명령어> hash
```

다음과 같은 8 개의 하위 명령이 있습니다:

- `data`: 객체의 데이터 부분의 원시 바이트 출력, stdout
  - 출력은 원시 데이터이므로 -encoding 옵션은 출력에 영향을주지 않음

- `diff`: 두 object 의 차이점 표시
  - v: 추가 정보 출력

- `get`: DAG 노드 가져 오기 및 직렬화, stdout
  - 인코딩 옵션; protobuf, json, xml의 세 가지 데이터 출력 형식 지정

- `links`: 출력 대상의 각 분할에 대한 링크
  - v: 헤더 출력

- `new`: 제공된 템플릿을 기반으로 새 객체 만들기
  - 새 객체 만들기 템플릿이 제공되지 않으면 기본적으로 빈 객체 생성

- `patch`: 기존 DAG (사용자 지정 DAG) 를 기반으로 새 object 생성
  - 다음과 같은 4 개의 부속 명령이 있습니다: 
    - `add-link <root><name><ref>`: 지정된 객체에 링크 추가
      - root: 조정할 노드를 지정하는 hash
      - name: 생성 될 노드의 이름
      - ref: 추가 할 링크
      - -p: 브로커 노드 생성

    - `append-data <root><data>`: DAG 노드의 데이터 세그먼트에 데이터 추가
      - root: 조정할 노드를 지정하는 hash
      - data: 추가 할 데이터

    - `rm-link <root><link>`: object 에서 링크 제거
    - `set-data <root><data>`: object 데이터 세그먼트 설정
 
- `put`: 입력 정보를 DAG object 로 저장하고 hash 출력

- `stat`: object 의 상태 제공
