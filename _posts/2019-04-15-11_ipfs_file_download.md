---
layout: post
title: "IPFS - file download"
categories:
  - IPFS_Review
tags:
  - IPFS_file_download
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS 는 어떻게 파일을 다운로드 합니까?

IPFS 는 파일을 로컬 노드로 다운로드 합니다. 이때 사용하는 명령은 `ipfs get` 이며 로컬에 캐시됩니다. `ipfs pin add` 는 다운로드 한 캐시 데이터를 잠그는데 사용됩니다. IPFS 에서 pin 은 파일이 오랫동안 로컬에 저장되고 가비지 수집되지 않으며 pin 에 의해 처리되지 않은 데이터가 정기적으로 정리된다는 것을 의미합니다.

![](https://pocket-image-cache.com/direct?url=http%3A%2F%2Fimage.chaindesk.cn%2FIPFS101.jpg%2Fmark&resize=w1408)

위의 그림은 `ipfs get` 명령의 흐름을 보여줍니다. 현재 노드의 경우 연결된 모든 피어 노드가 swarm 네트워크를 형성합니다. 로컬 노드가 get 요청을하면 로컬 블록 저장소에서 요청 된 데이터를 먼저 찾은 다음 찾지 못하면 swarm 네트워크에 DHT 라우팅을 통해 데이터를 소유 한 노드를 찾는 요청을 보냅니다. 요청 된 데이터가 있는 노드가 발견되면 노드는 데이터를 피드백 합니다. 그런 다음 로컬 노드는 수신 된 블록 데이터를 로컬 블록 저장소에 캐시하여 전체 네트워크가 원본 데이터의 복사본과 동일하게 만듭니다. 더 많은 노드가 데이터를 요청하면 노드의 데이터가 점점 더 많아지기 때문에 데이터가 손실되기가 거의 불가능 해집니다. 이것이 IPFS 네트워크가 영구적으로 데이터를 저장할 수 있는 방법입니다.

`get` 명령을 사용하여 해시가 가르키는 데이터를 얻을 수 있습니다. `cat` 옵션을 통하여 해시의 데이터를 볼 수 있습니다.

```
$ ipfs get QmbrevseVQKf1vsYMsxCscRf6D7S2dftYpHwxkYf94pc7T
Saving file(s) to QmbrevseVQKf1vsYMsxCscRf6D7S2dftYpHwxkYf94pc7T
 17 B / 17 B [====================================] 100.00% 0s

$ ipfs cat QmbrevseVQKf1vsYMsxCscRf6D7S2dftYpHwxkYf94pc7T
IZ*ONE violeta
```

`ipfs pin` 명령어는 다음과 같이 사용된다.

```
$ ipfs pin add QmbrevseVQKf1vsYMsxCscRf6D7S2dftYpHwxkYf94pc7T
pinned QmbrevseVQKf1vsYMsxCscRf6D7S2dftYpHwxkYf94pc7T recursively
```
