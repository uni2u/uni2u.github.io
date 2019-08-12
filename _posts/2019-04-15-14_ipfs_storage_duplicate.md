---
layout: post
title: "IPFS - storage duplicate"
categories:
  - IPFS_Review
tags:
  - IPFS_storage_duplicate
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS 데이터 저장소가 중복되면?

IPFS 네트워크에서 데이터 저장소는 중복 될 수 있습니다. 이것은 IPFS 중복 제거와 다소 상반됩니다.

데이터는 IPFS 스토리지에 블록으로 저장됩니다. IPFS 에서 데이터를 분할하는 방법은 여러 가지가 있습니다. 분할 방법은 IPFS 소스 코드 `core/commands/add.go` 코드에 설명되어 있습니다.

```protobuf
The chunker option, '-s', specifies the chunking strategy that dictates
how to break files into blocks. Blocks with same content can
be deduplicated. The default is a fixed block size of
256 * 1024 bytes, 'size-262144'. Alternatively, you can use the
rabin chunker for content defined chunking by specifying
rabin-[min]-[avg]-[max] (where min/avg/max refer to the resulting
chunk sizes). Using other chunking strategies will produce
different hashes for the same file.

  > ipfs add --chunker=size-2048 ipfs-logo.svg
  added QmafrLBfzRLV4XSH1XcaMMeaXEUhDJjmtDfsYU95TrWG87 ipfs-logo.svg
  > ipfs add --chunker=rabin-512-1024-2048 ipfs-logo.svg
  added Qmf1hDN65tR55Ubh2RN1FPxr69xq3giVBz1KApsresY8Gn ipfs-logo.svg
```

위의 내용은 3 가지 방법으로 분할할 수 있는 것을 설명하고 있습니다.

1. 기본 모드에서 블록의 크기는 256kb, 즉 256 * 1024 바이트 크기 = 262144 에 해당 합니다. 이 명령은 매개 변수를 추가할 필요가 없습니다. (예: `ipfs add ipfs-logo.svg`)
2. 블록 크기 모드를 지정합니다. 명령은 `ipfs add --chunker = size-1000` 입니다. 뒤쪽의 1000은 262144 미만의 숫자 일 수 있습니다.
3. Rabin 가변 블록 크기 분할 모드로 명령은 `ipfs add --chunker = rabin-[min]-[avg]-[max] <file>` 입니다. min, avg 및 max 값은 각각 최소 블록 크기, 평균 블록 크기 및 최대 블록 크기이며 값은 262144 미만으로 설정됩니다. (예: `ipfs add --chunker = rabin-512-1024-2048 ipfs-logo.svg`)

위의 세 가지 방법은 여러 블록의 크기를 확장 할 수 있습니다. 즉, IPFS 에 동일한 파일이 저장되지만 저장 방법이 달라 반환되는 해시값이 다릅니다.

따라서 IPFS 의 블록 저장소는 중복되지 않습니다. IPFS 블록 파일의 패치 된 데이터가 복제 될 수 있습니다. 즉, 같은 파일을 다른 파일 분할 방법에 따라 IPFS 네트워크에 여러번 반복적으로 저장할 수 있습니다.

IPFS 관리는 IPFS 데이터 저장소가 중복되지 않기 때문에 시스템의 블록 데이터가 복제되지 않으며 블록 파일에 의해 함께 연결된 데이터를 반복 할 수 있습니다.

그러나 모든 사람들이 기본 분할 모드인  blocksize = 256kb 를 사용하는 것을 권고합니다. 따라서 IPFS 의 네트워크에 기존 콘텐츠를 저장할 때 IPFS 는 데이터가 네트워크 이전에 저장되었다는 것을 빠르게 알려줍니다.

같은 데이터가 네트워크에 저장되어 있다고 가정하면 전체 네트워크가 저장할 하드 디스크 저장 공간의 양은 다음과 같이 생각할 수 있다. 매우 인기있는 영화를 가지고 있다면 모든 사람들이 컴퓨터의 디스크나 다른 하드 디스크 저장 장치에 영화를 저장합니다. 1억 명의 전세계 사람들이 영화를 저장하면
이것은 큰 저장 낭비가 될 것 입니다. IPFS 네트워크에서 동영상은 하나의 노드에만 저장되며 사용자가 이를 읽어야 할 때 새로운 백업이 생성됩니다. 누구든지 데이터를 사용하면 데이터가 누구에게 복사됩니다. 이렇게하면 많은 하드 디스크 저장 공간을 절약 할 수 있습니다.

노드가 IPFS 네트워크에 연결되면 노드는 전체 네트워크에서 사용할 하드 디스크 공간 (기본값 10G) 을 제공 합니다. 정상적인 상황에서 파일을 저장할 때 하드 디스크 공간의 이 부분은 네트워크를 통과할 필요가 없으므로 가장 빠릅니다. 저장소가 완료되면 네트워크의 모든 노드가 파일에 액세스할 수 있습니다. 다른 노드가 액세스 하는 경우 해당 노드는 종종 데이터 복사본을 자신의 캐시 공간에 복사 합니다. 이러한 방식으로 전체 네트워크에 두 개의 복사본이 있습니다. 이 파일에 관심이 많은 사람들이 있을 때, 네트워크에 있는 사본의 수는 늘어날 것 입니다.

복사본은 일반적으로 캐시에 저장됩니다. 즉, 임시로 저장되며 오랜 시간이 지나면 자동으로 삭제됩니다. 인기 데이터일 수록 복사본이 많아 지지만 인기도가 떨어질 수록 복사본 수가 줄어들어 자연스럽게 공간 활용도와 액세스 효율성이 균형을 이루게 됩니다.
