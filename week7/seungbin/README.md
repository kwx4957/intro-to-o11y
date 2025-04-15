### Tracing, Logging, Metrics 수집 구성 가이드

---
## 1. Tempo – Distributed Tracing

### 목적
- 분산 추적 데이터(Trace)를 저장 및 쿼리
- OpenTelemetry 및 Istio와 연동하여 서비스 간 호출 추적

### 구성 방식
- **OpenTelemetry Collector → Tempo** (OTLP gRPC/HTTP or Jaeger, Zipkin 등)
- **Tempo metricGenerator → Prometheus (remote_write)**

### 주요 설정
```yaml
metricsGenerator:
  enabled: true
  config:
    processors:
      - span-metrics
      - service-graphs
    remoteWrite:
      - url: http://<prometheus>/api/v1/write

extraArgs:
  - -target=all,metrics-generator
```

## 2. Loki – Logging

### 목적

- Kubernetes 환경의 컨테이너 로그 수집 및 저장
    
- Grafana를 통한 로그 분석 및 연계
    

### 구성 방식

- **Promtail → Loki**
    
- Kubernetes Pod 로그 → Promtail 수집 → Loki 저장
    

### 설정

- `scrape_configs`에 `namespace`, `pod_name`, `container_name` 라벨 추가
    
- Istio Proxy (Envoy) 로그를 stdout으로 출력하게 설정하고 수집
    

---
## 3. Istio – Service Mesh & Tracing

### 목적

- 서비스 간 트래픽을 제어하고 관찰 가능성(Observability)을 제공
    
- Envoy 프록시를 통해 자동 트레이스 생성
    
### 구성 방식

- Istio Sidecar(Envoy)가 자동으로 트레이스를 생성하여 OTLP, Zipkin 등으로 전송
    
- Trace는 OpenTelemetry Collector나 Tempo로 전달
    
### 주요 설정 예시

``` yaml
meshConfig:
  defaultConfig:
    holdApplicationUntilProxyStarts: true
    tracing:
      zipkin:
        address: otel-collector.observability.svc.cluster.local:9411
```
### 주의사항

- SDK 없이도 자동 트레이싱 가능
    
- 내부 함수 단위 추적이 필요하면 OpenTelemetry SDK 계측 필요
    
- Propagation 설정 (B3, W3C 등)을 Istio ↔ 앱 ↔ Collector 간 일치시켜야 함

--- 

## 4. OpenTelemetry Collector
### 목적

- 다양한 형식의 trace, metric, log 데이터를 수집하고 변환하여 백엔드(Tempo, Prometheus, Loki 등)로 전달
    
- Istio, SDK, Promtail 등 다양한 소스에서 오는 데이터를 중앙 집중 처리
    
### 구성 방식

- **Receivers**: OTLP (gRPC/HTTP), Jaeger, Zipkin, Prometheus, Logs
    
- **Processors**: Batch, Attributes, Resource, Filter 등
    
- **Exporters**: OTLP, Tempo, Prometheus, Loki 등
    

### 예시 구성 (`otel-collector-config.yaml`)

``` yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:
      zipkin:

processors:
  batch: {}
  resource:
    attributes:
      - key: service.namespace
        action: insert
        value: default

exporters:
  otlp/tempo:
    endpoint: tempo.observability.svc.cluster.local:4317
    tls:
      insecure: true

  prometheus:
    endpoint: "0.0.0.0:8889"

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch, resource]
      exporters: [otlp/tempo]

    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheus]
```

### Helm 설치 예시

``` yaml
helm upgrade --install otel-collector open-telemetry/opentelemetry-collector \
  -n observability --create-namespace \
  -f otel-collector-values.yaml
```
### 활용

- Istio에서 생성된 trace를 OTLP 수신 → Tempo로 전송
    
- 애플리케이션에서 SDK를 통해 보낸 trace/metric/log 수집
    
- Prometheus exporter로 노출된 metric → Grafana에서 시각화 가능
    
- Loki exporter 구성 시 로그 수집도 가능 (파일/컨테이너 로그 등)
    
### 진단 및 확인

- 수신 여부: `otelcol_receiver_accepted_spans` 메트릭
    
- 전송 여부: `otelcol_exporter_sent_spans` 메트릭
    
- 상태 확인: `otelcol_component_*` 계열 메트릭
    
- `/metrics`, `/healthz` 등 헬스엔드포인트 노출