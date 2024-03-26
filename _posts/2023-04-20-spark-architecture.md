---
title: Spark의 아키텍처
author: guaqamole
date: 2023-04-20 18:32:00 +0900
categories: [Data, Spark]
tags: [Data Engineering, Spark, Distributed Computing]
image:
  path: /common/spark.png
---

## 스파크 구성

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdO7I5E%2FbtrLPn3nvsE%2F9NnFhSeoAzaHIHvGCCI4p0%2Fimg.png)

스파크 코어(Spark Core)는 작업 스케줄링, 메모리 관리, 장애 복구, 저장 장치와의 연 동 등등 기본적인 기능들로 구성됩니다. 스파크 코어는 탄력적인 분산 데이터세트(RDD, Resilient Distributed Dataset)를 정의하는 API의 기반이 되며, 이것이 주된 스파크 프로그 래밍 추상화의 구조입니다. RDD는 여러 컴퓨터 노드에 흩어져 있으면서 병렬 처리될 수 있 는 아이템들의 모음을 표현합니다. 스파크 코어는 이 컴포넌트들을 생성하고 조작할 수 있는 수많은 API를 지원합니다.

<br>

## 스파크 클러스터 아키텍처

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlEDkS%2FbtrjjNpiLda%2FcLKtcKE75vbcVMEkZPLFI0%2Fimg.png)

스파크 클러스터의 구조는 크게 Master node 와 worker 노드로 구성됩니다. 

- Master node는 전체 클러스터를 관리하고 분석 프로그램을 수행하는 역할을 합니다. (사용자가 만든 분석 프로그램을 Driver Program 이라고 합니다.) 
- 이 분석 프로그램을 스파크 클러스터에 실행하게 되면 하나의 JOB이 생성됩니다.
- 이렇게 생성된 JOB이 외부 저장소로 부터 데이터를 로딩하는 경우, 이 데이터는 스파크 클러스터 Worker node로 로딩이 되는데, 로딩된 데이터는 여러 서버의 메모리에 분산되어 로딩 됩니다.
- 이렇게 스파크 메모리에 저장된 데이터 객체를 RDD라고 합니다. 

이렇게 로딩된 데이터는 애플리케이션 로직에 의해서 처리되는데, 하나의 JOB이 여러 worker node에 분산된 데이터를 이용해서 분산되어 실행되기 때문에, 하나의 JOB은 여러개의 Task로 분리되어 실행 됩니다. 이렇게 나눠진 Task를 실행하는 것을 Executor 라고 합니다. 

<br>

## 클러스터 매니저

스파크는 데이터를 분산 처리하기 위해서 하나의 클러스터 내에 여러대의 머신, 즉 워커(Worker)들로 구성됩니다. 하나의 JOB이 여러대의 워커에 분산되서 처리되기 위해서는 하나의 JOB을 여러개의 TASK로 나눈 후에, 적절하게 이 TASK들을 여러 서버에 분산해서 배치 해야 합니다. 또한 클러스터내의 워크 들도 관리를 해줘야 하는데, 이렇게 클러스터내의 워커 리소스를 관리하고 TASK를 배치 하는 역할을 하는 것이 클러스터 매니저입니다.

![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbdRFlH%2FbtrjmWEOc3T%2FdICVW0GHeFZx8lFFqpsLf1%2Fimg.png)

워크들을 관리할 수 있는 클러스터 매니저는 일종의 스파크 런타임이라고 생각하면 되는데, 위 그림과 같이 Standalone , Yarn, SIMR 등의 타입이 있습니다. 

<br>

## 스파크 드라이버

드라이버란 프로그램의 main() 메소드가 실행되는 프로세스를 말합니다. **드라이버는 SparkContext를 생성하고 RDD를 만들고 트랜스포메이션과 액션을 실행하는 사용자 코드를 실행하는 프로세스입니다.** 스파크 셸을 실행할 때 이미 드라이버 프로그램을 만든 것입니다(스파크 셸은 미리 로딩된 sc라고 불리는 SparkContext 객체를 갖고 시작합니다) 드라이버가 종료되면 애플리케이션도 종료됩니다.

### 역할 1. **사용자 프로그램을 태스크로 변환**

스파크 드라이버는 사용자 프로그램을 물리적 실행 단위가 되는 태스크(job은 전체적인 '작업'을 의미하며 task는 하나의 노드에서 실행되는 단위 작업을 의미)로 바꿀 책임을 갖습니다. 상위 수준에서 모든 스파크 프로그램은 동일한 구조를 갖습니다. 입력으로부터 RDD를 만들고, 트랜스포메이션을 사용하여 새로운 RDD를 받아오며, 데이터를 가져오거나 저장하기 위해 액션을 사용합니다. 스파크 프로그램은 내부적으로 연산들의 관계에 대해 논리적인 비순환 그래프(DAG: Directed Acyclic Graph)를 생성합니다. 드라이버가 실행될 때 드라이버는 이 논리 그래프를 물리적인 실행계획으로 변환합니다.

