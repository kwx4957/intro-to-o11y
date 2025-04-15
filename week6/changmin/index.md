# 옵저버빌리티: 메트릭 직접 구현해보기

## 메트릭이란 무엇인가?

메트릭은 시스템에서 발생하는 다양한 이벤트와 상태를 정량적으로 측정한 데이터

CPU 사용량, 메모리 사용량, 요청 응답 시간 등이 대표적인 예. 이러한 메트릭은 시스템의 성능을 모니터링하고 병목 현상을 발견하는 데 필수

## 메트릭 수집을 위한 설정

### 1. 애플리케이션 내 메트릭 수집 설정

먼저, 애플리케이션이 자체적으로 메트릭을 수집할 수 있도록 설정

예를 들어, Node.js 환경에서는 `prom-client` 라이브러리를 사용하여 메트릭을 수집할 수 있음.

**Node.js에서 Prometheus 메트릭 설정 예제**

```js
const client = require("prom-client");

// 메트릭 레지스트리 생성
const register = new client.Registry();

// 메트릭 생성
const httpRequestDuration = new client.Histogram({
  name: "http_request_duration_seconds",
  help: "HTTP 요청 응답 시간",
  labelNames: ["method", "route", "status_code"],
});

// 레지스트리에 메트릭 등록
register.registerMetric(httpRequestDuration);

// HTTP 요청에 대한 메트릭 측정
app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer();
  res.on("finish", () => {
    end({
      method: req.method,
      route: req.route?.path || req.path,
      status_code: res.statusCode,
    });
  });
  next();
});

// 메트릭 엔드포인트 생성
app.get("/metrics", async (req, res) => {
  res.setHeader("Content-Type", register.contentType);
  res.end(await register.metrics());
});
```

위 코드를 통해 애플리케이션에서 HTTP 요청 응답 시간과 같은 데이터를 수집할 수 있음.

### 2. Prometheus 설정

이제 수집된 메트릭을 백엔드 서버로 전송하는 설정을 진행

우선, Prometheus를 통해 메트릭을 수집

**Prometheus**
Prometheus 설정 파일(`prometheus.yml`)에서 메트릭을 수집할 애플리케이션의 엔드포인트를 지정

```yaml
scrape_configs:
  - job_name: "node_app"
    static_configs:
      - targets: ["localhost:3000"] # 애플리케이션의 메트릭 엔드포인트
```

### 3. docker-compose로 grafana와 prometheus 설정

```yaml
version: "3"
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3001:3000"
    depends_on:
      - prometheus
```

## 메트릭 수집 환경 설정하기

위에서 설명한 Prometheus 외에도 Grafana와 연계하여 메트릭을 시각화하거나, 다른 도구를 사용하여 원하는 환경에 맞게 메트릭 수집을 구현할 수 있음

**Grafana와 Prometheus 연동**

1. Grafana를 설치
2. Grafana 대시보드에서 Prometheus를 데이터 소스로 추가
3. 원하는 메트릭을 기반으로 그래프를 생성하여 시스템 상태를 시각화

## 결과

메트릭 설정이 완료되면, 애플리케이션을 실행하여 데이터가 제대로 수집되고 있는지 확인

Prometheus 대시보드에서 수집된 메트릭을 조회하거나, Grafana에서 시각화된 데이터를 검토할 수 있음

![](https://i.imgur.com/qCfxDIp.png)

## 마무리

1. 보다 더 다양한 메트릭 + 내가 실제로 모니터링하고 사용할 메트릭을 수집해보기
2. prometheus 외에 다양한 소스 연결하여 사용해보기
3. 분산환경에서 사용해보기
