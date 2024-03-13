## Microservice

``` xml title="pom.xml""
<!-- metricas de uso -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<!-- exporta no formato prometheus -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <scope>runtime</scope>
</dependency>
```

``` yaml title="application.yaml"
management:
  endpoints:
    web:
      base-path: /gateway/actuator
      exposure:
        include: [ 'prometheus' ]
```

## Docker

``` yaml title="docker-compose.yaml"
  prometheus:
    image: prom/prometheus:latest
    container_name: store-prometheus
    ports:
      - 9090:9090
    volumes:
      - $VOLUME/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    networks:
      - private-network

  grafana:
    container_name: store-grafana
    image: grafana/grafana-enterprise
    ports:
      - 3000:3000
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - $VOLUME/grafana:/var/lib/grafana
      - $VOLUME/grafana/provisioning/datasources:/etc/grafana/provisioning/datasources      
    restart: always
    networks:
      - private-network
```


## [Prometheus](https://prometheus.io){target="_blank"}

``` yaml title="$VOLUME/prometheus/prometheus.yml"
scrape_configs:

  - job_name: 'GatewayMetrics'
    metrics_path: '/gateway/actuator/prometheus'
    scrape_interval: 1s
    static_configs:
      - targets:
        - host.docker.internal:8443
        labels:
          application: 'Gateway Application'
```

[http://localhost:9090/](http://localhost:9090/){target="_blank"}

## [Grafana](https://grafana.com){target="_blank"}

``` yaml title="$VOLUME/grafana/provisioning/datasources/datasourse.yml"
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
```

[http://localhost:3000/](http://localhost:3000/){target="_blank"}

- [Dashboard MarketPlace](https://grafana.com/grafana/dashboards/){target="_blank"}
