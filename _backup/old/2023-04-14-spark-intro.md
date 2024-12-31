---
title: Spark를 시작하기 앞서
author: guaqamole
date: 2023-04-14 18:32:00 +0900
categories: [Data, Spark]
tags: [Data Engineering, Spark, UC Berkly, Distributed Computing, Hadoop]
image:
  path: /common/spark.png
---

그동안 정말 하고싶었지만 시간이 안돼 미루고 미뤘던 스파크 스터디를 정식으로 시작하려고 한다! 

스터디의 목표는 확실하게 & 꾸준하게 스파크를 공부하는것인데, 

놓치고 넘어가는 개념이 없도록 하나하나 짚고 넘어갈 예정이다.

<br>



****

# 스파크의 시작

스파크는 UC 버클리의 RAD 연구실의 연구 프로젝트로 2009년에 시작됐다. 정확히는 AMPLab 소속인 마테이 자라이아, 모샤라프 카우두리, 마이클 플랭클린, 스콧 쉔커, 이온 스토이카가 발표한 논문 "Spark: Cluster Computing with Working Sets"를 통해 처음 세상에 알려졌다. 

당시 하둡의 맵리듀스는 수천 개의 노드로 구성된 클러스터에서 병렬로 데이터를 처리하는 최초의 오픈소스 시스템이자 클러스터 환경용 병렬 프로그래밍 엔진의 대표주자였다. 다만 AMPLab 연구원들은 전통적인 머신러닝 알고리즘을 맵리듀스로 처리하려면 매번 데이터를 처음부터 읽어야 하는 문제에 직면했었다.

그들은 이런 문제를 해결하기 위해 먼저 여러 단계로 이루어진 애플리케이션을 간결하게 개발할 수 있는 함수형 프로그래밍 기반의 API를 설계했다. 그리고 연산 단계 사이에서 메모리에 저장된 데이터를 효율적으로 공유할 수 있는 새로운 엔진 기반의 API를 구현했다. 이어서 스파크 팀은 UC버클리 대학교와 외부 사용자를 대상으로 시스템에 대한 실험을 시작했고, 끝내 Spark 엔진을 개발하게 되었다.

2009년에 탄생한 직후, 여러 학술 콘퍼런스에서 스파크에 관한 연구 논문들이 발표되었으며, 이때도 이미 특정 작업들에 대해서는 맵리듀스보다 10~20배는 빨랐다고 하네요. 스파크는 2010년 3월에 처음으로 오픈 소스로 전환되었고, 이후 3년뒤인 2013년 6월에 공식적으로 아파치 소프트웨어 재단(Apache Software Foundation)에 이관되었다. 

