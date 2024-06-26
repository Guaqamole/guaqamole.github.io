---
title: 로드 밸런서(Load Balancer)
author: guaqamole
date: 2023-02-02 18:32:00 +0900
categories: [CS, Network]
tags: [CS, Network, LoadBalancer, TCPIP]
image:
  path: /common/lb.png
---

유저들에게 제공하는 서비스가 초기 단계라면 적은 수의 클라이언트로 인해 서버 한 대로 요청에 응답하는 것이 가능 할것이다. 

하지만 서비스의 규모가 확장되고, 클라이언트의 수가 늘어나게 되면 기존 서버의 리소스 만으로는 정상적인 서비스를 유지하기가 어렵다. 

또한 어느 특정 이벤트 때문에 임시적으로 유저의 수가 급진적으로 증가한다면,  잠깐 늘어날 유저수 때문에 미리 대비하여 서버를 재구성하는것도 힘들것이다. 따라서 이렇게 급진적으로 증가하는 트래픽에 대처할 수 있는 방법은 크게 2가지가 있다.

#### Scale Up

- 서버 자체의 성능을 확장하기 위해 **리소스를 증가시**키는 행위

#### Scale Out

- 기존 서버와 동일하거나 낮은 성능의 서버를 **두 대 이상 증설**하여 운영하는 행위

서버를 추가로 증설하는 경우, 여러 대의 서버로 트래픽을 균등하게 분산해주는 중간다리 역할이 필요하게 된다. 이처럼 클라이언트와 서버 사이에서 트래픽, 즉 **서버의 부하(Load)를 분산**시켜주는 기술을 ***로드밸런싱*** 이라고한다.

요즘 인프런에서 구한 스터티원분들과 같이 개발 스터디를 하면서, 로드 밸런서 개념을 정리할필요가 있다고 생각했다. 

그분들에게 공유할겸 한번 정리해보자!

<br>

****


## **로드 밸런서란**
>  `서버나 장비의 트래픽을 분산시키기 위해 클라이언트와 서버 사이에 위치한 네트워킹 장치 혹은 어플리케이션`



로드 밸런서는 OSI 모델의 여러 계층에서 사용되며, 일반적으로 Layer 4 (TCP/UDP) 또는 Layer 7 (HTTP/HTTPS)에서 많이 사용된다.

#### L4 Load Balancing
![](/230702/l4.png){: width="972" height="589" }

- **전송 계층**에서 로드를 분산
- **IP주소나 포트번호, MAC주소** 등에 따라 트래픽을 나누고 분산처리가 가능
- CLB(Connection Load Balancer) 혹은 SLB(Session Load Balancer)라고 부름

**장점**

- 패킷의 내용을 확인하지 않고 로드를 분산하므로 **속도가 빠르고 효율이 높음**
- 데이터의 내용을 **복호화할 필요가 없기에** 안전
- L7 로드밸런서보다 **가격이 저렴**

**단점**

- 패킷의 내용을 살펴볼 수 없으므로, **섬세한 라우팅 불가.**
- **사용자의 IP가 수시로 바뀌는 경우**라면, 연속적인 서비스를 제공하기 어려움

<br>

#### L7 Load Balancing
![](/230702/l7.png){: width="972" height="589" }

- **애플리케이션 계층**에서 로드를 분산
- OSI 7계층의 프로토콜(HTTP, SMTP, FTP 등)을 바탕으로도 분산 처리가 가능

**장점**

- 상위 계층에서 로드를 분산하기 때문에 **훨씬 더 섬세한 라우팅** 가능
- **캐싱(Cashing) 기능**을 제공
-  비정상적인 트래픽을 사전에 필터링할 수 있어 **서비스 안정성 증가**

**단점**

