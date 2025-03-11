# 이론 및 개념 학습
## 관측 가능성의 요소
### 관측 가능성이란
>관측 가능성 (observability): 시스템에서 외부로 출력되는 값만을 사용해서 시스템의 내부 상태를 이해하고 예측하는 것

<p align="center">
 <img src = "./images/모니터링과 관측 가능성 차이점.png" width="500">
</p>

- 블랙박스 모니터링 / 화이트박스 모니터링: 관찰자가 모니터링의 대상을 밖에서 보면 블랙박스, 내부에서 보면 화이트박스에 해당함
- 관측 가능성은 내부 시스템에 대한 자세한 이해를 기반으로 미래에 발생할 이벤트를 예측하는 것을 지향
- 모니터링에서의 APM(ApplicationPerformanceManagement) 대신 **Tracing**이라고 함
- 모니터링은 전체적인 시스템 상태를 이해하는 데에 적합하며 관측 가능성은 분산되고 복잡한 시스템에서 발생하는 이벤트에 대한 통찰력과 태그, 로그 등을 결합한 서비스 콘텍스트를 제공하는 데에 목적이 있음

### 용어 정리

>메트릭 (Metric): 일정 시간 동안 측정된 데이터를 집게하고 수치화 하는 것. 전체적인 시스템의 상태를 보고하는 데 유용함.
>
>- 시계열과 히스토그램 차트로 설명
>- 프로메테우스가 해당

</br>

>로그 (Log): 어플리케이션 실행 시 생성되는 텍스트 라인으로 JSON 형식 또는 비구조적인 텍스트 형식으로 출력됨.
>
>- 오픈텔레메트리를 사용해서 구조화된 로그를 생성
>- 오픈텔레메트리 로깅을 사용하면 로그 표준화가 가능

</br>

>추적 (Tracing): 마이크로서비스가 시스템을 경유하며 트랜잭션을 처리하는 과정에서 발생하는 세부적인 정보를 보여줌. 트랙잭션이 시스템을 이동하는 경로, 처리 과정에서 발생하는 대기/지연시간, 병목현상이나 에러를 일으키는 원인을 메타데이터에 출력함.
>
>- 예거는 추적 결과를 분석하고 추적에 대한 이해를 도움.

</br>

>가용성
>
>- 가용성 신호: 시스템의 전반적인 상태와 정상 작동 여부를 거시적 관점에서 측정함
>- SLI (Service Level Indicater, 서비스 수준 지표)
>- SLO (Service Level Objective, 서비스 수준 목표)
>- SLA (Service Level Agreement, 서비스 수준 협약)

## 실습 자료 확인
실습 자료로는 https://github.com/grafana/intro-to-mltp 레포지토리를 사용 </p>
본 자료는 Grafana를 쉽게 사용하며 실습할 수 있도록 되어 있음.</p>
<p align="center">
 <img src = "https://raw.githubusercontent.com/grafana/intro-to-mltp/refs/heads/main/images/Introduction%20to%20MLTP%20Arch%20Diagram.png">
</p>

### Grafana Observability Pipeline 작동 원리

#### 1. 데이터 수집 (Instrumentation)
   - Traces: OpenTelemetry SDK (자동/수동)
   - Logs: stdout/HTTP 출력
   - Metrics: Prometheus 라이브러리 및 Exporters
   - Profiles: Pyroscope SDK
   - Beyla: eBPF hooks 사용

#### 2. 서비스 계층
   - Mythical Server
   - Mythical Requester
   - Mythical Recorder
   - PostgreSQL과 RabbitMQ를 통해 데이터를 저장 및 교환

#### 3. 부하 테스트
   - Grafana k6 사용하여 서비스에 부하를 가하고 k6 파생 메트릭을 생성

#### 4. 데이터 수집 및 전달 (Grafana Alloy & OpenTelemetry Collector)
   - Grafana Alloy: 
     - Metrics Scraper
     - Log Scraper
     - Trace Receiver
     - Profile Receiver
   - OpenTelemetry Collector (선택)
     - Metrics Scraper
     - Log Receiver
     - Trace Receiver

#### 5. 데이터 저장소
   - Grafana Tempo (Trace Storage)
     - OTLP gRPC 사용하여 Trace 데이터 수집 및 저장
     - 로컬 볼륨 스토리지 사용
   - Grafana Mimir (Metrics Storage)
     - Prometheus gRPC를 통해 메트릭 저장
     - 로컬 볼륨 스토리지 사용
   - Grafana Loki (Logs Storage)
     - Loki HTTP로 로그 데이터 저장
     - 로컬 볼륨 스토리지 사용
   - Grafana Pyroscope (Profile Storage)
     - Pyroscope HTTP로 프로파일 데이터 저장
     - 로컬 볼륨 스토리지 사용

#### 6. Grafana Cloud 및 원격 저장 (선택)
   - CSP Cloud Storage (GCS/S3/Azure Blob)로 데이터 백업 및 클라우드 저장
   - TLS를 지원하는 gRPC/HTTP 통신을 통해 데이터를 송신

#### 7. 데이터 조회 및 시각화
   - 각 저장소는 HTTP API Query를 통해 Grafana에서 데이터를 조회하고 시각화

</br>

# 실습
## 환경 실행
https://github.com/grafana/intro-to-mltp 에서 repository clone 후 docker-compose를 사용해 실행</p>

<details>
<summary>명령어 및 실행예시</summary>
<div markdown="1">

```
git clone https://github.com/grafana/intro-to-mltp.git
cd intro-to-mltp
docker-compose up -d
docker ps -a
```

<p align="center">
 <img src = "./images/docker-compose 실행예시.png">
</p>

※주의※ 실행 시 메모리 사용량 8GB 도달 이후 5GB 사용함. i5-12600K 32GB 환경</p>

</div>
</details>

</br>

## Grafana 접속해보기
### 접속
http://localhost:3000/ 로 접속 </p>

</br>

### 데이터 소스 확인
Connections → Data sources 접속시 다음과 같이 데이터 소스가 등록되어 있음 </p>
<p align="center">
 <img src = "./images/grafana datasources.png">
</p>

<details>
<summary>안된다면?</summary>
<div markdown="1">

Add new data sources → 데이터 소스 선택 (경로 입력 필수)

</div>
</details>

</p>

</br>

### 대시보드 확인
Dashboard 접속시 다음과 같이 자동으로 MLTP 관련 JSON 대시보드가 불러와져 있음 </p>
<p align="center">
 <img src = "./images/grafana dashboard.png">
</p>

<details>
<summary>안된다면?</summary>
<div markdown="1">

New → Import → Upload 선택 후 intro-to-mltp/grafana/definitions 의 json 파일들 선택

</div>
</details>