벌써 10년뒤인 지금은 스파크가 아파치 재단의 [최상위 레벨 프로젝트](https://en.wikipedia.org/wiki/List_of_Apache_Software_Foundation_projects)가 된것을 확인 할 수 있다.

<br>

# 하둡과 맵리듀스, 그리고 스파크와의 관계

보통 하둡과 스파크를 비교 할 때 처리속도로 비교를 하게 되는데 엄연히 말하자면 하둡과 스파크는 서로 **사용목적이 다르기 때문에 비교대상이 아니라고 생각한다.**

## 하둡

> "**scalable** & **distributed**"

하둡은 **저장 기술인 HDFS**와 **분산처리 기술인 맵리듀스**(MapReduce)를 제공함으로서 *대용량 데이터를 분산 처리할 수 있는 자바 기반의 오픈소스 프레임워크*로 불리고 있다.

비교적 대용량 데이터를 많이 다루는 큰 기업을 예시로 들어볼겠다. 하드디스크의 입출력 속도는 보통 100MB/s 정도이지만 우리가 평소에 다루는 데이터는 크지 않아서, 데이터를 저장하고 처리하는데 큰 문제가 없다. 

하지만 기업환경에서 마주하는 데이터는 최소 기가바이트(GB) 에서 테라바이트(TB), 페타바이트 (PB) 까지 이르는 경우가 많기 때문에, 데이터를 읽고 쓰는데 여러가지 문제에 직면하게 된다. 따라서:

- 하드 디스크는 1TB짜리 인데, 3TB 데이터를 저장하고 싶을 때
- 100 개의 파일에 대해 똑같은 작업 수행 후 결과를 합치는 분산 컴퓨팅을 하고자 할 때 
- TB 단위 이상의 데이터를 빠르게 Disk I/O (로드 또는 저장) 하고 싶을 때 

하둡이 유용하게 쓰일 수 있을거다. 또한 하둡은 자원을 추가하더라도 코드의 수정 등을 할 필요 없이 동일한 방법으로 프로세싱을 할 수 있기 때문에 확장성에 용이해서 대규묘 데이터를 저장해야하는 기업입장에서는 하둡을 선호 할 수 밖에 없다.

<br>

## Map-Reduce

![](/230314/3.png){: width="400" height="300" }

맵리듀스는 **프로그래밍 모델임과 동시에 구현체를 부르는 말**로, 그 구현체는 '분산처리엔진' 역할을 하는 하둡의 중심 모듈 중 하나이다. 

1. 사용자는 **키/값 쌍을 처리하는 map 함수를 설정하여 중간 결과물 형태의 키/값 쌍 데이터를 만들고**

2. reduce 함수를 설정하여 앞선 **중간 결과물의 키 맵핑을 통해**

3. **같은 키를 가진 값들을 합쳐서 최종 결과물을 만든다**.

이렇게 함수형 스타일로 작성된 프로그램은 자동적으로 병렬화되어 처리되며, 일반적인 상용 기기에서(특별한 고성능 기기가 아닌) 대용량으로 처리를 가능하게 한다. 그렇기에 이러한 프로그램을 사용하여 분산병렬 애플리케이션을 손쉽게 개발하고 사용할 수 있게 한다. 

MapReduce는 Google이 관리해야하는 큰 규모의 데이터와 Borg라는 클러스터 매니저로 부터 발생하는 문제를 해결하기 위해 설계되었다. Google에서 이 문제를 해결하는 과정에서 두 가지 어려움이 있었다:

- 방대한 데이터 세트 - 당시에는 수십 또는 수백 테라바이트의 데이터를 관리했지만, 지금은 아마 페타바이트 이상이 될 거다. 당시 전체 시스템 호스트의 메모리의 합보다 처리해야할 데이터의 용량이 더 컸다고 하니 문제가 될 수 밖에 없었다.

- 워커 노드의 다운타임 - Googled은 중요한 데이터가 손실되는것을 막기위해 노력을 많이했다. 그래서 탄력적인 분산 스토리지(예: GFS)에 데이터의 영구 복사본을 유지하려고 했고 노드의 고가용성 확보가 필요했다.

위 문제점들을 해결하기 위해 바로 MapReduce가 등장한것이다.

<br>

## YARN

하둡 초기 버전인 Hadoop 1.0의 MRV1(MapReduce Version1)는 작업의 처리와 자원의 관리를 한번에 관리했다. 즉, Single-Master 노드에 해당하는 job Tracker는 자원을 할당하고, 스케줄링 작업도 수행하며 처리중인 작업까지 모니터링을 수행했다. 그리고 하위 노드에 해당하는 Task Tracker에 Map and Reduce 작업을 부여하였고, 하위 노드들은 주기적으로 그들의 진행상황을 Job Tracker에 보고하는 체계였다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FnkX0w%2FbtqVkLoi1Pc%2Fivmzlm8JIkkUfIk1RXIPc1%2Fimg.png)

이러한 MRV1구조는 Task의 규모가 커짐에 따라, 하나 뿐인 Job Tracker에 부하가 걸리며 bottleneck이 발생할 뿐만 아니라, 컴퓨터의 자원이 비효율적으로 사용된다는 것을 알게 되었다. 



바로 이러한 문제를 해결하기 위해 Hadoop 2.0부터 YARN(Yet Another Resource Negotiator)이 추가된것이다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdY5446%2FbtqVuhsIGCO%2FrJHnz0lXFIPYfDftv8Umc1%2Fimg.png)

자원관리 및 작업 스케줄링에 대한 책임을 Yarn에 인수하여 MapReduce가 경량화되었으며, MapReduce 이외의 작업을 Hadoop이 처리할 수 있게 해주었다. 그래서 Hadoop 2.0은 위와 같은 구조로 재탄생하게 되었고. Yarn은 HDFS에 저장된 데이터를 바탕으로 그래프 처리, 대화식 처리, 스트림 처리 및 일괄 처리와 같은 일들을 수행할 수 있게 했다. 또한 YARN의 등장으로 실시간 처리를 위한 Spark, SQL 전용의 Hive, NoSQL 전용의 HBase등 다양한 도구 또한 사용할 수 있게 되었다.

