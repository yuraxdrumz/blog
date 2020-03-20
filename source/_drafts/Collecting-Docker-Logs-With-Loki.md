---
title: Collecting Docker Logs With Loki
tags:
  - Docker
  - Logging
  - Loki
  - Development
  - Containers
  - Tutorial
thumbnail: images/loki.jpg
category: DevOps
---
Loki is a multi-tenant log aggregation system inspired by Prometheus. 
It is cost effective, easy to operate and allows viewing logs directly in Grafana, thus avoiding running an expensive ELK stack.
In this blog post, I will show how to setup a Loki container using docker compose, how to define the loki logging driver to automatically ship all container logs to Loki and finally, how to use the logs in Grafana.

### Disclaimer
I have been using this setup for 3 months now with 24 services in a micro-services architecture with success.

### Setup Loki Container
First of all, we will need to define a loki container in the `docker-compose.yml`.

```yaml
  services:
    loki:
      container_name: loki
      image: grafana/loki:latest
      ports:
        - "3100:3100"
      command: -config.file=/etc/loki/local-config.yaml
      volumes:
        - ./volumes/loki/etc:/etc/loki
```

The command part tells loki to read the config from the `local-config.yaml` which we add as a volume.
The `local-config.yaml` is a default from the Loki github page. You can find more about the config [here](https://github.com/grafana/loki/blob/master/docs/configuration/README.md).

Here is an example of the config I use.
```yaml
auth_enabled: false

server:
  http_listen_port: 3100
  log_level: error

ingester:
  lifecycler:
    address: 127.0.0.1
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
    final_sleep: 0s
  chunk_idle_period: 5m
  chunk_retain_period: 30s

schema_config:
  configs:
    - from: 2018-04-15
      store: boltdb
      object_store: filesystem
      schema: v9
      index:
        prefix: index_
        period: 168h

storage_config:
  boltdb:
    directory: /tmp/loki/index

  filesystem:
    directory: /tmp/loki/chunks

limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h

chunk_store_config:
  max_look_back_period: 0

table_manager:
  chunk_tables_provisioning:
    inactive_read_throughput: 0
    inactive_write_throughput: 0
    provisioned_read_throughput: 0
    provisioned_write_throughput: 0
  index_tables_provisioning:
    inactive_read_throughput: 0
    inactive_write_throughput: 0
    provisioned_read_throughput: 0
    provisioned_write_throughput: 0
  retention_deletes_enabled: false
  retention_period: 0
```

### Install Loki Docker Plugin
After adding the container for Loki, we need to add the Loki logging driver as a docker plugin.
```shell script
  docker plugin install grafana/loki-docker-driver:latest --alias loki --grant-all-permissions
```
Don't forget to restart the docker daemon after installing the plugin.

If you are on Mac:
```shell script
	killall Docker && open /Applications/Docker.app
```
If you are on Linux:
```shell script
  sudo systemctl restart docker
```

After installing the plugin verify it is enabled.
```shell script
    docker plugin ls
```
Should output
```log
    ID                  NAME                DESCRIPTION           ENABLED
    412df8a79e8e        loki:latest         Loki Logging Driver   true
```

### Adding Loki as a Logging Driver
We want every container we add to our docker-compose file to send logs to Loki automatically. 
Add the logging section to each container
#### Docker-compose.yaml
```yaml
  logging:
    driver: loki
    options:
      loki-url: "http://host.docker.internal:3100/loki/api/v1/push"
```

An easier way since docker-compose 3.4 is anchors

```yaml
# logger driver - change this driver to ship all container logs to a different location
x-logging: &logging
  logging:
    driver: loki
    options:
      loki-url: "http://host.docker.internal:3100/loki/api/v1/push"
```
The `x-` tells docker compose to ignore this section. The `&` is a yml feature which allows referencing the entire section under it later in the file.

Here is an example with a reference to the &logging anchor we defined earlier.
```yaml
  services:
    my_service:
      *logging
      container_name: xxx
      image: xxx/xxx
```
#### Daemon.json
We can also change the default logging driver for all containers in `/etc/docker/daemon.json`.
I prefer adding it in the `docker-compose.yaml` directly, especially after anchor support came out.

### Running Docker with Loki
After you run `docker-compose up` all container logs will be sent to Loki.
One thing to note is, there is an error after running the containers, which says
```log
WARNING: no logs are available with the 'loki' log driver
```
But after checking Grafana I saw all logs are always sent to Loki, so I just ignore this message.

### Adding Loki to Grafana
Login to your Grafana instance and add a new data source of type Loki. Make sure the Grafana can reach the Loki instance
![](./adding-loki.png)

After that, go to explore and you should see all containers appear with automatic labels set by the Loki logging driver.

![](./grafana-loki.png)

You can also filter by words using the logql syntax, for example: `|=` includes syntax or `!=` excludes syntax. You can find out more about Loki logql syntax [here](https://github.com/grafana/loki/blob/master/docs/logql.md).

### Closing Notes
We saw how easy it is to define a Loki docker container, afterwards we added Loki logging driver as a docker plugin, and at last, we sent all of our logs to Grafana.
