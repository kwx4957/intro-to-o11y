# 이론 및 개념 학습
## 관측 가능성의 요소
<p align="center">
 <img src = "./images/모니터링과 관측 가능성 차이점.png">
</p>



## 실습 자료 확인
실습 자료로는 https://github.com/grafana/intro-to-mltp 레포지토리를 사용 </p>
본 자료는 Grafana를 쉽게 사용하며 실습습할 수 있도록 되어 있음.</p>
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