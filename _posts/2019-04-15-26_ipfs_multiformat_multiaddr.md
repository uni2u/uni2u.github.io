---
layout: post
title: "IPFS - multiaddr"
categories:
  - IPFS_Review
tags:
  - IPFS_multiaddr
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS Multiformats - multiaddr

multiaddr 는 자가 기술 네트워크 주소입니다. 현재 네트워크 주소 형식을 개선하는 좋은 방법이며 네트워크를 결합하여 효율적으로 사용할 수 있는 방안으로 생각할 수 있습니다.

현재 주소 지정 체계가 가지고 있는 주요 문제점은 다음과 같습니다.

- 프로토콜 마이그레이션과 프로토콜 간의 상호 운용성을 방해합니다.
- X-over-Y 구조 uri/url 또는 host:port 체계에서만 해결할 수 있습니다.
- 멀티플렉싱을 지원하지 않습니다. 주소 포트가 아니라 프로세스입니다.
- 암묵적으로 out-of-band 값과 컨텍스트를 가정합니다.
- 기계로 읽을 수 있는 효율적인 표현을하지 못합니다.

multiaddr 는 이러한 문제를 개선하기 위해 다음 기능을 제공합니다.

- multiaddrs 는 모든 네트워크 프로토콜의 주소를 지원합니다.
- multiaddrs 는 자가 기술합니다.
- multiaddrs 는 간단한 구문을 따르므로 구문 분석 및 구성이 중요하지 않습니다.
- multiaddrs 는 사람이 읽을 수 있고 효율적인 기계 판독이 가능합니다.
- multiaddrs 는 잘 포장되어있어 캡슐화 레이어의 간단한 패키징 및 전개가 가능합니다.

아래의 두 예를 보겠습니다.

보통 형식:

```
127.0.0.1:9090   # ip4. is this TCP? or UDP? or something else?
[::1]:3217       # ip6. is this TCP? or UDP? or something else?

http://127.0.0.1/baz.jpg
http://foo.com/bar/baz.jpg
//foo.com:1234
 # use DNS, to resolve to either ip4 or ip6, but definitely use
 # tcp after. or maybe quic... >.<
 # these default to TCP port :80.
```

multiaddr 형식:

```
/ip4/127.0.0.1/udp/9090/quic
/ip6/::1/tcp/3217
/ip4/127.0.0.1/tcp/80/http/baz.jpg
/dns4/foo.com/tcp/80/http/bar/baz.jpg
/dns6/foo.com/tcp/443/https
```

multiaddr 을 사용하면 더 명확 해지며 형식이 더 읽기 쉽습니다.

## multiaddr 형식

Multiaddr은 재귀 TLV (Type+Length+Value) 형식으로 인코딩되며 두 형식이 있습니다.

- 인간 친화적 버전 (UTF-8 인코딩 사용)
- 저장 및 전송을 위한 컴퓨터 친화적 버전

인간 친화적 형식은 다음과 같이 정의됩니다.

`(/<addr-protocol-str-code>/<addr-value>) +`

- 경로 기호 중첩 프로토콜 및 주소 (예 : /ip4/127.0.0.1/udp/4023/quic)
- 유형 (addr-protocol-str-code) 은 네트워크 프로토콜을 식별하는 문자열 코드입니다. 프로토콜 테이블은 구성 가능합니다.
- 값은 문자열 형식의 네트워크 주소 값입니다.

컴퓨터 친화적 형식은 다음과 같이 정의됩니다.

`(<addr-protocol-code><addr-value>) +`

- 유형 (addr-protocol-code) 은 네트워크 프로토콜을 식별하는 변수 정수입니다. 프로토콜 테이블은 구성 가능합니다.
- 길이는 바이트 단위의 주소 값 길이를 계산하는 데 사용되는 부호없는 변수 정수입니다.
- 값 (addr-value) 은 네트워크 주소 길이 값입니다.

## multiaddr 설치 및 사용

Golang 을 사용하여 multiaddr 을 사용합니다.
multiaddr 설치는 다음을 통하여 설치합니다.

`go get github.com/multiformats/go-multiaddr`

코드:

```go
package main

import (
    "fmt"

    ma "github.com/multiformats/go-multiaddr"
)

func main() {
    // construct from a string (err signals parse failure)
    m1, err := ma.NewMultiaddr("/ip4/127.0.0.1/udp/1234")
    if err != nil {
        panic(err)
    }
    // construct from bytes (err signals parse failure)
    m2, err := ma.NewMultiaddrBytes(m1.Bytes())
    if err != nil {
        panic(err)
    }

    // true
    fmt.Println(m2.Equal(m1))
    fmt.Println(m1.Protocols())
    m3, err := ma.NewMultiaddr("/sctp/5678")
    if err != nil {
        panic(err)
    }
    fmt.Println(m1.Encapsulate(m3))
    m4, err := ma.NewMultiaddr("/udp/1234")
    if err != nil {
        panic(err)
    }
    fmt.Println(m1.Decapsulate(m4))
}
```

설치 확인 테스트는 아래와 같이 수행합니다.

```
$ go run multiaddr.go
true
[{ip4 4 [4] 32 false {0x4cd930 0x4cdd00 <nil>}} {udp 273 [142 2] 16 false {0x4cdd70 0x4cdf80 <nil>}}]
/ip4/127.0.0.1/udp/1234/sctp/5678
/ip4/127.0.0.1
```

