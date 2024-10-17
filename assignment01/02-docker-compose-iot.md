# IoT Docker compose
>> ให้นำไฟล์ docker-compose.yaml มาอธิบายว่า แต่ละส่วนคืออะไร โดยใช้การ comment ในไฟล์ docker-compose.yaml
```
  volumes:
    prometheus_data: {}
    grafana_data: {}
    zookeeper-data:
      driver: local
    zookeeper-log:
      driver: local
    kafka-data:
      driver: local

services:

  # ZooKeeper เป็นบริการที่เป็นตัวกลางสำหรับการจัดการข้อมูลการกำหนดค่า, การตั้งชื่อ, การกระจาย, และการให้บริการกลุ่ม
  # ให้การประสานงานแบบกระจายสำหรับ Kafka cluster ของเรา
  zookeeper:
    image: confluentinc/cp-zookeeper
    container_name: zookeeper
    # ZooKeeper is designed to "fail-fast", so it is important to allow it to
    # restart automatically.
    restart: unless-stopped
    volumes:
      - zookeeper-data:/var/lib/zookeeper/data
      - zookeeper-log:/var/lib/zookeeper/log
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_LOG4J_ROOT_LOGLEVEL: INFO
      ZOOKEEPER_LOG4J_PROP: INFO,ROLLINGFILE
      ZOOKEEPER_LOG_MAXFILESIZE: 10MB
      ZOOKEEPER_LOG_MAXBACKUPINDEX: 10
      ZOOKEEPER_SNAP_COUNT: 10
      ZOOKEEPER_AUTOPURGE_SNAP_RETAIN_COUNT: 10
      ZOOKEEPER_AUTOPURGE_PURGE_INTERVAL: 3


  # Kafka เป็นแพลตฟอร์มการสตรีมข้อมูลแบบกระจาย ใช้เพื่อสร้าง pipeline แบบ real-time
  # สามารถสร้างแอปพลิเคชันการสตรีมข้อมูลแบบ real-time
  kafka:
    image: confluentinc/cp-kafka
    container_name: kafka
    volumes:
      - kafka-data:/var/lib/kafka
    restart: unless-stopped
    environment:
      # Required. Instructs Kafka how to get in touch with ZooKeeper.
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_NUM_PARTITIONS: 1
      KAFKA_COMPRESSION_TYPE: gzip
      # Required when running in a single-node cluster, as we are. We would be able to take the default if we had
      # three or more nodes in the cluster.
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      # Required. Kafka will publish this address to ZooKeeper so clients know
      # how to get in touch with Kafka. "PLAINTEXT" indicates that no authentication
      # mechanism will be used.
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
    links:
      - zookeeper


  # Kafka REST Proxy ให้การเชื่อมต่อผ่าน RESTful กับ Kafka cluster
  # ทำให้การผลิตและบริโภคข้อความ, ดูสถานะของ cluster, และดำเนินการจัดการโดยไม่ต้องใช้
  # โปรโตคอลหรือไคลเอนต์ Kafka แบบดั้งเดิม
  kafka-rest-proxy:
    image: confluentinc/cp-kafka-rest:latest
    container_name: kafka-rest-proxy
    environment:
      # Specifies the ZooKeeper connection string. This service connects
      # to ZooKeeper so that it can broadcast its endpoints as well as
      # react to the dynamic topology of the Kafka cluster.
      KAFKA_REST_ZOOKEEPER_CONNECT: zookeeper:2181
      # The address on which Kafka REST will listen for API requests.
      KAFKA_REST_LISTENERS: http://0.0.0.0:8082/
      # Required. This is the hostname used to generate absolute URLs in responses.
      # It defaults to the Java canonical hostname for the container, which might
      # not be resolvable in a Docker environment.
      KAFKA_REST_HOST_NAME: kafka-rest-proxy
      # The list of Kafka brokers to connect to. This is only used for bootstrapping,
      # the addresses provided here are used to initially connect to the cluster,
      # after which the cluster will dynamically change. Thanks, ZooKeeper!
      KAFKA_REST_BOOTSTRAP_SERVERS: kafka:9092
    # Kafka REST relies upon Kafka, ZooKeeper
    # This will instruct docker to wait until those services are up
    # before attempting to start Kafka REST.
    restart: unless-stopped
    ports:
      - "9999:8082"
    depends_on:
      - zookeeper
      - kafka

  # Kafka Connect เป็นส่วนประกอบโอเพ่นซอร์สของ Apache Kafka เช่น databases, key-value stores หรือ file systems
  kafka-connect:
    image: confluentinc/cp-kafka-connect:latest
    hostname: kafka-connect
    container_name: kafka-connect
    environment:
      # Required.
      # The list of Kafka brokers to connect to. This is only used for bootstrapping,
      # the addresses provided here are used to initially connect to the cluster,
      # after which the cluster can dynamically change. Thanks, ZooKeeper!
      CONNECT_BOOTSTRAP_SERVERS: "kafka:9092"
      # Required. A unique string that identifies the Connect cluster group this worker belongs to.
      CONNECT_GROUP_ID: kafka-connect-group
      # Connect will actually use Kafka topics as a datastore for configuration and other data. #meta
      # Required. The name of the topic where connector and task configuration data are stored.
      CONNECT_CONFIG_STORAGE_TOPIC: kafka-connect-meta-configs
      # Required. The name of the topic where connector and task configuration offsets are stored.
      CONNECT_OFFSET_STORAGE_TOPIC: kafka-connect-meta-offsets
      # Required. The name of the topic where connector and task configuration status updates are stored.
      CONNECT_STATUS_STORAGE_TOPIC: kafka-connect-meta-status
      # Required. Converter class for key Connect data. This controls the format of the
      # data that will be written to Kafka for source connectors or read from Kafka for sink connectors.
      CONNECT_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      # Required. Converter class for value Connect data. This controls the format of the
      # data that will be written to Kafka for source connectors or read from Kafka for sink connectors.
      CONNECT_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      # Required. The hostname that will be given out to other workers to connect to.
      CONNECT_REST_ADVERTISED_HOST_NAME: "kafka-connect"
      CONNECT_REST_PORT: 8083
      # The next three are required when running in a single-node cluster, as we are.
      # We would be able to take the default (of 3) if we had three or more nodes in the cluster.
      CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: "1"
      CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: "1"
      CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: "1"
      #Connectos path
      CONNECT_PLUGIN_PATH: "/usr/share/java,/data/connectors/"
      CONNECT_LOG4J_ROOT_LOGLEVEL: "INFO"
    restart: unless-stopped
    volumes:
      - ./kafka_connect/data:/data
    command: 
      - bash 
      - -c 
      - |
        echo "Launching Kafka Connect worker"
        /etc/confluent/docker/run & 
        #
        echo "Waiting for Kafka Connect to start listening on http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors ⏳"
        while [ $$(curl -s -o /dev/null -w %{http_code} http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors) -ne 200 ] ; do 
          echo -e $$(date) " Kafka Connect listener HTTP state: " $$(curl -s -o /dev/null -w %{http_code} http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors) " (waiting for 200)"
          sleep 5 
        done
        nc -vz $$CONNECT_REST_ADVERTISED_HOST_NAME $$CONNECT_REST_PORT
        echo -e "\n--\n+> Creating Kafka Connect MongoDB sink Current PATH ($$PWD)"
        /data/scripts/create_mongo_sink.sh 
        echo -e "\n--\n+> Creating MQTT Source Connect Current PATH ($$PWD)"
        /data/scripts/create_mqtt_source.sh
        echo -e "\n--\n+> Creating Kafka Connect Prometheus sink Current PATH ($$PWD)"
        /data/scripts/create_prometheus_sink.sh
        sleep infinity
    # kafka-connect relies upon Kafka and ZooKeeper.
    # This will instruct docker to wait until those services are up
    # before attempting to start kafka-connect.
    depends_on:
      - zookeeper
      - kafka

  # Mosquitto เป็น broker ของ MQTT protocol ใช้สำหรับส่งข้อมูล
  mosquitto:
    image: eclipse-mosquitto:latest
    hostname: mosquitto
    container_name: mosquitto
    restart: unless-stopped
    ports:
      - "1883:1883"
      - "9001:9001"
    volumes:
      - ./mosquitto/config:/mosquitto/config

  # MongoDB เป็น data base ที่ใช้เก็บข้อมูล
  mongo:
    image: mongo:4.4.20
    container_name: mongo
    env_file:
      - .env
    restart: unless-stopped
    environment:
      - MONGO_INITDB_ROOT_USERNAME=${MONGO_ROOT_USER}
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_ROOT_PASSWORD}
      - MONGO_INITDB_DATABASE=${MONGO_DB}
  
  # Grafana เป็นแอปพลิเคชันโอเพ่นซอร์สสำหรับการวิเคราะห์และการแสดงผลแบบโต้ตอบ เช่น กราฟ และแผนภูมิ
  # สามารถโหลด plug-in เพิ้มได้
  grafana:
    image: grafana/grafana:latest-ubuntu
    container_name: grafana
    user: '0'
    volumes:
      - ./grafana/data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
      - ./grafana/data/plugins:/var/lib/grafana/plugins
     
    environment:
      - GF_SECURITY_ADMIN_USER=${ADMIN_USER:-admin}
      - GF_SECURITY_ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin}
      # - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-worldmap-panel,grafana-piechart-panel
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_SECURITY_ANGULAR_SUPPORT_ENABLED=True
      - GF_FEATURE_TOGGLES_ANGULARDEPRECATIONUI=False
    restart: unless-stopped
    links:
       - prometheus
    ports:
      - '8085:3000'
  
  # Prometheus เป็นแอปพลิเคชันซอฟต์แวร์ฟรีสำหรับการตรวจสอบเหตุการณ์และการแจ้งเตือน
  # บันทึกข้อมูลเมตริกแบบเรียลไทม์ในฐานข้อมูล time series โดยใช้โมเดล HTTP pull
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus/:/etc/prometheus/
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=200h'
      - '--web.enable-lifecycle'
    restart: unless-stopped
    ports:
      - '8086:9090'
 

  # IoT Sensor 1 เป็น service ที่จำลองการทำงานของ sensor ที่อยู่ในตัว server
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

  # IoT Processor เป็น service ที่ประมวลผลข้อมูล และ ส่งไปที่ Kafka
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
sh start_0zookeeper_kafka.sh
```
โดยคำสั่ง
```
docker compose up zookeeper kafka
```
จะเป็นการ start 2 service 
- zookeeper มีหน้าที่ในการจัดการ application แบบกระจายตัว
- kafka เป็น streaming platform ที่สร้าง real-time data pipelines และ streaming applications. โดยสามารถจัดการข้อมูลขนาดใหญ่ได้

