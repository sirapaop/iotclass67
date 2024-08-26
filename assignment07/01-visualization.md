# Data Visualization.
เป็นการสร้าง Dashboard เพื่อดูค่าที่รับมาจาก server โดยใช้ **_"Grafana"_** ร่วมกับ plugin ที่ชื่อว่า **_"FlowCharting"_**
ซึ่งช่วยในการออกแบบและสร้างแผนผังสถานที่เพื่อให้ Dashboard ดูสวยงามและดูง่ายยิ่งขึ้น ในการสร้างผังสถานที่เราจะสร้างผ่าน **_"Draw.io"_**

### Install flowcharting

### **_1. Mount Volumn ใน Grafana เพื่อเก็บ plugins_**

สิ่งแรกที่ต้องทำ คือ การ mount volumn ของ container ให้ถูกต้อง เพื่อป้องกันไม่ให้ plugin ที่เรากำลังจะลงหายไปเวลาเราสั่ง down container 
- เปิดการรองรับ Angular UI แต่ไม่แสดงการเตือนเกี่ยวกับการเลิกใช้งาน Angular UI ใน Grafana โดยเพิ่ม 
    - GF_SECURITY_ANGULAR_SUPPORT_ENABLED=True
    - GF_FEATURE_TOGGLES_ANGULARDEPRECATIONUI=False

```yml
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
```
เมื่อ mount volumn เสร็จแล้วให้ save และ run grafana ใหม่

 ```bash
 docker compose up grafana
```

### **_2. ติดตั้ง FlowCharting_**

#### 1. cd เข้าไปใน plugin directory

```bash
cd /var/lib/grafana/plugins
```
#### 2. สร้าง directory มา 1 directory ด้วยชื่ออะไรก็ได้ เพื่อเก็บ plugin file ที่เราจะทำการ get และ cd เข้าไป

```bash
mkdir src
cd src
```

#### 3. โหลด plugin โดยการใช้ wget ตามด้วยลิงค์ zip file
- plugin มาจาก https://github.com/skyfrank/grafana-flowcharting/releases/tag/v1.0.0e
- click ขวา copy link address ของ agenty-flowcharting-panel-1.0.0e.231214594-SNAPSHOT.zip

```bash
wget https://github.com/skyfrank/grafana-flowcharting/releases/download/v1.0.0e/agenty-flowcharting-panel-1.0.0e.231214594-SNAPSHOT.zip
```

- สำหรับใครที่ใช้ wget ไม่ได้ให้ 

```bash
sudo apt-get update
sudo apt-get install wget
```

#### 4. extract zip file ด้วยการใช้ unzip

```bash
unzip agenty-flowcharting-panel-1.0.0e.231214594-SNAPSHOT.zip
```

#### 5. move dist folder to grafana-flowcharting-panel

```bash
mv dist ../grafana-flowcharting-panel
```
</br>
#### แก้ไขให้ grafana รองรับ angular plugin ได้เนื่องจาก grafana เวอร์ชั่นใหม่ ๆ ไม่รองรับแล้ว แต่เราแก้ไขได้

- เข้าไปใน container ของ grafana

```bash
docker exec -it grafana /bin/bash
```
- ไปที่ path /etc/grafana

 ```bash
 cd /etc/grafana

 ```

 - find angular (/angular)
 - change ;angular_support_enabled = false    to    angular_support_enabled = true
 - restart docker

```bash
docker compose restart grafana
```

**จากนั้นก็เข้าไปใน grafana ผ่าน browser เพื่อตรวจสอบว่ามี plugin ชื่อ FlowCharting ติดตั้งแล้วหรือยัง ถ้ามีแสดงว่าติดตั้งสำเร็จแล้ว**

</br>