# Grafana Loki
## 개요
- [Grafana Loki](#grafana-loki)
  - [개요](#개요)
  - [Grafana Loki](#grafana-loki-1)
  - [Loki 특징](#loki-특징)
  - [Loki 장단점](#loki-장단점)
  - [ElsaticSeach 차이점](#elsaticseach-차이점)
  - [Loki 아키텍처](#loki-아키텍처)
  - [Loki 구성 요소](#loki-구성-요소)
  - [코드 분석](#코드-분석)

## Grafana Loki
로그 레이블에 대한 메타데이터만 인덱싱 하자는 아이디어로 구축되었다. 그런 다음 로그 데이터 자체는 압축되어 AWS S3와 같은 객체 저장소에 청크로 저장되거나 파일 시스템에 로컬로 저장된다. 인덱스와 고도로 압축된 청크로 작업을 간소화하고 저렴한 객체 스토리지를 사용함으로써 운영 비용이 저렴하다.

Loki 철학
> Like Prometheus, But for logs

동일한 레이블은 어떻게 처리할까?  
로그 스트림으로 처리한다. 로그 스트림이란 동일한 레이블 세트를 가진 로그들의 집합을 의미한다. 

구성 요소
- 에이전트(grafana alloy, promtail)  
로그를 스크래핑하고, 레이블을 추가하여 로그를 스트림으로 변환하고, HTTP API를 통해 스트림을 Loki로 푸시합니다.
- Loki   
로그를 수집하고 저장하고 쿼리를 처리하는 서버
- Grafana  
logcli를 사용하거나 loki api를 사용하여 로그를 쿼리한다.

## Loki 특징
- PUSH를 통한 로그 수집
- 수평 확장성, 고가용성, 멀티 테넌트
- 객체 스토리지 활용
- LogQL
- Grafana 대시보드 
- 알림 
  - Ruler라 하는 구성 요소가 포함되어 있다 Ruler는 로그에 대한 쿼리를 지속적으로 평가하고 결과에 따라 작업을 수행한다. 이를 통해 로그에서 이상 현상 또는 이벤트를 모니터링할 수 있다. Prometheus Alertmanager와 연계하여 사용이 가능하다.  

## Loki 장단점
**장점**  
- 설정이 간편하다. 
- 그라파나와 통합된 대시보드 제공
- 클러스터 구성 시 메모리 사용량이 적음에도 동일한 성능을 제공한다. 

**단점**
- 객체 스토리지를 사용하여 비용이 매우 효율적이지만 블록 스토리지에 비해 상대적으로 느리다. 하지만 이러한 단점을 보완하기 위해 로그에 대한 메타데이터인 레이블을 인덱스하는 아이디어를 중심으로 구성되었다.

## ElsaticSeach 차이점
GPT의 답변

| 항목 | Grafana Loki | Elasticsearch |
|------|-------------|--------------|
| 저장 방식 | 메타데이터 기반 인덱싱 | 역색인 기반 인덱싱 |
| 데이터 저장 비용 | 상대적으로 저렴함 | 높은 저장 비용(색인 필요) |
| 성능 | 로그 검색 속도가 빠름 (경량) | 색인화로 인해 무거운 작업 가능 |
| 확장성 | 높은 확장성 | 높은 확장성, 무거운 자원 소모 |
| 복잡성 | 설정이 비교적 간단 | 설정 및 운영이 복잡함 |
| 검색 기능 | 정규식 기반 검색 | 다양한 검색(Full-text, Structured) 지원 |

## Loki 아키텍처 
![alt text](https://grafana.com/docs/loki/latest/get-started/loki_architecture_components.svg)
인덱스 파일(TSDB)에 대해서 Index Shipper라는 어댑터를 사용하여 청크 파일을 저장하는 것과 동일하게 객체 스토리지에 저장한다.

**데이터 형식**
- 인덱스(TSDB)
- 청크  

![alt text](https://grafana.com/docs/loki/latest/get-started/chunks_diagram.png)


## Loki 구성 요소
- Distributor
- Ingester
- Query frontend
- Query scheduler
- Querier
- Index Gateway
- Compactor
- Ruler