<br>

## YARN의 구조

위에서 언급하였듯이 Yarn이 탄생하게 된 근본적인 아이디어는 Resource Management 기능과 Job Scheduling/Monitoring 기능을 분리된 데몬에서 처리하자는거다. 이를 구현하기 위해서 Global Resource Manager(RM)와 application 단위로 존재하는 Application Master(AM) 을 가지도록 하였다. 여기서 말하는 Application은 하나의 작업 또는 작업의 DAG(Directed Acyclic Graph, 비순환 방향 그래프)을 의미한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbwXQ6k%2FbtqVvODO28B%2FkK0yROyby7cOgaY4rkqlFk%2Fimg.png)

### Resource Manager

> Master Daemon에서 구동되며, 클러스터들로의 자원 할당을 관리한다.

Resource Manager는 Resource Allocation에 대한 최상위 권위자로, 어디에 자원을 할당할지 결정하여 Cluster들의 활용을 최적화시킨다. 또한 일에 대한 처리 요청을 받으면, Request들의 일부를 해당하는 Node Manager로 전달한다. 이러한 역할을 수행하는 Resource Manager는 크게 2가지로 구성되는데, 바로 Scheduler와 Applications Manager이다.

**1) Scheduler**

Scheduler는 Capacities(용량), Queue(대기줄) 등의 제한조건에 따라 실행중인 다양한 Applications들에 자원을 할당한다. Resource Manager의 Scheduler는 Application의 상태를 추적하거나 모니터링하지 않는, 순수한 Scheduler이다. 또한 순수한 스케줄러이기 때문에 Hardware failures나 Application failures에 의한 재시작 역시 보장하지 않으며, Application들의 자원 요구에 기반하여 Scheduling 기능을 수행한다. 이것은 메모리, CPU, 디스크, 네트워크와 같은 요소들을 포함하는 (resource) Container를 기반으로 작동한다.

또한 Scheduler는 Plug-in Policy를 사용하는데, 현재는 Capacity Scheduler와 Fair Scheduler를 플러그인으로 사용한다. Capacity Scheduler는 Larger Cluster를 여러 사용자가 함께 사용하는 Multi-tenancy를 구현하여 할당된 용량의 제한하에 적시에 Application들에 응용을 할당할 수 있게 해준다. Fair Scheduler는 YARN으로 하여금 Larger Cluster들이 공평하게 자원을 공유할 수 있게 해준다.

**2) Applications Manager**

Applications Manager는 Job의 제출을 수락하고, Application Master를 실행하기 위한 첫번째 컨테이너를 설정하고 실패 시 Application Master의 Container를 재시작하게 해준다. Application Master는 Scheduler를 통해 적합한 Container를 선정하고, 그것을 Tracking하고 진행 상황을 모니터링하는 역할을 수행한다.



### Node Manager

> Slave Daemon에서 구동되며, 각 단일 노드의 Task 실행을 담당한다.

Node Manager는 개별 노드들을 관리하고 주어진 노드에서 사용자의 작업 및 Work Flow를 관리한다. Node Manger는 Resource Manager에 등록되며 노드의 상태를 판별하기 위한 heartbeats를 전송한다. 그 중에서 Node Manager의 가장 중요한 임무는 Resource Manager을 통해 할당받은 Container를 관리하는 것이다. Container의 자원 사용량을 감시하고, 이를 Resource Manager로 보내는데, 문제가 발생한 경우 Resource Manager로부터 Container를 Kill하라는 명령을 받아 수행하기도 한다.

### Application Master

> 개별적인 응용에서 필요로 하는 자원과 Job에 대한 lifecycle을 관리한다. 이것은 Node Manager와 함께 작동하며, 작업의 실행을 모니터링한다.

여기서 말하는 Application은 프레임워크에 제출된 단일 작업을 의미하며, 각각의 단일 작업은 고유한 Application Master를 지니게 된다. Application Master는 클러스터에서 응용의 실행을 조정하고 오류를 관리한다. 또한 Node Manager와 함께 작동하여 Task를 실행시키고 모니터링하며 Resource Manger를 통해 자원에 대해 협상한다. 자원을 협상한 후에는 Resource Manager로부터 적당한 Resource(conatiner)를 할당받고 그것들의 상태를 추적하고 모니터링한다. 한번 Application Master가 실행되면 주기적으로 Resource Manager에 heartbeats를 전송하여 상태를 확인시켜주고, 자원의 요구사항을 갱신한다.

