# IoT Docker compose
>>
```
  # IoT Sensor 1
  iot_sensor_1:
    # image: ssanchez11/iot_sensor:0.0.1-SNAPSHOT
    build:
      context: ./microservices/iot_sensor
      args:
        - MQTT_SERVER=${MQTT_SERVER}
    container_name: iot_sensor_1
    restart: unless-stopped
    environment:
      - sensor.id=${IOT_SENSOR_1_ID}
      - sensor.name=${IOT_SENSOR_1_NAME}
      - sensor.place.id=${IOT_SENSOR_1_PLACE_ID}
      - sensor.mqtt.username=${IOT_SENSOR_1_USERNAME}
      - sensor.mqtt.password=${IOT_SENSOR_1_PASSWORD}
      - MQTT_SERVER=${MQTT_SERVER}
    depends_on:
      iot-processor:
        condition: service_started
        restart: true

  # # # # IoT Sensor 2
  # iot_sensor_2:
  #   image: ssanchez11/iot_sensor:0.0.1-SNAPSHOT
  #   container_name: iot_sensor_2
  #   restart: unless-stopped
  #   environment:
  #     - sensor.id=${IOT_SENSOR_2_ID}
  #     - sensor.name=${IOT_SENSOR_2_NAME}
  #     - sensor.place.id=${IOT_SENSOR_2_PLACE_ID}
  # # # # IoT Sensor 1

  # iot_sensor_3:
  #   image: ssanchez11/iot_sensor:0.0.1-SNAPSHOT
  #   container_name: iot_sensor_3
  #   restart: unless-stopped
  #   environment:
  #     - sensor.id=${IOT_SENSOR_3_ID}
  #     - sensor.name=${IOT_SENSOR_3_NAME}
  #     - sensor.place.id=${IOT_SENSOR_3_PLACE_ID}

  # IoT Processor
  iot-processor:
    image: ssanchez11/iot_processor:0.0.1-SNAPSHOT
    container_name: iot-processor
    restart: unless-stopped
    ports:
      - '8080:8080'
    depends_on:
      kafka-connect:
        condition: service_started
        restart: true
```
## start-service #0
>> 
```
docker compose up zookeeper kafka
```
zookeeper: มีหน้าที่ในการจัดการ application แบบกระจายตัว
kafka: Apache Kafka เป็น streaming platform ที่สร้าง real-time data pipelines และ streaming applications. โดยสามารถจัดการข้อมูลขนาดใหญ่ได้

## start-service #1
>> 
```
docker compose up kafka-rest-proxy kafka-connect mosquitto mongo grafana prometheus
```
kafka-rest-proxy: เรียกใช้งาน interface แบบ RESTful สำหรับการทำงานกับ Kafka ทำให้สามารถส่งและรับข้อความผ่าน HTTP ได้
kafka-connect: เป็นเครื่องมือที่ช่วยในการ streaming ข้อมูลระหว่าง Kafka และระบบอื่นๆ
mosquitto: เป็น broker ของ MQTT ที่ช่วยในการส่งข้อมูล ระหว่างอุปกรณ์ในระบบ IoT โดยใช้การ publish/subscribe
mongo: เป็น database แบบ NoSQL ที่ใช้ในการจัดเก็บและเรียกคืนข้อมูล
grafana: เป็นเครื่องมือที่ใช้วิเคราะห์และติดตามที่ใช้สำหรับการแสดงผลข้อมูลจากแหล่งข้อมูลต่างๆ เช่น Prometheus และ MongoDB
prometheus: ระบบตรวจสอบและการแจ้งเตือนที่ใช้ในการรวบรวมข้อมูลจากบริการต่างๆ โดยปกติจะใช้ร่วมกับ Grafana สำหรับการแสดงผลข้อมูล

## start-service #2
>> 
```
docker compose up iot-processor
```
iot-processor: เป็นบริการที่ทำหน้าที่ประมวลผลข้อมูลจาก sensor ในระบบ

## start-service #3
>> 
```
docker compose up iot_sensor_1
```
iot_sensor_1: เป็นบริการที่จำลองข้อมูลจากอุปกรณ์ IoT โดยบริการนี้จะส่งข้อมูลเซนเซอร์ไปยัง IOT Processor ผ่านทาง MQTT

## Error we found 
1. Zookeeper on docker_compose.yml 
2. root error c:/rootfs:ro0 on docker_compose.yml  
3. don't have permission for ./grafana/setup.sh:/setup.sh 

## How to solve the problems.
1. comment all zookeeper line(201-235) on docker_compose.yml
2. change all c:/rootfs:ro0 to /:/rootfs:ro because it not window on docker_compose.yml
3. Allow permission by cd in Gramfana folder and type chmod 777 setup.sh