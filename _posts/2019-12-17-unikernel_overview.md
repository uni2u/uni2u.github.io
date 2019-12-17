---
layout: post
title: "unikernel-overview"
categories:
  - unikernel
tags:
  - unikernel
lang: ko
author: "uni2u"
meta: "Springfield"

---

# unikernel overview

`docker` 가 세상을 집어삼키듯 거대한 파도가 되었다. `openstack` 으로 인한 `cloud` 세상은 점차 `container` 화 되어가고 있으며 그 정점에 `docker` 가 있다.



그렇다면 다음 파도는 무엇이될 것인가? 수많은 후보들이 저마다 특색과 장점을 말하고 있으나 `unikernel` 에 대하여 이야기 하고자 한다.



## unikernel 이란?

`google` 에서 _unikernel_ 이라는 키워드로 검색을 하면 아마도 'docker 사의 unikernel systems 인수' 라는 기사에 가장 먼저 눈이 갈 것이다. 기술적인 내용으로는 2013년에 발표된 논문 [Unikernels: Library Operating Systems for the Cloud](http://anil.recoil.org/papers/2013-asplos-mirage.pdf) 또는 [Unikernels: Rise of the Virtual Library Operating System](https://queue.acm.org/detail.cfm?id=2566628) 이 매우 유명하다. 



unikernel 을 정의한 내용만 뽑아보면 다음과 같다.



> 특정 application program 을 구동하기 위한 작은 OS, 설정 파일을 하나로 묶은 것



일반적인 application 서비스는 하나의 OS 를 설치한 뒤 그 OS 상에서 복수의 application 또는 service 를 작동시키는 것이 정석이다. unikernel 에서는 복수의 application 또는 service 를 작동시킬 때 OS 와 1 대 1 로 묶여있기 때문에  각 application 또는 service 만 동작시킬 각각의 작은 OS 가 동작한다.



```
+---+   +---+   +---+   +---+          +---+   +---+   +---+   +---+
|app|   |app|   |app|   |app|          |app|   |app|   |app|   |app|
+---+---+---+---+---+---+---+          +---+   +---+   +---+   +---+
|             OS            |          |OS |   |OS |   | OS|   | OS|
+---------------------------+          +---+---+---+---+---+---+---+
|             HW            |          |          Hypervisor       |
+---------------------------+          +---------------------------+
                                       |             HW            |
                                       +---------------------------+
   <일반적 application 및 OS>             <unikernel 상에서 application>
```



unikernel 의 OS 이미지 파일의 크기는 매우 작다. 예를 들면 50 개의 unikernel OS 가 필요한 상황이 되어도 최대 500mb 정도가 될 수 있다. (Hypervisor 영역을 잡아도 1~2GB)



## unikernel 장점

[unikernel](http://unikernel.org/) 공식 페이지를 참조하면 unikernel 의 장점은 4가지로 강조하고 있다.



1. Imporved security
2. Small footprint
3. Highly optimized
4. Fast boot



### 1. Improved security

Small footprint 와 관계가 있다. unikernel 의 OS 이미지 사이즈가 작은 이유는 소스코드 크기 (양) 와 관련있다.



application  만 동작할 수 있는 크기의 작은 OS 를 사용하기 때문이다. 이렇게 되면 외부로 부터 공격받는 부분이 줄어들게 된다. 또한 특정 application 기동을 위한 것으로 보통 OS 레이어에 위치하여 user 라는 개념이 존재하지 않는다. 즉, root 와 같은 슈퍼 유저도 존재하지 않기 때문에 권한에 대한 문제가 없다.



execve() 나 system() 등과 같은 함수도 없기 때문에 CPU 의 NX bit 와 조합해 스택 영역에서 코드 실행을 막으면 Shellcode 와 같은 의도하지 않은 코드 실행이 어려워진다.



하지만 unikernel 실장이 버그에 의해 버퍼 오버플로를 발생시키므로 주의가 필요하다.



### 2. Small footprint

문장 그대로 unikernel 의 OS 이미지가 작다는 것이다.



이것으로 인해



i) 이용 가능 디바이스 종을 제한 (물리 서버 상에서 직접 동작을 중지하고 특정 하이퍼바이저 상에서 동작하게 함)

ii) application 동작에 필요없는 OS 기능을 OS 이미지에 포함시키지 않음 (Linux kernel module 과 같은 동적 OS 구성 변경)



