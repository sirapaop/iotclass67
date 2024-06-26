# IoT Docker compose


## How to start docker compose
1. cd into your gitHub folder
```bash
2. sudo docker compose up
```

## Error we found 
1. Zookeeper on docker_compose.yml 
2. root error c:/rootfs:ro0 on docker_compose.yml  
3. don't have permission for ./grafana/setup.sh:/setup.sh 

## How to solve the problems.
1. comment all zookeeper line(201-235) on docker_compose.yml
2. change all c:/rootfs:ro0 to /:/rootfs:ro because it not window on docker_compose.yml
3. Allow permission by cd in Gramfana folder and type chmod 777 setup.sh

## Output

- [ ] IoT Sensor - Dashboards - Grafana 
- [ ] UI for Apache Ka
- [ ] Mongo Expr
- [ ] Node Expor
- [ ] Prometheus Time Series Collection and Processing Ser
- [ ] Prometheus Pushgateway
- [ ] ZooNavigator


### IoT Sensor - Dashboards - Grafana URL
1. check your ip address
2. check your user and password for Grafana (user: admin, password: admin)
3. open browser and type in your ip address:8085 for example = 172.20.49.244:8085

### UI for Apache Kafka

### Mongo Express

### Node Exporter

### Prometheus Time Series Collection and Processing Server

### Prometheus Pushgateway

### ZooNavigator
