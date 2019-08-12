---
layout: post
title: "IPFS - block"
categories:
  - IPFS
tags:
  - IPFS
  - block
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS block

ipfs 네트워크에 업로드 된 파일의 크기가 설정된 크기 (기본값은 256k)를 초과하면 시스템은 파일을 블록으로 분할하여 따로 저장합니다. 블록 명령은 이러한 블록을 조작하는 데 사용됩니다.
블록 명령 형식은 다음과 같습니다:

```protobuf
ipfs block <명령어>
```

다음과 같은 4 개의 하위 명령이 있습니다.

- `get <hash>`: 블록 내용을 검색하고 표시

- `put <file>`: 다음 세 가지 옵션을 사용하여 파일을 ipfs 데이터 블록에 저장
  - format: 새로 생성 된 블록의 cid 형식 지정
  - mhtype: 여러 해시 함수 지정
  - mhlen: 여러 해시 길이 지정

- `rm <hash>`: 블록 삭제, 고정 된 블록을 삭제할 수 없으면 다음과 같은 두 가지 옵션
  - f: 삭제 시 존재하지 않는 블록 무시
  - q: 출력 최소화 알림 메시지

- `stat <hash>`: hash 값 및 블록 크기를 포함하여 블록에 대한 정보 표시

```protobuf
$ipfs block put hello.txt
QmSoLV4Bbm51jM9C4gDYZQ9Cy3U6aXMJDAbzgu2fzaDs64

$ipfs block stat QmSoLV4Bbm51jM9C4gDYZQ9Cy3U6aXMJDAbzgu2fzaDs64
Key: QmSoLV4Bbm51jM9C4gDYZQ9Cy3U6aXMJDAbzgu2fzaDs64
Size: 19

$ipfs block get QmSoLV4Bbm51jM9C4gDYZQ9Cy3U6aXMJDAbzgu2fzaDs64
"hi, every"

$ipfs block rm QmSoLV4Bbm51jM9C4gDYZQ9Cy3U6aXMJDAbzgu2fzaDs64
removed QmSoLV4Bbm51jM9C4gDYZQ9Cy3U6aXMJDAbzgu2fzaDs64
```
