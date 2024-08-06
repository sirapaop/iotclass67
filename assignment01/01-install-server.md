# Install Server and Docker


## How to install Server
1. ลง ubuntu server
2. ลง Ubuntu Server บน Dell gateway
3. สร้าง user และ password สำหรับใช้งานบน Ubuntu Server
4. เชื่อม internet ระหว่าง Dell gateway กับ bridge network บนคอมพิวเตอร์ของเราผ่านสาย LAN
5. ลอง ping network ตรวจสอบว่า dell gateway สามารถต่อ internet ได้
6. เมิ่อต่อ internet สำเร็จ ให้พิมพ์คำสั่ง sudo apt-get update เพื่ออัปเดตแพ็กเกจ


## How to install Docker

1. install docker ตามลิงค์ https://docs.docker.com/engine/install/ubuntu/
2. ตรวจสอบ docker ใช้คำสั่ง docker --version
3. install git โดยใช้คำสั่ง sudo apt install git
4. ตรวจสอบ git ใช้คำสั่ง git --version
5. สร้าง folder เพื่อรองรับ git clone
6. ทำการ clone project ลงใน folder ที่สร้างใช้คำสั่ง git clone https://github.com/sergio11/iot_event_streaming_architecture.git
7. ตรวจสอบว่ามีไฟล์ที่ลงใน folder โดยใช้คำสั่ง ls


