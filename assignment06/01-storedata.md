# Store data.
การจัดเก็บข้อมูล ในขั้นตอนนี้จะเน้นการย้ายข้อมูลที่ผ่านการประมวลผลจาก Kafka topics ไปยัง MongoDB collections เพื่อสามารถดูข้อมูลเหล่านี้ผ่านเครื่องมืออย่าง MongoDB-Express ได้ โดยมีการตั้งค่าตัวเชื่อมต่อ MongoDBSinkConnector ทั้งหมด 3 ตัว ซึ่งแต่ละตัวจะทำงานกับ topic ที่แตกต่างกันจาก Kafka และย้ายข้อมูลไปยัง collection ที่เกี่ยวข้องใน MongoDB

โดยเราจะใช้ MongoDB Sink Connector  ซึ่งเป็น connector ที่เราใช้ในการเขียนและส่ง ข้อมูลจาก Kafka topic ไปยัง database หรือ data system ที่เราต้องการได้ ซึ่งในงานนี้เราจะใช้ทั้งหมด 3 ตัวคือ 
- iot-frames 
- iot_aggregate_metric_sensor
- iot_aggregate_metrics_place

## ตัวเชื่อมต่อแรก
ตัวเชื่อมต่อนี้จะย้ายข้อมูลดั้งเดิมทั้งหมดจาก topic iot-frames ไปยัง collection iot_frames ในฐานข้อมูล iot โดยจะทำงานกับ JSON โดยไม่กำหนด schema เช่น Avro หรือ JSON-Schema เพื่อทำให้กระบวนการง่ายขึ้น
```
{
   "name":"iot-frames-mongodb-sink",
   "config":{
      "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
      "tasks.max":1,
      "topics":"iot-frames",
      "connection.uri":"mongodb://devroot:devroot@mongo:27017",
      "database":"iot",
      "collection":"iot_frames",
      "value.converter": "org.apache.kafka.connect.json.JsonConverter",
      "key.converter": "org.apache.kafka.connect.json.JsonConverter",
      "value.converter.schemas.enable": false,
      "key.converter.schemas.enable":false
   }
}
```
## ตัวเชื่อมต่อที่สอง
ตัวเชื่อมต่อนี้จะมีหน้าที่ในการย้ายข้อมูลที่ผ่านการประมวลผลใน topic iot-aggregate-metric-sensor ไปยัง collection iot_aggregate_metric_sensor
```
{
   "name":"iot-aggregate-metrics-sensor-mongodb-sink",
   "config":{
      "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
      "tasks.max":1,
      "topics":"iot-aggregate-metrics-sensor",
      "connection.uri":"mongodb://devroot:devroot@mongo:27017",
      "database":"iot",
      "collection":"iot_aggregate_metrics_sensor",
      "value.converter": "org.apache.kafka.connect.json.JsonConverter",
      "key.converter": "org.apache.kafka.connect.storage.StringConverter",
      "value.converter.schemas.enable": false,
      "key.converter.schemas.enable":false
   }
}
```
## ตัวเชื่อมต่อที่สาม
ตัวเชื่อมต่อนี้จะทำการย้ายข้อมูลที่ aggregate ตามสถานที่ใน topic iot-aggregate-metric-place ไปยัง collection iot_aggregate_metrics_place
```
{
   "name":"iot-aggregate-metrics-place-mongodb-sink",
   "config":{
      "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
      "tasks.max":1,
      "topics":"iot-aggregate-metrics-place",
      "connection.uri":"mongodb://devroot:devroot@mongo:27017",
      "database":"iot",
      "collection":"iot_aggregate_metrics_place",
      "value.converter": "org.apache.kafka.connect.json.JsonConverter",
      "key.converter": "org.apache.kafka.connect.storage.StringConverter",
      "value.converter.schemas.enable": false,
      "key.converter.schemas.enable":false
   }
}
```

## ตัวเชื่อมต่อสำหรับ Prometheus
นอกจาก MongoDB ยังมีการกำหนดตัวเชื่อมต่อเพื่อย้ายข้อมูลไปยัง Prometheus โดยจะดึงข้อมูลจาก topic iot-metric-time-series และทำให้สามารถเข้าถึงได้ผ่าน HTTP endpoint ที่ Prometheus จะดึงข้อมูลเข้ามา
```
{
  "name" : "prometheus-connector-sink",
  "config" : {
   "topics":"iot-metrics-time-series",
   "connector.class" : "io.confluent.connect.prometheus.PrometheusMetricsSinkConnector",
   "tasks.max" : "1",
   "confluent.topic.bootstrap.servers":"kafka:9092",
   "prometheus.scrape.url": "http://0.0.0.0:8084/iot-metrics-time-series",
   "prometheus.listener.url": "http://0.0.0.0:8084/iot-metrics-time-series",
   "value.converter": "org.apache.kafka.connect.json.JsonConverter",
   "key.converter": "org.apache.kafka.connect.json.JsonConverter",
   "value.converter.schemas.enable": false,
   "key.converter.schemas.enable":false,
   "reporter.bootstrap.servers": "kafka:9092",
   "reporter.result.topic.replication.factor": "1",
   "reporter.error.topic.replication.factor": "1",
   "behavior.on.error": "log"
  }
}
```
## ข้อจำกัดของตัวเชื่อมต่อ Prometheus:

- ไม่รองรับ Timestamp: Prometheus จะใช้ timestamp เมื่อดึงข้อมูล metric ขึ้นมา ข้อมูล timestamp ใน Kafka จะถูกละเลย
- ระบบดึงข้อมูล: Prometheus ใช้ระบบดึงข้อมูล (Pull-based) โดยตัวเชื่อมต่อจะสร้าง HTTP server ที่ worker node เพื่อให้ Prometheus ดึงข้อมูลผ่าน HTTP endpoint