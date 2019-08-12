---
layout: post
title: "IPFS - multistream"
categories:
  - IPFS_Review
tags:
  - IPFS_multistream
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS Multiformats - multistream

multistream 은 multicodec 을 사용하여 자가 기술 함수를 구현합니다. 다음은 자바 스크립트를 기반으로 한 예제입니다. 첫째, 내부에 json 객체인 새로운 버퍼 객체를 작성한 다음 protobuf 접두어를 지정하여 multistream 을 구성하고 네트워크를 통해 전송할 수 있도록 합니다. 구문 분석에서 먼저 코덱 접두어를 취한 다음 접두사를 제거하여 특정 데이터 컨텐츠를 가져올 수 있습니다.

```json
// encode some json
const buf = new Buffer(JSON.stringify({ hello: 'world' }))

const prefixedBuf = multistream.addPrefix('json', str) // prepends multicodec ('json')
console.log(prefixedBuf)
// <Buffer 06 2f 6a 73 6f 6e 2f 7b 22 68 65 6c 6c 6f 22 3a 22 77 6f 72 6c 64 22 7d>

const.log(prefixedBuf.toString('hex'))
// 062f6a736f6e2f7b2268656c6c6f223a22776f726c64227d

// let's get the Codec and then get the data back

const codec = multicodec.getCodec(prefixedBuf)
console.log(codec)
// json

console.log(multistream.rmPrefix(prefixedBuf).toString())
// "{ \"hello\": \"world\" }
```

결과 출력:

```protobuf
hex:   062f6a736f6e2f7b2268656c6c6f223a22776f726c64227d
ascii: /json\n"{\"hello\":\"world\"}"
```

## 1. 멀티 스트림 소스 코드 분석

소스 경로: src\github.com\multiformats\go-multistream\multistream.go

go-multistream 설치

`get github.com/multiformats/go-multistream`

멀티플렉서를 사용하여 사용자가 다른 "프로토콜"을 처리 할 수 있는 핸들러를 추가 할 수 있습니다:

```go
package main

import (
    "fmt"
    "io"
    "io/ioutil"
    "net"
    ms "github.com/multiformats/go-multistream"
)

// 사용되지 않는 프로토콜에 대한 다른 핸들러를 작성하기 위해 멀티플렉서를 작성
// "/cats" 및 "/docs" 두 가지 프로토콜을 만듭니다.
func main() {
    mux := ms.NewMultistreamMuxer()
    mux.AddHandler("/cats", func(proto string, rwc io.ReadWriteCloser) error {
        fmt.Fprintln(rwc, proto, ": HELLO I LIKE CATS")
        return rwc.Close()
    })
    mux.AddHandler("/dogs", func(proto string, rwc io.ReadWriteCloser) error {
        fmt.Fprintln(rwc, proto, ": HELLO I LIKE DOGS")
        return rwc.Close()
    })

    list, err := net.Listen("tcp", ":8765")
    if err != nil {
        panic(err)
    }

    go func() {
        for {
            con, err := list.Accept()
            if err != nil {
                panic(err)
            }

            go mux.Handle(con)
        }
    }()

    // 테스트 시작
    conn, err := net.Dial("tcp", ":8765")
    if err != nil {
        panic(err)
    }

    // /cats 프로토콜
    mstream := ms.NewMSSelect(conn, "/cats")
    cats, err := ioutil.ReadAll(mstream)
    if err != nil {
        panic(err)
    }
    fmt.Printf("%s", cats)
    mstream.Close()

    conn, err = net.Dial("tcp", ":8765")
    if err != nil {
        panic(err)
    }
    defer conn.Close()
    // /dogs 프로토콜
    err = ms.SelectProtoOrFail("/dogs", conn)
    if err != nil {
        panic(err)
    }
    dogs, err := ioutil.ReadAll(conn)
    if err != nil {
        panic(err)
    }
    fmt.Printf("%s", dogs)
    conn.Close()
}
```

확인 테스트는 아래와 같이 수행합니다.

```protobuf
$go run multistream.go
/cats :HELLO I LIKE CATS
/dogs :HELLO I LIKE DOGS
```