소형 IoT 디바이스의 OS 로서 unikernel 의 사용 가능성이 높다.



### 3. Highly optimized

상호 관계가 깊은 코드 양이 적어서 application 레이어에서 OS 레이어에 걸치 폭넓은 최적화를 적용할 수 있다.



### 4. Fast boot

문장 그대로 기동시간이 매우 짧다. 하이퍼바이저 상에서 움직이는 unikernel 이라고 하여도 BIOS 초기화를 필요로 하지 않는 OS/VM 레이처의 실장이라면 통상의 application 처럼 빠르게 작동된다.



가상 머신에서 격리된 안전한 function 을 동작시켜 FaaS (Function as a Service) 를 실현하기 위한 방법으로 사용될 수 있다.



## unikernel 단점

우선 자료도 매우 부족하다. (매뉴얼, git, 튜토리얼 등의 정보 부족) 또한 하이퍼바이저 상에서 동작시키는 unikernel 의 디버깅과 튜닝이 매우 복잡하다.



## unikernel 종류

application/service 레이어의 언어, 용도, 플랫폼도 서로 다르다. 



### 언어별

[Clive](http://lsub.org/ls/clive.html): Go 언어로 작성된 application 으로 베어메탈 환경에서 동작

[HaLVM](https://galois.com/project/halvm/): Haskell 언어로 작성된 application 으로 Xen 환경에서 동작

[IncludeOS](http://www.includeos.org/): C++ 언어로 작성된 application 으로 KVM 환경에서 동작
[LING](http://erlangonxen.org/): Erlang/OTP 언어로 작성된 application 으로 Xen 환경에서 동작
[MirageOS](https://mirage.io/): OCaml 언어로 작성된 application 으로 베어메탈/Xen/KVM 환경에서 동작
[OSv](http://osv.io/): C/Java/Ruby/Node.js 언어로 작성된 application 으로 Xen/KVM/Virtualbox/VMware 환경에서 동작
[runtime.js](http://runtimejs.org/): Javascript 언어로 작성된 application 으로 KVM 환경에서 동작



### 용도별

[ClickOS](http://cnp.neclab.eu/projects/clickos/): NFV 용도로 사용되며 Xen 환경에서 동작

[Hermitcore](https://hermitcore.org/): HPC 용도로 사용되며 베어메탈/KVM 환경에서 동작



### 호환성

[Rumprun](http://rumpkernel.org/): POSIX application 으로 베어메탈/Xen/KVM 환경에서 동작



### Linux

[UKL](http://www.cs.bu.edu/~jappavoo/Resources/Papers/unikernel-hotos19.pdf): boston university & RedHat 에 의한 Linux 앱을 위한 unikernel

[UniLinux](https://www.youtube.com/watch?v=I_bGUjYGMZY): Linux 와 Ring0 싱글 프로세스에서 동작



### Windows

[Drawbridge](http://research.microsoft.com/en-us/projects/drawbridge/): 윈도우 application, 베어메탈 환경에서 동작



### 주목할 내용

[Unikraft](https://xenproject.org/developers/teams/unikraft/): 라이브러리화 된 기능 모듈을 바탕으로 Konfig 를 이용해 unikernel 작성



자작 application 을 unikernel 로 작성하고 싶다면 언어별 unikernel 을 시험하는 것이 가장 좋다. NFV 혹은 HPC 라는 특정 용도를 목적으로 한다면 ClickOS/Hermitcore 를 사용하는 것이 좋다.



기존의 application 을 unikernel 로 동작하고 싶다면 호환성을 장점으로 가진 Rumprun 이 선택사항이 된다. (이미 [Apache Cassandra](https://github.com/cloudius-systems/osv/wiki/Cassandra-quickstart-on-OSv-with-KVM) 예가 있다)



UKL 은 RedHat 이 관여하고 있는 프로젝트이며 기존 프로그램의 쉬운 unikernel 화 라는 장점을 가지고 있다.



아직 시작한지 얼마되지 않은 단계이지만 논문 제목에도 있듯 '[The Next Stage of Linux's Dominance](https://www.cs.bu.edu/~jappavoo/Resources/Papers/unikernel-hotos19.pdf)' 가 될지 매우 기대된다.



소개된 위의 종류에서 활발한 활동을 보이는 unikernel 이 있는 반면 활동이 거의 없는 unikernel 이 있으니 실제 사용에는 주의가 필요하다. (홈페이지나 github 의 갱신 상태로 추측이 가능하다.)



Microsoft Research 의 Drawbridge 는 그 자체가 공개적으로 available 되지 않지만 [Windows Subsystem for Linux](https://docs.microsoft.com/ja-jp/windows/wsl/about) (Window 상에서 Linux 프로그램 동작) 에서 동작한다. [SQL Server on Linux](https://blogs.msdn.microsoft.com/dataplatjp/2017/03/08/whats-sqlserver-on-linux-2/) (SQL Server 를 Linux 상에서 동작) 에 활용되고 있다.



보안 측면에서 application 레이어에 OCaml (함수형 언어) 이 활발하게 활동하고 있으며 unikernel 은 MirageOS 의 개발 커뮤니티가 활발하게 활동하고 있다.



## unikernel 관련 업체

- [Docker](https://www.docker.com/)
  - unikernel system 매수 업체, 함께 MirageOS 개발
- [Tarides](https://tarides.com/)
  - 프랑스 업체로 docker 사에서 MirageOS 나 MAC/Windows 에서의 docker 등에 특화
  - MirageOS core 개발과 사용 업체를 위한 서포트/개발
- [robur](https://robur.io/Home)
  - MirageOS 를 기반으로 SW 를 개발하고 있는 비영리 조직
- [nanovms](https://nanovms.com/)
  - Deferpanic 이라는 이름으로 설립된 스타트업으로 Unikernel IaaS 플랫폼 제공
- [Zededa](https://www.zededa.com/)
  - IoT 용 플랫폼에 Unikernel 적용 ([sdxcentral](https://www.sdxcentral.com/articles/news/zededa-wants-to-use-the-cloud-to-power-iot-at-the-edge/2018/04/))
- [ARM](https://www.arm.com/)
  - MirageOS 는 ARM 아키텍처에서의 동작을 서포트하고 있거나 Unikernel 전용 가상 머신 모니터인 [hvt](https://github.com/Solo5/solo5/tree/master/tenders/hvt) 의 ARM 프로세서 대응에 ARM 사의 엔지니어가 종사
- [NEC Laboratories Europe](http://sysml.neclab.eu/)
  - ClickOS 나 Unikraft, Uniprof 의 연구
- [EMC](https://www.dellemc.com/)
  - Unik 이라는 Unikernel 개발/관리 툴 개발
- [VMWare](https://research.vmware.com/)
  - 중국 R&D 팀이 UniLinux 소속으로 개발
- [Boston University & RedHat](https://www.bu.edu/rhcollab/2018/11/27/ukl-a-linux-based-unikernel-with-a-community-based-approach-created-via-the-boston-university-red-hat-collaboratory/)
  - Collaboratory 라는 형태로 UKL 의 연구개발
- [SAP](https://www.sap.com/)
  - ERP 나 HANA 로 유명한 기업
  - 클라우드 앱의 콘텍스트에서 [ReasonML](https://reasonml.github.io/) 이라는 언어 (Javascript 와 OCaml 의 좋은 점치기 언어) 개발



## unikernel 정보

unikernel 웹사이트는 [unikernel.org](http://unikernel.org/) 이다. 또, Unikernel 에 대한 논의는 [Unikernel Devel](https://devel.unikernel.org/) 에서 이루어진다.



## Linuxkit

Docker 컨테이너 이미지의 footprint 를 작게 하고 그 컨테이너를 움직이기 위한 작은 LinuxOS 가 포함된 VM 이미지를 작성하는 [Linuxkit](https://github.com/linuxkit/linuxkit) 이라는 툴이 Moby project (특정 용도의 컨테이너를 낭비하지 않고 개발하기 위한 프레임워크) 에서 개발되고 있다.



DockerCon17 에서 Redis 의 컨테이너가 동작하는 VM 이미지를 작성하고 있는 예가 있다. ([SlideShare](https://www.slideshare.net/Docker/dockercon-2017-general-session-day-1-solomon-hykes-75362520))



## 앞으로

다소 생소한 단어이지만 웬지 모두들 생각하고 있었을 unikernel 에 대해 차츰 알아보고자 한다.