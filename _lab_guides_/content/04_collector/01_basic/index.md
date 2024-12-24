## Basic configuration

### 📌 Task 1: Pipelines

Configure the traces and metrics service pipelines

You can find <mark>**otel-collector-config.yaml** </mark> in the directory

```
collector/otel-collector-config.yaml
```

Scroll down to the section called `service`. Make the necessary changes to activate the `traces` and `metrics` pipelines.

For the metrics pipeline enable the pre-configured processors `metricstransform` and `filter`.

<details>
  <summary>Expand to see solution</summary>

  <h4> For the traces pipeline </h4>

  ```yaml
      traces:
        receivers: [otlp]
        processors: [batch]
        exporters: [debug, otlphttp]
  ```

  <h4> For the metrics pipeline </h4>

  ```yaml
      metrics:
        receivers: [otlp]
        processors: [batch, metricstransform, filter]
        exporters: [debug, otlphttp]
  ```

</details>

### 📌 Task 2: Configure exporters

Reroute OpenTelemetry exporters to the OpenTelemetry collector

The <mark>**OTEL_EXPORTER_OTLP_ENDPOINT** </mark> environment variable will send ***all OpenTelemetry signals*** via OTLP (i.e. traces, metrics and logs) to the specified endpoint, without the need to configure each of them separately.

```bash
OTEL_EXPORTER_OTLP_ENDPOINT=http://opentelemetry-collector:4318
```

#### order-api service

The <mark>**Dockerfile**</mark> can be found in

```
order-api/Dockerfile
```

1. edit the dockerfile
1. uncomment line 35 to enable the collector endpoint
1. comment line 39 and 42 to disable the Dynatrace URL endpoint

#### order-backend service

The <mark>**Dockerfile**</mark> can be found in

```
order-backend/Dockerfile
```

1. edit the dockerfile
1. uncomment line 26 to enable the collector endpoint
1. comment line 34, 35, 39, 41 to disable the Dynatrace URL endpoint

#### Rebuild the containers

For the OpenTelemetry Collector, you will need to completely stop and remove all containers. To do so, run the command

```bash
docker compose down
```

Once all the containers are stopped, run the `compose up` command again.

```bash
docker compose up -d --build
```

### ✅ Verify results

- Launch the `Distributed trace` app to validate if you are still receving traces with the latest timestamps
- Launch a `Notebook` or `Dashboard` app, use the `Metric explorer` to search for the `shop` metrics
- What do you notice about the metrics? ***Hint:*** look at the metric names...

### 💡 Troubleshooting tips

Check log output of the OpenTelemetry collector container
```bash
docker logs opentelemetry-collector
```

Filtering logs for a specific string, example validating which exporters are activate
```bash
docker logs opentelemetry-collector 2>&1 | grep -i exporter
```

Expected output
```bash
2024-12-20T12:30:34.590Z        info    builders/builders.go:26 Development component. May change in the future.        {"kind": "exporter", "data_type": "metrics", "name": "debug"}
2024-12-20T12:30:34.591Z        info    builders/builders.go:26 Development component. May change in the future.        {"kind": "exporter", "data_type": "traces", "name": "debug"}
```