스파크는 맵/트랜스포메이션을 "pipelining"해서 합치는 등의 여러가지 최적화를 통해 실행 그래프를 여러 개의 단계(stage)로 바꿉니다. 각 단계는 순서에 따라 여러 개의 태스크로 이루어지며 단위 작업들은 묶여서 클러스터로 전송됩니다. 태스크는 스파크의 작업 계층에서 가장 작은 단위의 개체입니다. 대개 사용자 프로그램은 수백 개에서 수천 개의 개별 태스크를 실행합니다.

### 역할 2. **익스큐터에서 태스크들의 스케줄링**

물리적 실행 계획이 주어지면 스파크 드라이버는 익스큐터에서의 개별 작업들을 위한 스케줄을 조정합니다. 익스큐터들은 시작하면서 드라이버에 등록을 하게 되므로 항상 애플리케이션의 실행에 대해 전체적으로 볼 수 있습니다. 각 익스큐터는 태스크들을 실행하고 RDD 데이터를 저장하는 프로세스입니다.

스파크 드라이버는 항상 실행 중인 익스큐터들을 살펴보고 각 태스크를 데이터 위치에 기반해 적절한 위치에서 실행될 수 있도록 노력합니다. 작업들이 실행되면 이미 캐시된 데이터를 또 저장하는 부작용이 있을수도 있습니다. 드라이버는 또한 이것들을 추적하여 그 데이터를 사용하게될 이후의 작업들이 적절하게 스케줄될 수 있도록 합니다.

드라이버는 기본적으로 4040 포트를 사용하는 WebUI를 통해 스파크 애플리케이션의 실행 정보를 보여줍니다. 

<br>

## Spark Executor

스파크의 익스큐터는 주어진 스파크 작업의 개별 태스크들을 실행하는 작업 실행 프로세스입니다. 익스큐터는 스파크 애플리케이션 실행시 최초 한번 실행되며 대개 애플리케이션이 끝날 때까지 계속 동작하지만, 익스큐터가 오류로 죽더라도 스파크 애플리케이션은 계속 실행됩니다. 익스큐터는 두가지 역할을 합니다. 첫번째로 애플리케이션을 구성하는 작업들을 실행하여 드라이버에 그 결과를 되돌려 줍니다. 두번째로 각 익스큐터 안에 존재하는 블록 매니저라는 서비스를 통해 사용자 프로그램에서 캐시하는 RDD를 저장하기 위한 메모리 저장소를 제공합니다. RDD가 익스큐터 내부에 직접 캐시되므로 단위 작업들 또한 같이 실행되기에 용이합니다.

<br>

## Spark Session

데이터셋을 다루기 위해 가장 먼저 알아야 할것은 SparkSession입니다. RDD를 생성하기 위해 SparkContext가 필요했던 것처럼 데이터프레임을 생성하기 위해서는 SparkSession을 이용해야 합니다. SparkSession은 인스턴스 생성을 위한 build() 메서드를 제공하고, 이 메서드를 이용하면 기존 인스턴스를 재사용하거나 새로운 인스턴스를 생성할 수 있습니다. 만약 스파크셸을 사용합니다면 스파크 셸이 자동으로 spark라는 이름으로 SparkSession 인스턴스를 생성해주므로 별도의 생성 단계를 거치지 않고 spark라는 변수를 통해 접근할 수 있습니다. SparkSession 이전에는 Hive 사용 여부에 따라 SQLContext와 HiveContext라는 별도의 API를사용했지만 SparkSession에서 Hive를 기본으로 지원하면서 기존 API는 더이상 사용하지 않게 되었습니다.

<br>

## Spark Context

이 둘의 차이는 스파크 애플리케이션에서 사용되는 **스파크의 버전**에 있습니다.
스파크2.0 이전에는 SparkContext가 스파크 애플리케이션의 집합점이었고 모든 스파크 기능과 클러스터 설정과 스파크컨텍스트객체를 만드는 파라미터에 설정에 접근하기 위해서는 스파크 컨텍스트를 사용해야했습니다.

<br>

## 배포 모드

배포 모드는 Spark Application을 실행할 때 요청한 자원의 물리적인 위치를 결정합니다.
선택할 수 있는 배포 모드는 다음과 같습니다.

- Cluster Mode
- Client Mode
- Kubernetes
- Standalone (단독)
- Local Mode (로컬)

이 중 Cluster Mode와 Client Mode는 Driver, Executor, Cluster Manager 마다 차이가 있습니다:

| Deploy Mode | 드라이버                                          | 익스큐터 | 클러스터 매니저 |
| ----------- | ------------------------------------------------- | ----------- | ----------- |
| Cluster | Yarn Application Master에서 동작 | 얀의 노드매니저의 컨테이너 | Spark Application을 실행하는 Node에서 Driver 실행 |
| Client  | Cluster 외부 클라이언트에서 동작 | 얀의 노드매니저의 컨테이너 | 1) Yarn RM이 Yarn Application Master와 연계                              2) Node Manager에게 익스큐터를 위한 컨테이너들을 할당 |
| Kubernetes | Pod안에서 동작 | Worker가 자신의 Pod 내에서실행 | 쿠버네티스 마스터 |
| Standalone | 클러스터의 아무 노드에서 실행 가능 | 각 노드가 자체적인 익스큐터 JVM 실행 | 클러스터의 아무 노드에서 실행 가능 |
| Local | 단일 서버의 JVM 위에서 실행 | 드라이버와 동일한 JVM위에서 동작 | 동일한 호스트에서 실행 |



