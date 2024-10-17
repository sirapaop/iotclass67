# MQTT
## วิธี Run MQTT 
```
docker compose up mosquitto
```
## วิธี Check MQTT status
```
docker compose logs -f mosquitto
```
## วิธี restart MQTT
```
docker compose restart mosquitto
```

## iot-sensor-1
>> เป็น sensor ที่จำลองค่าที่อยู่ภายในเครื่อง server ของตนเอง
```
    IOT_SENSOR_1_ID=12434345
    IOT_SENSOR_1_NAME=iot_sensor_11
    IOT_SENSOR_1_PLACE_ID=32347983
    IOT_SENSOR_1_USERNAME=iot-wangs-11
    IOT_SENSOR_1_PASSWORD=12345
```

## iot-sensor-2
>> เป็น sensor ที่จำลองค่าที่อยู่ภายในเครื่องที่ต่ออยู่ใน Network วง LAN เดียวกัน
```
    IOT_SENSOR_2_ID=21321434
    IOT_SENSOR_2_NAME=iot-wangs-12
    IOT_SENSOR_2_PLACE_ID=32347983
    IOT_SENSOR_2_USERNAME=iot-wangs-12
    IOT_SENSOR_2_PASSWORD=12345
```

## iot-sensor-3-10
>> เป็น sensor ที่รับค่าจาก borad cucumber จากเพื่อนจำนวน 7 ตัวที่ subcribe mqtt topic และอยู่ใน Network วง LAN เดียวกัน
```
    IOT_SENSOR_3_ID=43245253
    IOT_SENSOR_3_NAME=iot-wangs-13
    IOT_SENSOR_3_PLACE_ID=32347983
    IOT_SENSOR_3_USERNAME=iot-wangs-13
    IOT_SENSOR_3_PASSWORD=12345

    IOT_SENSOR_4_ID=43245254
    IOT_SENSOR_4_NAME=iot-wangs-14
    IOT_SENSOR_4_PLACE_ID=32347983
    IOT_SENSOR_4_USERNAME=iot-wangs-14
    IOT_SENSOR_4_PASSWORD=12345


    IOT_SENSOR_5_ID=43245255
    IOT_SENSOR_5_NAME=iot-wangs-15
    IOT_SENSOR_5_PLACE_ID=32347983
    IOT_SENSOR_5_USERNAME=iot-wangs-15
    IOT_SENSOR_5_PASSWORD=12345

    IOT_SENSOR_6_ID=43245256
    IOT_SENSOR_6_NAME=iot-wangs-16
    IOT_SENSOR_6_PLACE_ID=32347983
    IOT_SENSOR_6_USERNAME=iot-wangs-16
    IOT_SENSOR_6_PASSWORD=12345

    IOT_SENSOR_7_ID=43245257
    IOT_SENSOR_7_NAME=iot-wangs-17
    IOT_SENSOR_7_PLACE_ID=32347983
    IOT_SENSOR_7_USERNAME=iot-wangs-17
    IOT_SENSOR_7_PASSWORD=12345

    IOT_SENSOR_8_ID=43245258
    IOT_SENSOR_8_NAME=iot-wangs-18
    IOT_SENSOR_8_PLACE_ID=32347983
    IOT_SENSOR_8_USERNAME=iot-wangs-18
    IOT_SENSOR_8_PASSWORD=12345

    IOT_SENSOR_9_ID=43245259
    IOT_SENSOR_9_NAME=iot-wangs-19
    IOT_SENSOR_9_PLACE_ID=32347983
    IOT_SENSOR_9_USERNAME=iot-wangs-19
    IOT_SENSOR_9_PASSWORD=12345

    IOT_SENSOR_10_ID=43245250
    IOT_SENSOR_10_NAME=iot-wangs-20
    IOT_SENSOR_10_PLACE_ID=32347983
    IOT_SENSOR_10_USERNAME=iot-wangs-20
    IOT_SENSOR_10_PASSWORD=12345
```