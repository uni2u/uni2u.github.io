---
layout: post
title: "IPFS - files"
categories:
  - IPFS
tags:
  - IPFS
  - files
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS files

```protobuf
ipfs files <명령어>
```

- `chcid <path>`: 주어진 경로의 루트 노드의 cid 버전 또는 hash 함수 변경
  - cid-var int: 채택 할 cid 버전
  - hash string: 사용할 hash 함수

- `cp <source><dest>`: 파일을 mfs에 복사

- `flush <path>`: 주어진 경로의 데이터를 디스크로 플러시

- `ls <path>`: 로컬 변수 네임 스페이스의 디렉토리 리스트

- `mkdir <path>`: 디렉토리 생성
  - p bool: 필요에 따라 부모 디렉토리 생성

- `mv <source><dest>`: 파일을 source 에서 dest 로 이동하면 hash 변경

- `read <path>`: 주어진 mfs 에 있는 파일을 읽음
  - o int: 오프셋을 읽기 시작하는 바이트 (int)
  - n int: 최대 바이트 (int)

- `rm <path>`: 파일 삭제
  - r: 재귀적 삭제

- `stat <path>`: 주어진 경로 파일의 상태
  - format string: 출력 형식 지정
  - hash bool: 해시만 출력
  - size bool: 크기만 출력

- `write <path><data>`: path 파일에 가변적인 data 파일 작성
  - o, n: 오프셋을 읽기 시작하는 바이트 (int), 최대 바이트 (int)
  - e bool: 파일이 존재하지 않으면 생성
  - t bool: 파일을 작성하기 전에 원본 내용을 지우고 처음부터 작성

```protobuf
$ipfs files ls
h
hide
hide1

$ipfs files stat /h
QmNnooDu7bfjPFoTZYxMNLWUQJyrVwtbZg5gBMjTezGAJN
Size: 19
CumulativeSize: 77
ChildBlocks: 1
Type: file

$ipfs files stat /hide
QmQCU2EcMqAqQPR2i9bChDtGNJchTbq5TbXJJ16u19uLTa
Size: 0
CumulativeSize: 4
ChildBlocks: 0
Type: directory

$ipfs files read /h
add wechat 18191727
```