### Cluster Mode

하나의 Worker Node에 Spark Driver를 할당하고, 다른 Worker Node에 Executor를 할당합니다.

![img](https://velog.velcdn.com/images/jskim/post/a293d0f1-2c54-49d5-99e9-a6e05d23d7e0/image.png)

> 실선 네모박스 : Spark Driver Process
>
> 점선 네모박스 : Spark Executor Process

- 가장 흔하게 사용되는 Spark Application 실행 방식
- Cluster Mode를 사용하려면 Compiled 된 JAR 파일 / Python Script / R Script를 Cluster Manager에 전달해야 합니다.
- Cluster Manager는 파일을 받은 다음 Worker Node에 Driver Process와 Spark Executor Process를 실행합니다.
- 이 때 Cluster Manager는 모든 Spark Application과 관련된 Process를 유지하는 역할을 합니다.


### Client Mode

Spark Driver는 Cluster 외부의 Machine에서 실행되며, 나머지 Worker는 Cluster에 위치해 있습니다.
![img](https://velog.velcdn.com/images/jskim/post/334634c1-2e99-444e-aef3-e9f65f7e8d32/image.png)

> 실선 네모박스 : Spark Driver Process
>
> 점선 네모박스 : Spark Executor Process

- Cluster Mode와 비슷하지만 다른 점은 Application을 제출한 Client Machine에 Spark Driver가 위치합니다.
- 이 때 Cluster Manager는 Executor Process를 유지하고, Client Machine은 Spark Driver Process를 유지합니다.
- 그림에서 보다시피 Spark Application이 Cluster와 무관한 Machine에서 동작하는데 보통 이런 Machine을 **Gateway Machine** 또는 **Edge Node**라고 합니다.

### Local Mode

- 이 경우 모든 Spark Application은 단일 Machine에서 실행됩니다.
- Local Mode는 Application의 병렬 처리를 위해 단일 Machine의 스레드(Thread)를 활용합니다.
- Spark 학습, Application Test, 개발 중인 Application 반복 실험 용도로 주로 사용됩니다.
- 운영용 Application을 실행할 때는 Local Mode를 권장하지 않습니다.

<br>

## 분산 데이터와 파티션

실제 물리적인 데이터는 HDFS나 클라우드 저장소에 존재하는 파티션이 되어 저장소 전체에 분산됩니다. 데이터가 파티션으로 되어 물리적으로 분산되면서, 스파크는 각 파티션을 고수준에서 논리적인 데이터 추상화, 즉 메모리의 데이터 프렝미 객체로 바라보게 됩니다. 항상 가능한것은 아니지만, 각 스파크 익스큐터는 가급적이면 데이터 지역성(geolocation)을 고려하여 네트워크에서 가장 가까운 파티션을 읽어 들이도록 태스크를 할당합니다.

이런 파티셔닝은 효과적인 병렬 처리를 가능하게 해주는데, 데이터를 조각내어 청크나 파티션으로 분산해 저장하는 방식은 스파크 익스큐터가 네트워크 사용을 최소화 하며 가까이 있는 데이터만 처리 할 수 있도록 도와줍니다.

다시말해, 각 익스큐터가 쓰는 CPU 코어는 작업해야 하는 데이터의 파티션에 할당되게 됩니다.

~~~python
log_df = spark.read.text("namkyu_text_file").repartition(8)
print(log_df.rdd.getNumPartitions())
~~~

위 예제 코드를 실행하게 되면 클러스터에 나뉘어서 저장된 물리적 데이터들을 8개의 파티션으로 나누고, 각 익스큐터는 하나 이상의 파티션을 메모리로 읽어 들이게됩니다.

<br>

## 스파크 프로그램 실행과정

1. 사용자는 spark-submit을 사용하여 애플리케이션을 제출합니다.
2. spark-submit은 드라이버 프로그램을 실행하고 사용자가 정의한 main() 메소드를 호출합니다.
3. 드라이버 프로그램은 클러스터 매니저에게 익스큐터 실행을 위한 리소스를 요청합니다.
4. 클러스터 매니저는 드라이버 프로그램을 대신해 익스큐터들을 실행합니다.
5. 드라이버 프로세스가 사용자 애플리케이션을 통해 실행됩니다. 프로그램에 작성된 RDD의 transformation과 action에 기반하여 드라이버는 작업 내역을 단위 작업 형태로 나눠 익스큐터들에게 보낸다.
6. 단위 작업들은 결과를 계산하고 저장히기 위해 익스큐터에 의해 실행됩니다.
7. 드라이버의 main()이 끈나거나 SparkContext.stop()이 호출됩니다면 익스큐터들은 중지되고 클러스터 매니저에 사용했던 자원을 반환합니다.

<br>

## Spark Job이 수행되는 과정

![](https://moons08.github.io/assets/img/post/spark/internals_of_job_execution_in_spark.png)



