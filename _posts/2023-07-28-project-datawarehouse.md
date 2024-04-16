---
title: 프로젝트 회고 | e-Commerce DW 구축기
author: guaqamole
date: 2023-07-28 13:00:00 +0900
categories: [Project, Marketboro]
tags: [DW, BigQuery, Airflow, Dimensional Modeling]
image:
  path: /common/torvalds.png
published: true
---

데이터 불모지인 유통 시장에서 큰 성공을 이뤄낸 마켓보로라는 회사에 Data Engineer로 입사 후, 처음으로 진행한 데이터 플랫폼 프로젝트다. 회사가 급성장함에 따라 **플랫폼 별로 흩어져 있는 데이터를 중앙화 해달라는 요구사항 이었다.**  데이터 사이즈와는 별개로 이건 해본 사람만 알 수 있는 난이도가 높은 작업이다.

**그중 나의 역할은 MySQL 데이터를 BigQuery로 적재 후, 전사적으로 사용할 DW를 모델링하고 구축 하는것이였다.**

Data Engineering에 대해 경험이 부족해서 그런지, 출근 할 때 마다 더 평소보다 긴장되었고 오랜만에 느껴보는 쫄깃함이 마음속에 공존했다. 누가 시키진 않았지만:

- 서비스 별로 도메인 지식과, DB 스키마, 데이터의 성격을 최대한 이해해야했고 (1달 걸림..)
- DW를 어떻게 모델링 해야할지 (책읽고 모델링하는데..1달)
- 분석가들의 데이터 니즈는 무엇인지 (항상..)
- 데이터를 어떻게 ETL 할지 (항상..)

출 퇴근시 매일 자료를 찾고 고민하며 살았던 3개월이였다.

**그중 가장 힘들었던건 부족한 레퍼런스였다**. Spring 처럼 자료가 많아서 한글로 검색하던 영어로 검색하던 블로그, 공식 문서가 쏟아져 나오는게 아니였다. 내가 유일하게 의지할 수 있었던건 Ralph Kimball의 명저 DataWarehouse Toolkit 이였다. 이 책은 한글 번역도 구하기가 힘들어 오랜만에 묵혀두었던 대학 시절 논문 노가다를 시전해야만 했으며 거의 교회 성경책 처럼 끼고 살았다.

역시 인생은 배움의 연속이라는걸 다시한번 느끼며 어려운 마음으로 프로젝트를 시작했다.

<br>

****

## Problem Statement

전사적으로 사용할 DW를 바닥 상태에서 구축하려면 다음 큰 문제들을 하나씩 풀어나가야 했다:

1. DW는 어디에 구축할것인가?
   1. Redshift vs BigQuery
2. Data Engineering - 어떻게 ETL 할것인가?
   1. Airflow DAG는 어떻게 작성할것인가?
   2. ELT vs ETL vs EtLT
   3. 증분 vs CDC
   4. 초기 데이터는 어떻게 Load 할것인가?
3. Dimensional Modelling - 어떻게 모델링 할것인가?
   1. Star vs Snowflake Schema
   2. Granularity
   3. Fact vs Dimension
   4. 어디까지 중복을 허용할 것인가?
4. Data Engineering - 어떻게 Transform 할것인가?
   1. BigQuery Insert vs Merge
   2. Partition
5. 비용 vs 성능
6. Data Engineering - 어떻게 DW를 운영할것인가?
   1. Data Integration 이슈 (중복, 누락) 발생시 어떻게 대처할까?
   1. 검증은 어떻게 할것인가?

등 정말 고민할게 너무나도 많았다. 사실 이걸 3개월만에 끝낼 수 있을까? 라는 걱정이 앞섰지만, 전사적으로 Data가 안정적으로 흘러가려면 "내가 진행한 프로젝트가 기간안에 무조건 마무리 되어야한다 라는 생각에 밤낮없이 작업했던거 같다.

## 초기 단계

스타트업은 시간이 생명이지만,  유한한 투자금 또한 생명이다. 투자가 많이 들어왔다고 해서 비용을 남발해서는 안되며, 엔지니어는 어떤 상황에서도 비용을 최적화 해야한다고 생각한다. 

또한 스타트업에선 데이터 조직이 개발 조직 만큼 당장 퍼포먼스를 낼 수 없기 때문에 초기부터 막대한 비용이 들어가면 리더분들이 난감해지기 때문에 "가장 저렴하고 효과적인 방안"을 우선적으로 생각했다.

### 1. DW는 어디에 구축할것인가?

현재 회사의 모든 서비스는 AWS 기반으로 구축 되어있다. Cloud 기반 Columnar Storage는 대표적으로 Amazon Redshift와 GCP BigQuery가 양대산맥이다. 

Redshift와 BigQuery를 둘다 사용해본 경험이 없기 때문에, 문서로만 이 둘의 스펙을 비교할 수 밖에 없었다. 하지만 나는 결국 BigQuery의 저렴한 비용과, Fully-managed 방식, 그리고 Google Platform과 높은 연계성에 매료되고 말았다..

물론 Redshift가 fully-managed가 아니란건 아니다. 하지만 redshift를 구성할 때 별도의 클러스터 관리가 필요하므로, 굳이 더 높은 비용 내고 성능 안정성을 위해 편의성을 포기할 순 없었다.

#### 결론

- 성능을 우선으로 고려하면 Redshift, 비용와 편이를 고려하면 BigQuery
- GA, Google spreadsheet 등 google platform과 연계 할 일이 잦다면 BigQuery

### 2. 어떻게 ETL 할것인가?

MySQL 데이터를 BigQuery로 적재하기로 결정 한 후 이제 "어떻게"의 문제에 직면했다. 머릿속에는 크게 두가지 방안이 당장 떠올랐다. 

DB 부하와 비용이 적게드는 증분 방식과, DB Transaction log를 읽어 실시간으로 데이터를 Sync 하는 CDC (Change Data Capture) 방식을 고민했다. CDC 방식으로 결정 할 경우 별도의 스트리밍 처리(kafka 구축)가 필요하게 되어 프로젝트가 더 복잡해지고, 비용도 높아질것이다.  

무엇보다 제일 중요한건, 실시간 분석 니즈가 당장 필요하지 않은데 굳이 높은 비용을 지불하며 실시간으로 데이터를 가져올 필요가 없다는 것이였다. 물론 추후에 실시간 니즈가 생길 가능성도 있었지만, 성능보단 안정성있는 데이터 플랫폼을 만드는 것이 우선이였고, 실시간 분석은 신설된 데이터 조직에게 아직 걸음마도 떼지 않은 영유아에게 달려보라는것과 같았다. 

따라서, 이번 프로젝트에서는 한시간마다 증분 데이터를 적재하는 Batch 방식을 선택했다. 대신 구축에 사용되는 비용을 최소(최적)화하여 대표님과 리더분들을 기쁘게 해드리로 결심했다.

![](/230728/2.png)

<br>

## Reference
- https://kubernetes.io/ko/docs/concepts/overview/