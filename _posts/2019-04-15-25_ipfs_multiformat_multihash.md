---
layout: post
title: "IPFS - multihash"
categories:
  - IPFS_Review
tags:
  - IPFS_multihash
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS Multiformats - multihash

기존의 암호화 시스템에서 시스템은 항상 정보를 처리하기 위해 해시 함수 또는 암호화 함수를 잠궈야 하고 처리 된 데이터는 고정된 길이 형식입니다. 시스템의 많은 도구 또는 방법은이 암호화 시스템에 의존합니다. 이런 경우 나중에 암호화 시스템을 업그레이드하는 경우 (예: 암호화 기능 유형 교체) 전체 프로그램에 악몽같은 일이 발생합니다. 모든 관련 도구 또는 방법을 업그레이드 하여야 하며 이는 새로운 해시 함수와 새로운 해시 길이를 사용하는 것으로 심각한 상호 운용성 문제에 직면하거나 오류가 발생할 수 있습니다.

이 문제에 대한 해결책으로 Protocol Lab 은 multihash 모델을 개발했습니다.

## 1. multihash 형식

멀티 해시는 TLV (Type-Length-Value) 형식으로 기술됩니다.

- hash-function-type: 서명되지 않은 varint 로 기술됩니다. 테이블을 사용하여 구성하십시오.
- digest-length: 다이제스트의 길이는 부호없는 varints 로 계산됩니다.
- digest-value: 다이제스트 길이를 갖는 해시 함수의 요약을 나타냅니다.

멀티 해시 예)

![](https://raw.githubusercontent.com/multiformats/multiformats/eb22cd807db692877a9094b5bfb4d2997fd0278a/img/multihash.006.jpg)

sha2-sha256 해시 함수를 사용하여 "hello" 문자열을 해시하고 16 진수 문자열로 출력 하는 경우 다음과 같습니다.

```go
var a = "hello"

b := sha256.Sum256([]byte(a))
fmt.Printf("%x",b)

출력：2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
```

그런 다음 multihash 처리 후 다음을 얻습니다.

```protobuf
Multihash： 12202cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824

# 의미는 다음과 같습니다.
- 해시 함수: sha2-256 (0x12)
- 길이: 32 (0x20)
- 초록: 41dd7b6443542e75701aa98a0c235951a28a0d851b11564d20022ab11d2589a8
```

해시 함수가 sha2-512 인 경우:

```protobuf
Multihash : 132052eb4dd19f1ec522859e12d89706156570f8fbab1824870bc6f8c7d235eef5f4

- 해시 함수: sha2-512 (16 진수 코드: 0x13)
- 길이: 32 (16 진수: 0x20)
- 초록: 52eb4dd19f1ec522859e12d89706156570f8fbab1824870bc6f8c7d235eef5f4
```

## 2. multihash 설치 및 소스 코드

Golang 을 사용하여 multihash 를 사용합니다.

multihash 설치는 다음과 같습니다.

`go get github.com/multiformats/go-multihash`

코드는 다음과 같습니다.

```go
package main

import (
    "encoding/hex"
    "fmt"

    "github.com/multiformats/go-multihash"
)

func main() {
    // 문자열을 바이트 배열로 변환
    buf, _ := hex.DecodeString("0beec7b5ea3f0fdbc95d0dd47f3c5bc275da8a33")
    // multihash 로 바이트 배열 인코딩
    mHashBuf, _ := multihash.EncodeName(buf, "sha1")
    // 코드화 요약 출력
    fmt.Printf("hex: %s\n", hex.EncodeToString(mHashBuf))
    // binary multihash 해시를 DecodedMultihash의 형식으로 변환
    mHash, _ := multihash.Decode(mHashBuf)
    // 디지트 된 요약 보기
    sha1hex := hex.EncodeToString(mHash.Digest)
    // 출력
    fmt.Printf("obj: %v 0x%x %d %s\n", mHash.Name, mHash.Code, mHash.Length, sha1hex)
}
```

설치 확인 테스트는 아래와 같이 수행합니다.

```protobuf
$ go run multihash.go
hex:11140beec7b5ea3f0fdbc95dodd47f3c5bc275da8a33
obj:sha1 0x11 20 0beec7b5ea3f0fdbc95d0dd47f3c5bc275da8a33
```

