
# Grafana Mimir

## 1. 메트릭 관측 시스템 구성: 저장소 vs 수집기

| 구성 요소 | 설명 |
|-----------|------|
| 메트릭 생성기 (Exporter/App) | 시스템 또는 애플리케이션이 `/metrics` endpoint에서 메트릭 생성 |
| 메트릭 수집기 (Prometheus, Grafana Alloy) | `/metrics`로부터 데이터를 주기적으로 가져옴 (Pull 방식) |
| 메트릭 저장소 (Grafana Mimir) | 수집된 메트릭을 장기 보관하고 쿼리할 수 있는 분산형 저장 시스템 |

> 핵심: Mimir는 저장소일 뿐, 직접 메트릭을 생성하거나 수집하지 않음. 수집기는 반드시 필요함.

---

## 2. Grafana Mimir

### Grafana Mimir
>- Prometheus와 완전 호환되는 확장형 메트릭 저장소  
>- 수평 확장, 고가용성, 멀티 테넌시 지원  
>- 외부 저장소(S3, GCS, Azure Blob 등) 연동 가능

### 구성 요소

| 컴포넌트 | 역할 |
|----------|------|
| Distributor | 수집기에서 전달된 데이터를 검증 및 분산 |
| Ingester | 메모리에 버퍼링 후 청크 저장 |
| Querier | 사용자의 PromQL 쿼리 처리 |
| Store-gateway | 장기 저장소에서 데이터 로딩 |
| Ruler | 알림 룰 평가 및 트리거 |

### 저장 방식

| 항목 | 설명 |
|------|------|
| 청크 | 실제 메트릭 값, 압축되어 저장됨 |
| 인덱스 | 라벨 및 시계열 메타데이터 저장 |

---

## 3. 메트릭 수집기: Prometheus vs Grafana Alloy

### Prometheus

>- 가장 널리 사용되는 메트릭 수집기  
>- `/metrics` endpoint를 pull 방식으로 수집  
>- `remote_write`를 통해 Mimir로 전송 가능

```yaml
scrape_configs:
  - job_name: 'myapp'
    static_configs:
      - targets: ['localhost:2112']

remote_write:
  - url: http://<mimir-host>:8080/api/v1/push
```

---

### Grafana Alloy

> Grafana가 만든 통합 수집기 (메트릭, 로그, 트레이스)
>- Prometheus 기능 포함 + 로그/트레이스도 수집 가능  
>- Agent, Collector 역할 모두 수행  
>- 수집한 메트릭을 Mimir로 전송 가능

```yaml
metrics:
  global:
    scrape_interval: 15s
  configs:
    - name: default
      scrape_configs:
        - job_name: 'my-go-app'
          static_configs:
            - targets: ['localhost:2112']
      remote_write:
        - url: http://<mimir-host>:8080/api/v1/push
```

---

## 4. Exporter 및 메트릭 생성

### Node Exporter

>- 리눅스 시스템 상태(CPU, 메모리, 디스크 등) 메트릭을 생성  
>- 대표적인 Exporter지만 필수는 아님

### 사용자 정의 애플리케이션 (Go, Python 등)

- 직접 Prometheus 클라이언트 라이브러리를 이용해 메트릭 생성  
- `/metrics` endpoint 제공 필수

---

## 5. 메트릭 수집 예제

gpt 작성

### Go 예제

```go
package main

import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
    "log"
    "net/http"
    "time"
)

var counter = prometheus.NewCounter(prometheus.CounterOpts{
    Name: "myapp_requests_total",
    Help: "Total requests",
})

func main() {
    prometheus.MustRegister(counter)
    http.Handle("/metrics", promhttp.Handler())

    go func() {
        for {
            counter.Inc()
            time.Sleep(5 * time.Second)
        }
    }()

    log.Println("Serving at :2112")
    log.Fatal(http.ListenAndServe(":2112", nil))
}
```

---

### Python (FastAPI 예시)

```python
from fastapi import FastAPI
from prometheus_client import Counter, generate_latest, CONTENT_TYPE_LATEST
from starlette.responses import Response

app = FastAPI()
REQUESTS = Counter("myapp_requests_total", "Total requests", ["method"])

@app.get("/")
def index():
    REQUESTS.labels(method="GET").inc()
    return {"message": "Hello, world!"}

@app.get("/metrics")
def metrics():
    data = generate_latest()
    return Response(content=data, media_type=CONTENT_TYPE_LATEST)
```

---

### Nginx 메트릭 수집 (nginx-prometheus-exporter 사용)

```bash
docker run -d --name nginx-exporter \
  -p 9113:9113 \
  -e NGINX_STATUS_URL=http://<nginx-host>/nginx_status \
  nginx/nginx-prometheus-exporter:latest
```

Prometheus 설정:

```yaml
scrape_configs:
  - job_name: 'nginx'
    static_configs:
      - targets: ['<exporter-host>:9113']
```

Nginx 설정 예시:

```nginx
location /nginx_status {
    stub_status;
    allow 127.0.0.1;
    deny all;
}
```

---

### MySQL 메트릭 수집 (mysqld_exporter 사용)

```bash
docker run -d --name mysqld-exporter \
  -p 9104:9104 \
  -e DATA_SOURCE_NAME="user:password@(localhost:3306)/" \
  prom/mysqld-exporter:latest
```

Prometheus 설정:

```yaml
scrape_configs:
  - job_name: 'mysql'
    static_configs:
      - targets: ['<exporter-host>:9104']
```

MySQL 사용자 생성:

```sql
CREATE USER 'exporter'@'%' IDENTIFIED BY 'password';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'%';
```

---

## 6. 핵심 비교 요약

gpt 작성

| 항목 | Grafana Mimir | Prometheus / Alloy |
|------|----------------|----------------------|
| 역할 | 메트릭 저장소 | 메트릭 수집기 |
| 메트릭 생성 여부 | 없음 (수신 전용) | 없음 (Exporter 또는 App 필요) |
| scrape 수행 여부 | 없음 | 있음 |
| remote_write 지원 | 있음 | 있음 |
| 고가용성 / 확장성 | 지원 | Prometheus는 제한적 |
| 멀티 테넌시 | 지원 | Prometheus는 미지원 |

---

## 7. 참고 자료

- https://grafana.com/docs/mimir/latest/  
- https://github.com/grafana/mimir  
- https://grafana.com/docs/alloy/latest/  
- https://prometheus.io/docs  
- https://github.com/prometheus/client_golang  
- https://github.com/prometheus/client_python