---
layout: post
title: "opensource serverless-overview"
categories:
  - serverless
tags:
  - serverless
lang: ko
author: "uni2u"
meta: "Springfield"
---

# opensource serverless

## 종류

오픈소스 기반 서버리스 (serverless) 프레임워크는 대표적으로 `apache OpenWhisk`, `OpenFaaS`, `Ignazio Nuclio`, `Platform9 Fission`, `bitnami Kubeless`, `Fn Project` 등이 있다.

이 중 다양한 서비스까지 소개되고 있고 나름 든든한 지원군이 많이 편성된 것이 apache OpenWhisk (IBM Cloud Functions 에 활용) 과 Fn Project (Oracle Fn 에 활용) 이다.

serverless 란 대부분 Event Trigger, Runtime Engine, BackEnd Service 를 구성하고 있으며 docker 또는 Kubernetes 에서 Runtime 및 CLI 를 지원한다.

|            | 지원 platform                                         | 지원 Runtime Engine                               |
| :--------: | :---------------------------------------------------- | :------------------------------------------------ |
| OpenWhisk  | Kubernetes, MESOS, docker compose, OpenShift          | python, go, Java, NodeJS, PHP, Ruby, Scala, Swift |
|  OpenFaaS  | docker, Swarm, Kubernetes                             | anything                                          |
|   Nuclio   | docker, Kubernetes, Azure Container, Google Container | python, go, Java, .NET, NodeJS                    |
|  Fission   | kubernetes                                            | python, go, .NET, NodeJS, PHP                     |
|  Kubeless  | kubernetes                                            | python, go, .NET, NodeJS, Ruby, PHP, Ballerina    |
| FN Project | docker, Swarm, Kubernetes                             | python, go, Java, NodeJS, Ruby                    |

### 1) apache OpenWhisk (https://openwhisk.apache.org/)

![](https://raw.githubusercontent.com/apache/openwhisk/master/docs/images/OpenWhisk_flow_of_processing.png)

위의 그림으로 대표되는 OpenWhisk 는 docker 기반 serverless 플랫폼으로 NginX, CouchDB, Kafka, docker container 로 구성된다. 오픈소스 서버리스 중에서 가장 넓은 지원군을 가지고 있다고 생각된다. 그만큼 넓은 프로젝트로서 필요한 구성 요소들을 선택하여 사용할 수 있다. (https://openwhisk.apache.org/documentation.html#project-structure)

### 2) iguazio Nuclio ([https://nuclio.io](https://nuclio.io/))

![](https://nuclio.io/docs/images/architecture.png)

Nuclio 는 HTTP, MQ, Kafka, Kinesis 등 다양한 Event Trigger, RunTime Engine, Pluggable DataBinder 를 제공하는 특징을 가지고 있다. 그 외에 docker, Swarm, Kubernetes, Raspberry Pi 플랫폼을 지원하며 CLI Tool, UI 를 제공하는 서버리스 플랫폼이다.

### 3) OpenFaaS ([https://www.openfaas.com](https://www.openfaas.com/))

![](https://github.com/openfaas/faas/raw/master/docs/of-layer-overview.png)

OpenFaaS 는 docker, Kubernetes 플랫폼을 지원하고 CLI, Runtime Engine 을 제공하며 Event Trigger 로 API Gateway/Watchdog 을 지원하고, 모니터링을 위한 Prometheus 로 구성된다.

### 4) bitnami Kubeless ([https://kubeless.io](https://kubeless.io/))

Kubeless 는 이름처럼 Kubernetes Less 가 아닌 Kubernetes-Native 서버리스 프레임워크의 의미를 가진다. CLI, UI, Runtime Engine, Event Tigger 로 구성되며 Kubernetes 환경에서 쉽게 설치가 가능하고 K8s 명령어를 그대로 활용할 수 있다.

### 5) Fn Project ([http://www.fnproject.io](http://www.fnproject.io/))

Fn Project 도 다른 프레임워크와 마찬가지로 Trigger 에 해당하는 Fn LB, Runtime Engine 인 Fn Server, 데이터 처리를 위한 DataStore, MQ 등으로 구성되어 있다. 구동 환경은 docker 를 이용하고 CLI, FDK (Function Development Kit) 을 제공한다. 별도로 https://github.com/fnproject 에서도 확인가능하다.

### 6) Platform9 fission ([https://fission.io](https://fission.io/))

Fission 은 Kubernetes-native 서버리스 프레임워크 이다. CLI, HTTP/MQ/Timer 등의 Trigger, Runtime Engine 구성으로 이루어지며 Kubernetes 상에서 pod 로 구동된다.
