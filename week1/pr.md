## Summary
This is for briefly studying what observability is with hands-on labs.
Hands-on labs first to get experiences on Grafana and understand what it provides.
The first week focused on setting up and exploring various observability tools through a series of labs designed to provide practical experience with Grafana and its integrated services.
- https://github.com/grafana/intro-to-mltp?tab=readme-ov-file#grafana for Labs 

## Introduction
By forking the intro-to-mltp repository, I accessed pre-configured labs that allowed me to interact directly with tools such as Grafana, Mimir, Loki, and others. These labs served as an introduction to the capabilities of each tool in monitoring and visualizing data across different dimensions of observability — metrics, logs, traces, and profiles.

**The labs include services such as:**

### Grafana :: web application
Grafana is a popular open-source platform for monitoring and observability(web application). It provides powerful and customizable dashboards for visualizing time-series data, allowing users to track logs, metrics, and traces from various data sources like Prometheus and Elasticsearch. Grafana is widely used for its extensive support for different databases and its capability to create and share visual analytics across teams.

### Mimir :: backend store
Mimir (formerly known as Cortex) is a scalable Prometheus implementation that provides highly available and long-term storage for time-series data. Mimir enables multi-tenancy and sharding to handle high volumes of data across multiple Prometheus servers, making it suitable for large-scale deployments.

### Loki :: backend store
Loki is a horizontally scalable, highly available, multi-tenant log aggregation system inspired by Prometheus. It is designed to be very cost-effective and easy to operate. Instead of indexing the content of the logs, it indexes their metadata and allows storing logs in their compressed format. Loki is particularly good at pairing logs with Prometheus metrics and Grafana dashboards for a unified observability stack.

### Tempo :: backend store
Tempo is a distributed tracing backend that is cost-effective and requires only object storage to operate. It integrates with Prometheus and Grafana, providing full observability from metrics over logs to traces. Tempo supports high-throughput environments and is designed to capture, store, and retrieve traces for service architectures.

### Pyroscope :: backend store
Pyroscope is an open-source continuous profiling platform. It helps you analyze and understand how your software performs in production, making it easier to diagnose performance issues across various environments and code versions. Pyroscope can be integrated with other observability tools to correlate performance data with metrics and logs.

### k6 :: load testing suite
k6 is a modern load testing tool, built for engineering teams to test the performance and reliability of their systems. It is scriptable in JavaScript, allowing complex tests to simulate multiple users interacting with a system concurrently. k6 integrates with Grafana to visualize the impact of tests on system performance.

### Beyla
Beyla is an eBPF-based tool that can generate metrics and trace data without requiring changes to application code. It uses Linux kernel capabilities to capture and generate observability data, which can then be sent to systems like Grafana for visualization.

### Grafana Alloy
Grafana Alloy is a configuration of the Grafana Agent, designed to handle metrics, logs, and traces. It functions as a lightweight data collector that can be deployed close to application instances. Alloy supports various processing rules and sends collected data to long-term storage solutions like Loki, Mimir, and Tempo.

## Description
[Steps for labs]
- **Start docker compose provided by Grafana Git**
![docker compose up](<Screenshot 2025-03-11 at 4.10.57 PM.png>)
- **Explore local grafna monitoring tools with services such as mimir, pyroscope, loki, etc.**
![grafana_mimir](<Screenshot 2025-03-11 at 4.13.35 PM.png>)
![grafana_pyroscope](<Screenshot 2025-03-11 at 4.28.19 PM.png>)
- **Stopped docker compose**
![stopping docker](<Screenshot 2025-03-11 at 4.32.31 PM.png>)

## Refloctions
Although I successfully set up the observability stack and explored what Grafana provides, I still find myself grappling with the depth of "observability" as a concept. The labs showed how tools like Grafana facilitate observability but understanding the underlying mechanisms of how these tools capture, process, and visualize data will require deeper exploration. The interaction with the tools provided a good starting point, but the technical intricacies of how these tools interconnect and function still need clearer elucidation.

## Next Steps
[Action Items to do]
- **Deepen Technical Understanding**: Enhance my understanding of how each component of the observability stack works, particularly how they collect and aggregate data.
- **Understanding Grafana Querying**: Develop your skills in crafting and optimizing queries within Grafana. This includes using PromQL for Prometheus data, LogQL for Loki logs, and exploring the query functionalities provided for other data sources integrated with Grafana.
- **Explore Grafana Cloud**: Engage with the Grafana Cloud environments to understand the differences and benefits it offers compared to a local Grafana setup.