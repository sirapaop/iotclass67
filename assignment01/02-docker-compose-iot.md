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
Zookeeper: บริการนี้มีหน้าที่ในการจัดการและประสานงานแอปพลิเคชันแบบกระจาย เช่น Kafka โดยจะเก็บข้อมูลเกี่ยวกับโบรกเกอร์ของ Kafka และทำการจัดการเกี่ยวกับการเลือกผู้นำ (leader election)
Kafka: Apache Kafka เป็นแพลตฟอร์มสำหรับการสตรีมข้อมูลแบบกระจายที่ใช้สำหรับสร้างท่อข้อมูลแบบเรียลไทม์และแอปพลิเคชันสตรีมมิ่ง โดยสามารถจัดการข้อมูลขนาดใหญ่ได้อย่างทนทานต่อข้อผิดพลาด

## start-service #1
>> 
```
docker compose up kafka-rest-proxy kafka-connect mosquitto mongo grafana prometheus
```
Kafka REST Proxy: ให้บริการอินเทอร์เฟซแบบ RESTful สำหรับการทำงานกับ Kafka ทำให้ไคลเอนต์สามารถส่งและรับข้อความผ่าน HTTP
Kafka Connect: เครื่องมือที่ช่วยในการสตรีมข้อมูลระหว่าง Kafka และระบบอื่นๆ ทำให้ง่ายต่อการรวมข้อมูล
Mosquitto: ตัวกลาง (broker) ของ MQTT ที่ช่วยในการส่งข้อความแบบเบา (lightweight messaging) ระหว่างอุปกรณ์ในระบบ IoT โดยใช้รูปแบบการสื่อสารแบบ publish/subscribe
MongoDB: ฐานข้อมูลแบบ NoSQL ที่ใช้ในการจัดเก็บและเรียกคืนข้อมูลจำนวนมากที่ไม่ได้มีโครงสร้าง โดยส่วนมากจะใช้ในการเก็บข้อมูลจากเซนเซอร์ในระบบ IoT
Grafana: เครื่องมือวิเคราะห์และติดตามที่ใช้สำหรับการแสดงผลข้อมูลจากแหล่งข้อมูลต่างๆ เช่น Prometheus และ MongoDB
Prometheus: ระบบตรวจสอบและการแจ้งเตือนที่ใช้ในการรวบรวมข้อมูลจากบริการต่างๆ โดยปกติจะใช้ร่วมกับ Grafana สำหรับการแสดงผลข้อมูล

## start-service #2
>> 
```
docker compose up iot-processor
```
IoT Processor: บริการเฉพาะที่ทำหน้าที่ประมวลผลข้อมูลจากเซนเซอร์ในระบบ IoT โดยทั่วไปจะทำการดึงข้อมูลจาก Kafka หรือ MQTT และทำการคำนวณหรือแปลงข้อมูลก่อนที่จะส่งต่อผลลัพธ์ไปยังบริการอื่นๆ

## start-service #3
>> 
```
docker compose up iot_sensor_1
```
IoT Sensor 1: บริการที่จำลองหรือเก็บข้อมูลจริงจากอุปกรณ์ IoT โดยบริการนี้จะส่งข้อมูลเซนเซอร์ไปยัง IoT Processor ผ่านทาง MQTT หรือ Kafka เพื่อทำการวิเคราะห์ต่อไป