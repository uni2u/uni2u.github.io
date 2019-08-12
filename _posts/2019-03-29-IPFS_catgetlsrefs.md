---
layout: post
title: "IPFS - cat, get, ls, refs"
categories:
  - IPFS
tags:
  - IPFS
  - DHT
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS cat, get, ls, refs


## cat

cat 명령은 ipfs 네트워크에있는 파일의 내용을 표시하는 데 사용됩니다. 형식은 다음과 같습니다:

```protobuf
ipfs cat [options] 파일hash
```

두가지 옵션이 있습니다.

- o int: 표시 할 때 이전의 int 바이트 제거
- l int: 총 int 바이트 표시

다음과 같이 사용됩니다:

```protobuf
$ ipfs cat QmSoLSafTMBsPKadTEgaXctDQVcqN88CNLHXMkTNwMKPnu
"hi, I am IZ*ONE"

$ ipfs cat -o 10 QmSoLSafTMBsPKadTEgaXctDQVcqN88CNLHXMkTNwMKPnu
Z*ONE"

$ ipfs cat -o 1 QmSoLSafTMBsPKadTEgaXctDQVcqN88CNLHXMkTNwMKPnu
hi, I am IZ*ONE"

$ ipfs cat -l 8 QmSoLSafTMBsPKadTEgaXctDQVcqN88CNLHXMkTNwMKPnu
"hi, I am
```

## get

get 명령은 ipfs 네트워크에서 로컬로 파일을 다운로드하는 데 사용됩니다. 형식은 다음과 같습니다:

```protobuf
ipfs get [options] 파일hash
```

네 가지 옵션이 있습니다.

- o path: 로컬 저장 경로
- a: .tar 포맷으로 저장된 압축
- C: .gzip 포맷으로 저장된 압축
- l int: 압축 레벨 지정

## ls

ls 명령은 파일의 각 조각에 대한 정보를 나열하는 데 사용됩니다 (조각이있는 경우). 형식은 다음과 같습니다:

```protobuf
ipfs ls [options] 파일hash
```

주요 옵션은 다음과 같습니다.

- v: 출력에 헤더 추가

```protobuf
$ ipfs add -s size-10240 wechat.jpg
added QmaWLzjB6Y5RdkThm6TCxoK5ZnFRypxTNdK52QLGdoTvAs wechat.jpg

$ ipfs ls QmaWLzjB6Y5RdkThm6TCxoK5ZnFRypxTNdK52QLGdoTvAs
QmY6AWeX52Pemc54pv6jFUvwaA1VBH7yFqLZv6A64MSEWg 10251
QmSCKPtkEpffpKUasSxMqicDN7vbSxhw8k7BwGBFbjSDai 10251
QmQ2DZqTnHF3F9mqtSD66jvKDGUanBgEfUxRDTtMQJuQZp 10251
QmdejejQH5wP6fKrmkBHRqtevFvUrTeAEHpMcnEAGTmkDY 10251
QmWimAR2tjeJBcsBtJPkU3VxQRdieHcYyaK5fnAJCd4Byj 10251
QmUXSHDHLyz3kxsFGdEc29JjLoJoeGamTKvbu84FfZ2VXv 4534

$ ipfs ls -v QmaWLzjB6Y5RdkThm6TCxoK5ZnFRypxTNdK52QLGdoTvAs
Hash                                           Size  Name
QmY6AWeX52Pemc54pv6jFUvwaA1VBH7yFqLZv6A64MSEWg 10251
QmSCKPtkEpffpKUasSxMqicDN7vbSxhw8k7BwGBFbjSDai 10251
QmQ2DZqTnHF3F9mqtSD66jvKDGUanBgEfUxRDTtMQJuQZp 10251
QmdejejQH5wP6fKrmkBHRqtevFvUrTeAEHpMcnEAGTmkDY 10251
QmWimAR2tjeJBcsBtJPkU3VxQRdieHcYyaK5fnAJCd4Byj 10251
QmUXSHDHLyz3kxsFGdEc29JjLoJoeGamTKvbu84FfZ2VXv 4534
```

## refs

refs 명령은 파일의 관련 조각을 나열하는 데 사용됩니다. 형식은 다음과 같습니다:

```protobuf
ipfs refs [options] 파일hash
```

네 가지 옵션이 있습니다.

- format: 출력 형식 지정 (기본값은 샤드만 출력)
- e: 출력 포맷은 소스파일 -> 분할된 포맷
- u: 출력 결과 오프셋
- r: 하위 샤드도 나열
