## Otel을 통한 Trace 구현하기
## 구성도
```mermaid
flowchart LR
    subgraph Application
        A1[App]
    end

    subgraph OTEL_Collector
        direction TB
        B1[Receiver]
        B2[Processors]
        B3[Exporters]
    end

    subgraph Backends
        C1[Metrics Backend]
        C2[Logs Backend]
        C3[Traces Backend]
    end

    subgraph Visualization
        D1[Grafana]
    end

    A1 -->|Metric| OTEL_Collector
    A1 -->|Log| OTEL_Collector
    A1 -->|Trace| OTEL_Collector

    B1 --> B2
    B2 --> B3

    OTEL_Collector --> C1
    OTEL_Collector --> C2
    OTEL_Collector --> C3

    C1 <--> Visualization
    C2 <--> Visualization
    C3 <--> Visualization
```

## Otel Operator 
```sh
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update

# Opterator
helm upgrade --install my-opentelemetry-operator open-telemetry/opentelemetry-operator \
  --set "manager.collectorImage.repository=otel/opentelemetry-collector-k8s" \
  --set admissionWebhooks.certManager.enabled=false \
  --set admissionWebhooks.autoGenerateCert.enabled=true \
  --set manager.extraArgs[0]="--enable-nginx-instrumentation=true"
```

## Otel Collector  
- daemonset
- deployement
- statefulset

```sh
helm upgrade --install my-opentelemetry-collector open-telemetry/opentelemetry-collector \
  --set mode=daemonset \
  --set "image.repository=otel/opentelemetry-collector-k8s"
  --set presets.logsCollection.enabled=true \
  --set presets.kubernetesAttributes.enabled=true \
  --set presets.kubeletMetrics.enabled=true \
  --set presets.kubeletMetrics.enabled=true \
  --set exporters.otlp.endpoint= \ 
```

## Init Container
```mermaid
flowchart TB
    subgraph Init Container
        Init1[otel-agent-source-container-clone]
        Init2[otel-agent-attach-nginx]
    end
    subgraph Otel Operator
        I1[Instrumentation CRD]
    end

    subgraph Application
        direction TB
        Nginx[Nginx Container]
    end

    I1 -->|Inject init container| Nginx
    Init1 --> Nginx
    Init2 --> Nginx
```
## Instrumentation
```sh
kubectl apply -f - <<EOF
apiVersion: opentelemetry.io/v1alpha1
kind: Instrumentation
metadata:
  name: nginx-instrumentation
spec:
  exporter:
    endpoint: http://tempo:4317
  propagators:
    - tracecontext
    - baggage
    - b3
  sampler:
    type: parentbased_traceidratio
    argument: "0.25"
  nginx:
    attrs:
    - name: NginxModuleOtelMaxQueueSize
      value: "4096"
EOF
```

## Nginx App
```sh
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        instrumentation.opentelemetry.io/inject-nginx: "true" 
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.3
        ports:
        - containerPort: 80
EOF

ls /etc/nginx/conf.d/
default.conf  opentelemetry_agent.conf

cat /etc/nginx/conf.d/opentelemetry_agent.conf
NginxModuleEnabled ON;
NginxModuleOtelExporterEndpoint http://tempo:4317;
NginxModuleOtelMaxQueueSize 4096;
NginxModuleOtelSpanExporter otlp;
NginxModuleResolveBackends ON;
NginxModuleServiceInstanceId nginx-66bddcd74d-7tcgp;
NginxModuleServiceName nginx;
NginxModuleServiceNamespace default;
NginxModuleTraceAsError ON;
```
