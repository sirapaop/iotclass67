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

### Starting sever
- if you start the sever for the first time skip step 1 and follow step 2.
- if your sever is not running you need to follow step 1 and then follow step 2.

#### Step 1
Stop all container
```
docker stop $(docker ps -aq)
```

Delete all containers
```
docker container rm -f $(docker container ls -aq)
```

Delete all volumes
```
docker volume rm -f $(docker volume ls -q)
```

Delete network
```
docker network rm -f $(docker network ls -q)
```
#### Step 2
```
docker compose up zookeeper kafka
```
read the logs before start the next step
```
docker compose up kafka-rest-proxy kafka-connect mosquitto mongo grafana prometheus
```
read the logs before start the next step
```
docker compose up iot-processor
```
if the logs : start fail stop the iot-processor and restart again
continue to the next step after the state done
```
docker compose up iot_sensor_1
```


# IoT Docker compose
>> ให้นำไฟล์ docker-compose.yaml มาอธิบายว่า แต่ละส่วนคืออะไร โดยใช้การ comment ในไฟล์ docker-compose.yaml


## start-service #0
>> 
```
docker compose up zookeeper kafka
```

อธิบายว่า  หน้าจอที่ 1 ที่ต้องเปิดใช้งาน มีการเปิด service อะไรบ้างใน docker

## start-service #1
>> 
```
docker compose up zookeeper kafka
```

อธิบายว่า  หน้าจอที่ 1 ที่ต้องเปิดใช้งาน มีการเปิด service อะไรบ้างใน docker

## start-service #2
>> 
```
docker compose up zookeeper kafka
```

อธิบายว่า  หน้าจอที่ 1 ที่ต้องเปิดใช้งาน มีการเปิด service อะไรบ้างใน docker

## start-service #3
>> 
```
docker compose up zookeeper kafka
```

อธิบายว่า  หน้าจอที่ 1 ที่ต้องเปิดใช้งาน มีการเปิด service อะไรบ้างใน docker