### Container

> 한 노드의 자원을 모아둔 패키지 (RAM, CPU, Network, HDD 등)

Container는 하나의 노드에 할당받은 자원들로, Container Life Cycle(CLC)에 해당하는 Container Launch Context를 통해 관리된다. 레코드에는 환경 변수 맵, 원격 액세스 가능한 스토리지에 저장된 종속성, 보안 토큰, 노드 관리자 서비스의 페이로드 및 프로세스를 만드는 데 필요한 명령이 포함된다. 또한 Container는 특정 호스트에서 특정 양의 자원 (메모리, CPU 등)을 사용하도록 응용 프로그램에 권한을 부여한다.

<br>

## YARN의 동작 방식

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbgaZp1%2FbtqVuhsJBp5%2FD6HMNGVPIYnYJkJu4g1Zf1%2Fimg.png)

1. Client가 Resource Manager에게 Application에 대한 요청을 전송한다.
2. Resource Manager는 Application Master와 함께 Application을 등록한다. (이때 Application ID가 생성되며, 후에 Client에게 반환된다.)
3. Resource Manager는 각각의 분리된 Container에서 Application Master를 구동한다. 만약 Container가 구동 불가능하다면, 적합한 Container를 찾을 때까지 기다린다.
4. Application Master는 Node Manager에게 Container의 실행 명령을 전달한다.
5. Application 코드가 Container에서 실행된다.
6. Client는 Application의 상태를 모니터링하기 위해 Resource Manager/Application Manager와 주고 받는다.
7. Application Master가 Resource Manager에서 등록 해지된다.

### YARN의 Reservation System

YARN은 Reservation System을 통해 Resource Reservation을 지원하는데, 이것은 사용자가 마감일과 같은 시간적 제약사항이 있는 경우에, **자원을 예약하여 중요한 작업의 실행을 보장한다**. Reservation System은 시간을 두고 자원을 추적하고, 예약에 대한 승인 제어 기능을 수행하며, 예약이 가득 채워져 있는지 확인하기 위해 Scheduler에 동적으로 지시한다.

수천개 노드 이상으로 YARN을 확장하기 위해 YARN Federation 이라는 개념이 존재한다. Federation은 Transparent하게 여러 개의 Yarn Cluster를 묶어, 하나의 거대한 Cluster처럼 보이게한다. 이것은 여러 개의 독립된 Cluster들이 거대한 일을 처리하기 위해 사용되도록 한다.

<br>

## 작업 속도의 문제

하둡은 "*데이터를 읽고 처리하는 속도가 데이터의 양을 따라잡지 못한다*" 라는 문제에서 출발했다.

![](https://qph.cf2.quoracdn.net/main-qimg-89211c8bdfbaf100807a73efb3d9c9cc-lq)

Hadoop MapReduce 작업을 실행하면 하둡은 디스크와 메모리 간에 모든 작업을 단계마다 쓰고 (write) 읽게(read)된다. 더 자세히 말하자면, Reduce 작업이 끝나고 중간 결과를 HDFS에 저장 후 다시 Map과 Reduce 작업을 반복하게 되면 Disk I/O가 발생하게 되어 작업의 속도가 느려진다. 맵리듀스는 특히나 메모리에 들어가지도 않을 만큼 작은 Task들이 반복적으로 수행된다면 처리 속도가 크게 느려질 수 있다.

<br>


## 그럼 스파크는 어떻게 이 문제를 해결했나?

![](https://phoenixnap.com/kb/wp-content/uploads/2021/04/hadoop-spark-data-processing.png)



Spark는 RDD 또는 Resilient Distributed Dataset이라고 하는 데이터 구조를 활용하여 **작업의 중간 결과를 디스크에 저장하지 않고 인메모리 캐싱**으로 한번에 처리한다. 스파크는 미리 실행계획을 계산하여 DAG(비동기 사이클 그래프)로 작업 순서, 리소스를 최적화하여 Disk에 저장하지 않고 Map단계의 데이터를 메모리에 저장하여 Reduce에 직접 전달함으로써 데이터 작업 처리속도를 증가시켜준다.

또한, 이 RDD는 여러 컴퓨터 노드에 흩어져 있으면서 병렬 처리될 수 있는 아이템들의 모음을 표현한 객체이기 때문에, 분산처리와 병렬처리가 동시에 가능하다는 장점이있다.
