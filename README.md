# prom-loki-configs
contains the config files for tutorial at https://medium.com/@hr12rtk/keep-random-beacon-node-monitoring-grafana-prometheus-and-loki-4a4b669b31ea

# Summary of Instructions for Adding Keep Metrics to Grafana Dashboard
1. Update config.toml of random beacon client to enable metrics. The following entry in config.toml enables metrics at port 8080 of random beacon docker container.

```
[Metrics]
    Port = 8080
    NetworkMetricsTick = 60
    EthereumMetricsTick = 600
```
2. Add a new job in prometheus.yml for scraping keep metrics. Notice the port number is 8081. This port will be mapped to port 8080 of the docker container while starting random beacon
```
  - job_name: 'keep-metrics'
    scrape_interval: 60s
    static_configs:
         - targets: ['<IP/Hostname of Server Running Random Beacon>:8081']
```
3. Start random beacon docker container with addition port param mapping port 8081 with 8080 of docker container. Following are the commands for exporting environment variables and starting random beacon with additional port settings.

```
export KEEP_CLIENT_PERSISTENCE_DIR=~/<keep-client dir>/persistence
export KEEP_CLIENT_CONFIG_DIR=~/<keep-client dir>/config
export KEEP_CLIENT_KEYSTORE=~/<keep-client dir>/keystore
export KEEP_CLIENT_ETHEREUM_PASSWORD=<eth wallet password>
sudo docker run -dit \
--restart always \
--log-driver loki \
--log-opt loki-url="http://<IP|hostname of the server running Loki>:3100/loki/api/v1/push" \
--volume $HOME/keep-client:/mnt \
--env KEEP_ETHEREUM_PASSWORD=$KEEP_CLIENT_ETHEREUM_PASSWORD \
--env LOG_LEVEL=debug \
--name keep-client \
-p 3919:3919 \
-p 8081:8080 \
keepnetwork/keep-client:v1.3.0-rc --config /mnt/config/config.toml start
```
