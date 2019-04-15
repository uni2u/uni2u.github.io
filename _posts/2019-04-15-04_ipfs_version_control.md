---
layout: post
title: "IPFS - Version Control"
categories:
  - IPFS_Review
tags:
  - IPFS_Version_Control
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS 버전 관리 시스템

IPFS 의 파일 버전 제어 시스템은 Git 을 기반으로합니다. 먼저 Git 을 분석해 봅시다.

## 버전 제어 시스템 (Git)

git 학습의 개념:

1. 작업 공간: 현재 파일을 편집하는 영역이 작업 영역입니다. 예를 들어 디렉토리가 ./gitworkshop 인 경우 이 디렉토리의 내용이 작업 영역입니다.
2. 저장소: 프로젝트 제출의 전체 상태 및 내용을 기록하십시오. 이는 데이터가 손실되지 않음을 의미합니다.
3. 스테이징 영역: 스테이징 영역이 저장소에 있고 git add 후 데이터가 여기에 저장됩니다. git commit 후에 데이터는 브랜치로 이동되고 스테이징 영역이 지워집니다.
4. 네 가지 주요 객체, 즉 blob, tree, commit 및 refs (또는 태그)를 가지고 있다.
4.1. blob 은 파일의 내용을 보존합니다. git 이 파일을 추가하면 그림 61과 같이 데이터가 저장소의 오브젝트에 저장된다. 이 파일은 방금 추가 한 파일 다음에 생성된 BLOB 이다.
![](https://pocket-image-cache.com/direct?url=http%3A%2F%2Fimage.chaindesk.cn%2FIPFS%25E4%25B8%2580%25E9%2597%25AE%25E4%25B8%2580%25E7%25AD%259414.1.jpg%2Fmark&resize=w1408)
4.2. tree git 의 데이터 저장소는 트리 형식으로 저장됩니다. 아래 그림 참조:
![](https://git-scm.com/book/en/v2/images/data-model-1.png)
4.3. commit 이 실행되면 준비 영역의 내용이 git 저장소에 제출되고 최신 버전이 만들어지고 스냅샷이 만들어집니다. 아래 그림과 같이 최종 단계의 데이터는 마스터 지점에 제출됩니다.
![](https://cdn-images-1.medium.com/max/1600/1*lWD5aMyx3FVX69i6UH13CA.png)
4.4. refs 는 References 의 약자입니다. 참조는 변수 이름이며 값은 코드 변경을 추적하는 데 유용한 커밋 객체를 가리 킵니다. 다음 그림에서 볼 수 있듯이 각 버전마다 고유 한 루트 해시가 있으며 ref 는 HEAD 에 해당하는 루트 해시의 내용입니다.

![](https://git-scm.com/book/en/v2/images/data-model-4.png)

Git 은 변경된 내용으로만 데이터를 다시 저장하며 변경되지 않은 데이터는 처리되지 않습니다. 아래 그림 참조

![](https://whatdoesthequantsay.com/assets/images/ipfs_objects_versioned_filesystem.png)

파일에는 총 3 개가 포함되며 그 중 my_dir 아래의 my_file.txt 는 hello.txt 와 동일하므로 하나의 사본만 저장됩니다. 다음으로 my_dir 폴더에서 my_file.txt 파일을 수정했습니다. 내용이 변경되어 커밋이 이루어지면 새로운 버전의 내용인 새로운 루트 해시가 생성됩니다.

## IPFS 버전 제어 시스템

IPFS 버전 제어는 git 과 거의 같습니다. 아래 그림은 저장소의 내용입니다.

ipfs init 을 실행할 때 ./ipfs 숨김 파일을 생성합니다. 그 내용은 위 그림과 같습니다. 새 파일 내용을 추가 할 때 트리 구조의 저장소가 변경되는 버전, ldb 와 같이 데이터 저장소의 일부 내용이 변경되고 새 데이터 내용이 조각으로 블록에 저장됩니다. 블록의 데이터는 복제되지 않습니다.

각 블록 데이터는 고유 한 해시 값에 해당합니다.

여러 개의 파일과 다른 디렉토리 파일을 포함 할 수있는 디렉토리 파일. 제출 된 후 많은 해시 값을 반환하며, 마지막은 전체 디렉토리 파일의 해시 값, 즉 루트 해시입니다.이 루트 해시는 트리 해시 및 리프 해시를 포함합니다.

IPFS 는 git 처럼 데이터를 수정하거나 데이터를 검색합니다.