## start-service #1
>> 
```
sh start_1kafka_service.sh
```
โดยคำสั่ง
```
docker compose up kafka-rest-proxy kafka-connect mosquitto mongo grafana prometheus
```
จะเป็นการ start 6 service 
- kafka-rest-proxy เป็นการเรียกใช้งาน interface แบบ RESTful สำหรับการทำงานกับ Kafka ทำให้สามารถส่งและรับข้อความผ่าน HTTP ได้
- kafka-connect เป็นเครื่องมือที่ช่วยในการ streaming ข้อมูลระหว่าง Kafka และระบบอื่นๆ
- mosquitto เป็น broker ของ MQTT ที่ช่วยในการส่งข้อมูล ระหว่างอุปกรณ์ในระบบ IoT โดยใช้การ publish/subscribe
- mongo เป็น database แบบ NoSQL ที่ใช้ในการจัดเก็บและเรียกคืนข้อมูล
- grafana เป็นเครื่องมือที่ใช้วิเคราะห์และติดตามที่ใช้สำหรับการแสดงผลข้อมูลจากแหล่งข้อมูลต่างๆ เช่น Prometheus และ MongoDB
- prometheus เป็นระบบตรวจสอบและการแจ้งเตือนที่ใช้ในการรวบรวมข้อมูลจากบริการต่างๆ โดยปกติจะใช้ร่วมกับ Grafana สำหรับการแสดงผลข้อมูล

## start-service #2
>> 
```
sh start_2iot_processor.sh
```
โดยคำสั่ง
```
docker compose up iot-processor
```
จะเป็นการ start 1 service 
- iot-processor เป็นบริการที่ทำหน้าที่ประมวลผลข้อมูลจาก sensor ในระบบ

## start-service #3
>> 
```
sh start_3iot_sensor.sh
```
โดยคำสั่ง
```
docker compose up iot_sensor_1
```
จะเป็นการ start 1 service 
- iot_sensor_1 เป็นบริการที่จำลองข้อมูลจากอุปกรณ์ IoT โดยบริการนี้จะส่งข้อมูลเซนเซอร์ไปยัง IOT Processor ผ่านทาง MQTT

## Error we found 
1. Zookeeper on docker_compose.yml 
2. root error c:/rootfs:ro0 on docker_compose.yml  
3. don't have permission for ./grafana/setup.sh:/setup.sh 

## How to solve the problems.
1. comment all zookeeper line(201-235) on docker_compose.yml
2. change all c:/rootfs:ro0 to /:/rootfs:ro because it not window on docker_compose.yml
3. Allow permission by cd in Gramfana folder and type chmod 777 setup.sh