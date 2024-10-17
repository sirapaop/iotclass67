# Install Server and Docker


## How to install Server
### วิธีการลง Ubuntu server ในเครื่อง Dell IOT gateway
- ลง Ubuntu Server บน Dell gateway ผ่าน flash drive
- เลือกลงเป็นแบบภาษาอังกฤษทั้ง lauguage และ keyborad configure
- ทำการ set ค่า IP ของเครื่อง Dell gateway ตามที่เรากำหนดไว้กับสมาชิกอื่นๆ
```
172.16.46.66 # กลุ่มที่ 6
```
- ทำการกำหนด disk เป็นแบบ custom แล้ว add gpt partition เป็น 1G 
- สร้าง user และ password สำหรับใช้งาน Ubuntu Server
- หลังจากนั้นเชื่อม internet ระหว่าง Dell gateway กับ network ผ่านสาย LAN
- ลอง ping network ตรวจสอบว่า dell gateway สามารถต่อ internet ได้
```
ping 8.8.8.8
```
- เมิ่อต่อ internet สำเร็จ ให้พิมพ์คำสั่งทำการอัปเดตแพ็กเกจ

```
sudo apt-get update
```
## How to install Docker
- ในการ install docker เราจะใช้ลิงค์ https://docs.docker.com/engine/install/ubuntu/ เป็นตัวอย่างในการ install

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

- การลง docker version ที่ latest
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- ตรวจสอบ docker โดยใช้คำสั่งและลอง run
```
sudo docker --version
sudo docker run hello-world
```

### Install Git
- เราจะทำการ install git และ ตรวจสอบ git version โดยใช้คำสั่ง 
```
sudo apt install git
git --version
```

หลังจากนั้นเราจะทำการ clone project iot_event_streaming มาที่เครื่องของเรา
```
git clone https://github.com/sergio11/iot_event_streaming_architecture.git
```