-  L4 로드밸런서에 비해 **비쌈**
- **패킷의 내용을 복호화하여야** 하므로 더 높은 비용을 지불해야 함
- 클라이언트가 로드밸런서와 인증서를 공유해야 하기 때문에, 공격자가 로드밸런서를 통해 클라이언트의 데이터에 접근할 수 있는 **보안상의 위험성** 존재

<br>

#### **로드 밸런서의 기능**

1. **Health Check (상태 확인)**

   - 서버들에 대한 주기적인 Health Check를 통해 **서버들의 장애 여부를 판단**하여, **정상 동작 중인 서버로만 트래픽을 보냄**
   - L3 체크 : ***ICMP**를 이용하여 서버의 **IP 주소가 통신 가능한 상태인지를 확인**
   - L4 체크 : TCP는 3 Way-Handshaking (전송 - 확인/전송 - 확인) 를 기반으로 통신하는데, 이러한 **TCP**의 특성을 바탕으로 **각 포트 상태를 체크**하는 방식
   - L7 체크 : **어플리케이션 계층에서 체크**를 수행; **실제 웹페이지에 통신을 시도**하여 이상 유무를 파악

   

2. **Tunneling (터널링)**

   - 데이터 스트림을 인터넷 상에서 가상의 파이프를 통해 전달시키는 기술로, 패킷 내에 터널링할 대상을 캡슐화시켜 목적지까지 전송
   - 연결된 상호 간에만 캡슐화된 패킷을 구별해 캡슐화를 해제하게 함

   

3. **NAT (Network Address Translation)**

     - 내부 네트워크에서 사용하는 **사설 IP 주소**와 로드밸런서 외부의 **공인 IP 주소 간의 변환 역할.**

     - 로드밸런싱 관점에서는 여러 개의 호스트가 **하나의 공인 IP 주소(VLAN or VIP)를 통해 접속하는 것이 주 목적.**

     - **SNAT (Source Network Address Translation)** : 내부에서 외부로 트래픽이 나가는 경우, **내부 사설 IP 주소 -> 외부 공인 IP 주소**로 변환.

     - **DNAT (Destination Network Address Translation)** : 외부에서 내부로 트래픽이 들어오는 경우.
          **외부 공인 IP 주소 -> 내부 사설 IP 주소**로 변환.



4. **DSR (Destination Network Address Translation)**

   - 서버에서 클라이언트로 트래픽이 되돌아가는 경우, 목적지를 클라이언트로 설정한 다음, 네트워크 장비나 로드밸런서를 거치지 않고 바로 클라이언트를 찾아가는 방식 

   - 이 기능을 통해 **로드밸런서의 부하를 줄여**줄 수 있음

<br>


## 로드 밸런서의 장점

##### **부하 분산**

로드 밸런서는 서버 그룹에 들어오는 트래픽을 여러 서버로 균등하게 분산하여 각 서버의 부하를 분담한다. 이를 통해 특정 서버의 과부하를 방지하고 시스템 전체의 성능을 향상시킨다.

##### **고가용성** 

로드 밸런서는 여러 대의 서버를 관리하므로, 한 대의 서버에 장애가 발생해도 다른 서버로 요청을 전달할 수 있다. 이를 통해 시스템의 가용성을 높일 수 있다.

##### **확장성 **

새로운 서버를 시스템에 추가하거나 기존 서버를 제거하는 경우, 로드 밸런서는 자동으로 이를 감지하고 트래픽을 새로운 서버로 분배한다. 이를 통해 시스템의 확장성을 유지하고 유연한 운영이 가능하게 된다.

<br>

## 로드 밸런서의 동작원리

![](/230702/2.png){: width="972" height="589" }
_Fig 2. How Load Balancer Works_

1. **Get Request**: 클라이언트가 로드 밸런서에 요청을 한다.
2. **Analyze Traffic**: 로드 밸런서는 미리 설정된 알고리즘을 사용하여 요청을 분석한다. 알고리즘에는 라운드 로빈, 최소 연결, 해시 기반 등이 있다.
3. **Health Check**: 주기적인 상태 점검을 통해 서버의 건강을 확인하고 트래픽을 분배한다.
4. **Routing**: 알고리즘과 상태 점검을 바탕으로 적절한 서버를 선택한다.
5. **Send Request**: 선택된 서버로 요청을 전달한다.



