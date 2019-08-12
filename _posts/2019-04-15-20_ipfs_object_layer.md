---
layout: post
title: "IPFS - object"
categories:
  - IPFS_Review
tags:
  - IPFS_object
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS 기본 - Object 계층

IPFS 객체 계층은 저장소 계층입니다. Git 에서 사용하는 저장소 데이터 구조를 기반으로 IPFS 는 MerkleDAG 기술을 사용하여 객체 데이터 (일반적으로 Base58 로 인코딩 된 해시 참조)를 저장하기 위한 DAG 데이터 구조를 구성합니다. 이 데이터 구조의 특성은 다음과 같습니다:

- Content Addressable: 모든 콘텐츠가 다중 해시되고 고유하게 식별
- 변조 방지: 모든 컨텐츠가 해시되며 데이터가 변조되면 IPFS 네트워크가 감지
- 중복 제거: 동일한 내용은 동일한 해시를 가지며 한 번만 저장되므로 인덱싱 객체에 특히 유용

IPFS 객체의 구조는 다음과 같습니다:

```go
type IPFSLink struct {
    Name string     // 이 링크의 별칭
    Hash Multihash  // 타겟 해시
    Size int        // 총 타겟 크기
}

type IPFSObject struct {
    links []IPFSLink    // 링크 배열
    data []byte         // 불투명 콘텐츠 데이터
}
```

`ipfs object links <객체 hash>` 를 실행하면 그 객체 아래의 모든 IPFSLink 엔트리가 나열됩니다. 즉, IPFSLink 는 객체를 구성합니다.

```tex
$ ipfs object links QmZV47ZKbJ1DCLoHWHGhXZ4EMqsyTsZRJUsZuRwEAoiesn

QmUEHn7UnFyFMpJwkvotwuejnMX8XEkqxNsXMeDXJtiJyf 8019571 IZONE_violeta.mp4
QmaFN5uLVBujrsQ6HFzToxUAbtV9z7MfvCtmLrscS9npf3 52173   IZONE_LaViAnRose.jpeg
QmXd18A2gF1rDbNntsDTX48jexiytU7Wi3dT1rWcqNKjeV 71      IZONE
```

출력의 구조 타입은 `<object multihash> <object size> <link name>` 으로 표현되는 IPFSLink 의 구조 내용입니다.

## 1. 경로

IPFS 대상은 하나의 문자열에 대한 경로를 쿼리할 수 있으며 경로는 전통적인 UNIX 파일 시스템에 있는 경로와 마찬가지로 MerkleDAG 의 links 로 로깅을 단순화 합니다.

```tex
# 형식
/ipfs/<hash-of-object>/<name-path-to-object>
# 사용예
/ipfs/XLYkgq61DyaQ8Nhkcqyu7rcnSa7dSHQ16x/foo.txt
```

IPFS 개체는 다중 경로도 지원합니다.

```tex
/ipfs/<hash-of-foo>/bar/baz
/ipfs/<hash-of-bar>/baz
/ipfs/<hash-of-baz>
```

## 2. 로컬 객체

IPFS 노드에는 하드 디스크 저장소 또는 데이터를 저장하는 블록을 캐시하기 위한 로컬 저장소가 있습니다. 예) 데이터를 업로드 할 때 블록 데이터는 저장소의 하드 디스크 공간에 저장됩니다. 네트워크에서 일부 데이터를 다운로드하는 경우 블록 데이터가 저장소에 캐시됩니다.

## 3. 객체 잠금

일부 오브젝트의 데이터는 저장소에 캐시됩니다. IPFS 는 `pin` 옵션으로 로컬에 데이터를 영구 저장하며 관련 파생 오브젝트를 모두 재귀적으로 잠글 수 있습니다. 이것은 특히 완전한 객체 파일을 장기간 저장하는 데 유용합니다.

## 4. 객체 게시

IPFS 는 전 세계적으로 분산 된 파일 시스템으로 DHT 는 콘텐츠 해싱 기술을 사용하여 게시할 객체를 공정하고 안전하게 분산시킵니다. 누구나 객체를 게시 할 수 있으며 객체의 키를 DHT 에 추가하면 객체가 P2P 전송에 의해 추가 된 다음 다른 사용자의 액세스 경로로 추가됩니다.

## 5. 객체 레벨 암호화

IPFS 객체를 먼저 암호화하여 새 객체를 만든 다음 새 객체를 저장할 수 있습니다. 암호화 된 객체의 구조는 다음과 같습니다:

```go
type EncryptedObject struct {
    Object []bytes              // 암호화 된 원시 객체 데이터
    Tag []bytes                 // 선택적 암호화 식별자
    type SignedObject struct {
        Object []bytes          // 서명 된 원시 객체 데이터
        Signature []bytes       // HMAC 서명
        PublicKey   []multihash // 다중 해시 식별 키
    }
}
```

이것이 블록 체인 스토리지에 적용되는 이유입니다. 블록 체인은 공개 키로 객체를 암호화하여 새 객체를 얻은 다음 객체를 IPFS 네트워크에 저장하여 IPFS 해시 값을 얻습니다. 블록 체인은 해시 값만 저장합니다. 데이터를 얻을 때 먼저 IPFS 해시 값을 사용하여 암호화 된 객체를 가져온 다음 실제 데이터 객체를 얻기 위해 자체 개인 키로 객체를 해독합니다.
