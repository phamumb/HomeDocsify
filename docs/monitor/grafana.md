# Grafana
![grafana](assets/grafana.png)


## Resources
- Grafana dashboard ID: `16568`
- Influxdb hostname: `<INFLUXDB_HOST_IP>:8086`

## Docker Compose
```docker
version: "2.3"
services:
    grafana:
    container_name: grafana
    image: grafana/grafana-oss
    network_mode: "host"
    volumes:
        - <GRAFANA_CONFIG_DIR>:/var/lib/grafana
```

## Setup Data Source