#### 알고리즘 종류

##### 라운드로빈 (Round Robin Method)

- 서버에 들어온 요청을 순서대로 돌아가며 배정하는 방식
- 클라이언트의 요청을 순서대로 분배하기 때문에 여러 대의 서버가 동일한 스펙을 갖고 있고, 서버와의 연결(세션)이 오래 지속되지 않는 경우에 활용하기 적합함.

##### 가중 라운드로빈 (Weighted Round Robin Method)

- 각각의 서버마다 가중치를 매기고 가중치가 높은 서버에 클라이언트 요청을 우선적으로 배분하는 방식
- 주로 서버의 트래픽 처리 능력이 상이한 경우 사용되는 부하 분산 방식이다. 예를 들어 `A라는 서버가 5`라는 가중치를 갖고 `B라는 서버가 2`라는 가중치를 갖는다면, 로드밸런서는 라운드로빈 방식으로 `A 서버에 5개` `B 서버에 2개`의 요청을 전달함.

##### IP 해시 (IP Hash Method)

- 클라이언트의 IP 주소를 특정 서버로 매핑하여 요청을 처리하는 방식
- 사용자의 IP를 해싱해(Hashing, 임의의 길이를 지닌 데이터를 고정된 길이의 데이터로 매핑하는 것, 또는 그러한 함수) 로드를 분배하기 때문에 사용자가 항상 동일한 서버로 연결되는 것을 보장함.

##### 최소 연결 (Least Connection Method)

- 요청이 들어온 시점에 가장 적은 연결상태를 보이는 서버에 우선적으로 트래픽을 배분하는 방식
- 자주 세션이 길어지거나, 서버에 분배된 트래픽들이 일정하지 않은 경우에 적합함.

##### 최소 리스폰 타임 (Least Response Time Method)

- 서버의 현재 연결 상태와 응답시간(Response Time, 서버에 요청을 보내고 최초 응답을 받을 때까지 소요되는 시간)을 모두 고려하여 트래픽을 배분하는 방식
- 가장 적은 연결 상태와 가장 짧은 응답시간을 보이는 서버에 우선적으로 로드를 배분함.

<br>

#### L7 활용 예시

사용자 세션에 대한 정보는 종종 브라우저에 로컬로 저장된다. 예를 들어, 장바구니 애플리케이션에서 사용자 장바구니의 상품은 사용자가 구매할 준비가 될 때까지 브라우저 수준에 저장될 수 있다. 

근데 만약 쇼핑 세션 중에 해당 클라이언트로부터 요청을 받는 서버 엔드포인트가 변경되면 성능 문제가 발생하거나 완전한 트랜잭션 오류가 발생할 수 있다. 이러한 경우 세션이 진행되는 동안 클라이언트의 모든 요청이 동일한 서버로 전송되는 것이 중요한다. 

이렇게 상황에따라 로드 밸런서를 통해 세션별로 정보를 동일한 서버로 보내는것을 *세션 지속성* 이라고 한다 . [서버 친화성을](http://wiki.metawerx.net/wiki/ServerAffinity) 달성하기 위해 [Application Load Balancing](http://wiki.metawerx.net/wiki/ApplicationLoadBalancing) 과 함께 사용되는 방법이다.



<br>

## Links
<https://m.post.naver.com/viewer/postView.naver?volumeNo=27046347&memberNo=2521903>

<https://www.ncloud.com/product/networking/loadBalancer>

<https://nesoy.github.io/articles/2018-06/Load-Balancer>

<https://suuntree.tistory.com/348>

<https://julie-tech.tistory.com/73>