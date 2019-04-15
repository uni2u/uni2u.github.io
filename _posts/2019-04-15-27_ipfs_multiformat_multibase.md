---
layout: post
title: "IPFS - multibase"
categories:
  - IPFS_Review
tags:
  - IPFS_multibase
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS Multiformats - multibase

multibase 는 데이터를 다른 형식으로 인코딩하는 데 편리한 인코딩 형식을 나타냅니다. 예를 들어 2 진수, 8 진수, 10 진수, 16 진수 및 base58btc, base64 인코딩으로 생각할 수 있습니다.

지원되는 인코딩 형식은 다음과 같습니다.

```go
Multibase Table v1.0.0-RC (semver)

encoding      codes   name
identity      0x00    8-bit binary (encoder and decoder keeps data unmodified)
base1         1       unary tends to be 11111
base2         0       binary has 1 and 0
base8         7       highest char in octal
base10        9       highest char in decimal
base16        F, f    highest char in hex
base32        B, b    rfc4648 - no padding - highest letter
base32pad     C, c    rfc4648 - with padding
base32hex     V, v    rfc4648 - no padding - highest char
base32hexpad  T, t    rfc4648 - with padding
base32z       h       z-base-32 - used by Tahoe-LAFS - highest letter
base58flickr  Z       highest char
base58btc     z       highest char
base64        m       rfc4648 - no padding
base64pad     M       rfc4648 - with padding - MIME encoding
base64url     u       rfc4648 - no padding
base64urlpad  U       rfc4648 - with padding
```

## multibase 형식

`<varint-base-encoding-code><base-encoded-data>`

- varint-base-encoding-code: 인코딩 형식 (위의 내용 참조)
- base-encoded-data: 데이터

예:

```go
4D756C74696261736520697320617765736F6D6521205C6F2F # base16 (hex)
JV2WY5DJMJQXGZJANFZSAYLXMVZW63LFEEQFY3ZP           # base32
YAjKoNbau5KiqmHPmSxYCvn66dA1vLmwbt                 # base58
TXVsdGliYXNlIGlzIGF3ZXNvbWUhIFxvLw==               # base64
F4D756C74696261736520697320617765736F6D6521205C6F2F # base16 F
BJV2WY5DJMJQXGZJANFZSAYLXMVZW63LFEEQFY3ZP           # base32 B
zYAjKoNbau5KiqmHPmSxYCvn66dA1vLmwbt                 # base58 z
MTXVsdGliYXNlIGlzIGF3ZXNvbWUhIFxvLw==               # base64 M
```

시작되는 알파벳은 다음과 같습니다: F, B, z, M

## multibase 설치

Golang 을 사용하여 multibase 를 사용합니다.

multibase 설치는 다음을 사용합니다.

`go get github.com/multiformats/go-multibase`

multiaddr 패키지는 패키지 gx 에 의존하기 때문에 multiaddr 을 사용하려면 다음이 필요합니다.

```go
go get -u github.com/whyrusleeping/gx
go get -u github.com/whyrusleeping/gx-go
cd <your-project-repository>
gx init
gx import github.com/multiformats/go-multibase
gx install --global
gx-go --rewrite
```

## multibase 설치 및 사용

원본 경로: src\github.com\multiformats\go-multibase\multibase.go

기본 인코딩 테이블:

```go
// specified in standard are left out
var Encodings = map[string]Encoding{
    "identity":          0x00,
    "base16":            'f',
    "base16upper":       'F',
    "base32":            'b',
    "base32upper":       'B',
    "base32pad":         'c',
    "base32padupper":    'C',
    "base32hex":         'v',
    "base32hexupper":    'V',
    "base32hexpad":      't',
    "base32hexpadupper": 'T',
    "base58flickr":      'Z',
    "base58btc":         'z',
    "base64":            'm',
    "base64url":         'u',
    "base64pad":         'M',
    "base64urlpad":      'U',
}
```

 코드:
 
```go
// Encode encodes a given byte slice with the selected encoding and returns a
// multibase string (<encoding><base-encoded-string>). It will return
// an error if the selected base is not known.
func Encode(base Encoding, data []byte) (string, error) 
```

설치 확인 테스트는 아래와 같이 수행합니다.

```go
// Decode takes a multibase string and decodes into a bytes buffer.
// It will return an error if the selected base is not known.
func Decode(data string) (Encoding, []byte, error) 
```
