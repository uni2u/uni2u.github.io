---
layout: post
title: "A Fog storage software architecture for the Internet of Things"
categories:
  - fog storage
tags:
  - fog storage
lang: ko
author: "uni2u"
meta: "Springfield"
---

원문: [A Fog storage software architecture for the Internet of Things](https://hal.archives-ouvertes.fr/hal-02496105/document)

#### 요약(Abstract)

유럽의 Think Tank IDATE Digiworld 은 2030년 350억 장치가 연결될 것으로 예측했다. 이러한 물결은 data, application 및 service 의 폭발적인 성장을 동반할 것이다. 따라서 low latency, mobility (사용자, infrastructure) 및 네트워크 파티셔닝을 지원하는 Fog architecture를 제안하는 것이 매우 시급하다. 이 논문은 확장성과 성능을 모두 허용하는 object store system 과 확장형 NAS (Network Attached Storage) 를 결합하는 아키텍처를 설명한다. 또한 복제 관리를 위한 DNS (Domain Name Service) 에서 영감을 받은 새로운 프로토콜을 제안한다. 마지막으로 Fog Storage, CDN (Content Delivery Networks) 및 NDN (Named Data Networking) 간의 개념에 대해 설명한다. 또한 프랑스 클러스터인 Grid'5000에 대한 실험 평가가 포함된다.

## 1. Introduction

2012년 시스코에서 제안한 Fog 인프라는 네트워크 edge에 지리적으로 분산된 마이크로 나노 데이터 센터를 배포하는 것으로 구성된다. 각 소규모 데이터 센터는 Fog 사이트로 볼 수 있으며, low latency 기반의 컴퓨팅 및 스토리지 리소스를 제공하는 소수의 서버만 호스팅한다. IoT는 거리 및 전력 관점에서 계층적 토폴로지를 따른다: 클라우드는 네트워크 대기시간 측면에서 가장 긴 대기시간을 가지고 있으면서 가장 큰 컴퓨팅 및 스토리지 기능을 제공한다. Edge 장치는 로컬 컴퓨팅 및 스토리지 리소스의 장점을 가지고 있지만 클라우드에 비해 제한적이다. Fog 사이트는 IT 리소스의 거리와 전력 간의 균형을 제공하는 중간 시설로 볼 수 있다. 또한 Fog 사이트는 사용자 장치와 클라우드 컴퓨팅 센터간 요구를 충족시키기 위해 상호 보완적이다. 프로세싱 및 저장 측면에서 Fog 사이트는 수명이 며칠인 오퍼레이션 데이터를 처리하고 저장하도록 설계되었으며 클라우드 인프라는 몇 달 동안 히스토리를 저장하도록 설계되었다. 즉, 아키텍처는 수직 (클라우드~Edge) 과 수평 (Fog 사이) 을 골고루 갖추고 있다. 이렇게 분산 된 인프라는 대규모 병령 IoT 데이터 스트림을 지원하는데 필요하다. 가장 큰 문제는 어떤 종류의 분산 소프트웨어가 이러한 인프라를 관리하고 무엇을 할 수 있을까?

많은 use cases 가 Fog 인프라의 장점을 누릴 수 있다. 예를들어 Bonomi 등은 connected vehicles 에서 Fog 컴퓨팅을 사용할 것을 제안한다. Fog 는 Thing 으로 부터 메트릭을 수집하고 보행자가 도로에 감지되면 특정 지역의 차량을 정지하는 결정을 내릴 수 있다. 또 다른 use cases 는 Network Function이 사용자의 이동을 따라 움직일 수 있도록 Network Virtualization Functions 을 배포하는 것이다. 이것은 이동 중에 사용자 데이터에 대한 low latency 액세스를 제공하기 위해 버스나 기차에 내장된 Fog 사이트를 이용할 수 있도록 확장되어야 한다. 마지막으로 Fog 컴퓨팅은 인더스트리 4.0에서 사용할 수 있어야 한다. 많은 센서가 데이터를 Fog에 업로드 한 다음 많은 사용자가 이를 처리한다.

그림 1은 이러한 계층 구조와 use cases 일부를 표현하고 있다. 각 사이트는 스토리지 및 컴퓨팅을 제공하는 제한된 서버를 호스팅한다. 최종 사용자 장치 (스마트폰, 테블릿, 랩톱 등) 와 IoT 장치 (센서, 스마트 빌딩 등) 는 10ms (무선 링크 latency) 보다 짧은 low latency (L<sub>Fog</sub>) 로 Fog 사이트에 도달할 수 있다. Fog 사이트 (L<sub>Core</sub>) 간의 latency 시간은 50ms (Wide Area Network link 의 평균 latency) 이다. 클라이언트에서 클라우드 플랫폼 (L<sub>Cloud</sub>) 에 도달하는 latency 시간은 더 높으며 (약 200ms) 예측할 수 없다. 이 논문에서는 데이터 저장이 처리의 전제 조건이라고 주장한다. 따라서 엄청난 수의 IoT 장치에서 생성되는 방대한 양의 데이터를 처리하려면 Fog 계층에 배포 된 효율적인 스토리지 솔루션이 필요하다. 그러나 다중 사이트 및 low latency 에서 동작하도록 설계된 단일 스토리지 솔루션은 현재 존재하지 않으며 이것이 목표이다.

```
+----------------+                           [Cloud Computing]
|Domestic Network|-----------+                        |                                         +-----------+
+--------+-------+           |                 Cloud Latency                              +-----|Edge Device|
         |               +--------+                = 200ms                 +--------+     |     +-----------+
         |               |Fog site|----------+        |         +----------|Fog site|-----+-----|Edge Device|
         |               +--------+          |        |         |          +--------+     |     +-----------+
+--------+---------+         |               |        |         |                         +-----|Edge Device|
|Enterprise Network|---------+               |        |         |                               +-----------+
+------------------+                      +-----------------------+                    Edge to Fog latency
                                          |Inter Micro DCs latency|                         [10-100ms]
+-----------+                             |     [50ms~100ms]      |
|Edge Device|----------------+            +-----------------------+
+-----------+                |               |                  |
                         +--------+          |                  |          +--------+
                         |Mobile  |----------+                  +----------|Micro   |
                         |Fog site|                                        | Nano DC|
                         +--------+                                        +--------+
+-----------+                |              Site of Fog Computing
|Edge Device|----------------+
+-----------+

<Extreme Edge Frontier> | <Edge Frontier>                             <Edge Frontier>
```

이 논문의 목표를 위해 첫번째로 이러한 네트워크에서 데이터를 찾기위한 기존 전략의 장점과 단점을 제시한다. 둘째로 로컬에 저장된 데이터를 보다 효율적으로 찾을 수 있는 로컬 Scale-Out NAS 를 추가하여 기존의 스토리지 솔루션을 개션한다. 마지막으로 분산 네트워크에서 데이터를 효율적으로 찾을 수 있는 새로운 프로토콜을 제안한다. 특히 데이터 (미디어) 가 네트워크 edge 에서 생성되고 다른 사이트와 공유되는 Content Delivery Network 의 시나리오에 중점을 둔다. 이러한 use cases 의 주요 관점은 데이터가 수정되지 않기 때문에 복제는 static replicas 라는 것이다. 이 특성은 consistency 을 고려하지 않을 수 있다는 것이다.

다음 장에서 Fog 인프라에 이상적인 스토리지 솔루션이 무엇인지 알아보고 클라우드 아키텍처 용으로 개발된 기존 솔루션을 사용할 수 없는 이유를 알아보고 개선하기 위한 솔루션을 제시한다.

## 2. Distributed storage for the Fog: an overview


