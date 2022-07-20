## สรุปการใช้งาน Docker แบบ basic#3
ใน part ที่แล้วเขียนถึงการสร้าง image จาก Dockerfile ซึ่ง concept ของ docker คือ microservices อย่างที่เขียนไว้ก่อนหน้านี้ 
ว่าส่วนมาก 1 container ก็จะมี 1 service อะไรงี้ ดังนั้นเนี่ย ถ้าเรามีหลาย service ที่ต้องใช้ มันจะมีวิธีไหมนะ ที่เราจะสามารถสร้าง แล้วก็จัดการหลายๆ service ได้ง่ายๆ?

### มันจึงเป็นที่มาของ docker-compose ยังไงล่ะ
**docker-compose** เป็น tool ตัวนึงที่เราสามารถเขียน script เพื่อให้มันสร้าง container จาก image หลายๆตัวได้ แล้วก็อาจจะจัดการ container พวกนั้นได้ด้วยคำสั่งเดียว เช่นอยากให้หยุดการทำงาน ก็สั่งแค่คำสั่งเดียว อะไรประมาณนี้ แทนที่เราจะต้องมาสั่งให้ container นี้ container นั้นหยุดการทำงานทีละคำสั่ง เป็นต้น
เมื่อระบบโดยรวมหน้าตาประมาณนี้ 
```
(External user) --> 443 [frontend network]
                            |
                  +--------------------+
                  |  frontend service  |...ro...<HTTP configuration>
                  |      "webapp"      |...ro...<server certificate> #secured
                  +--------------------+
                            |
                        [backend network]
                            |
                  +--------------------+
                  |  backend service   |  r+w   ___________________
                  |     "redis"        |=======( persistent volume )
                  +--------------------+        \_________________/
```

ลักษณะหน้าตาของ `docker-compose.yml` มันก็จะเป็นประมาณนี้
```
version: "3.9"
services:
  webapp:
    build: .
    ports:
      - "443:8043"
    volumes:
      - .:/code
    configs:
      - httpd-config
    secrets:
      - server-certificate
    environment:
      FLASK_ENV: development
  redis:
    image: "redis:alpine"
    volumes:
      - db-data:/etc/data
    
volumes:
  db-data:
    driver: flocker
    driver_opts:
      size: "10GiB"
      
 configs:
  httpd-config:
    external: true

secrets:
  server-certificate:
    external: true
```

จะเห็นได้ว่าถ้าใช้คำสั่ง `docker run` เป็นน่าจะเข้าใจความหมายในแต่ละบรรทัดของ `docker-compose.yml` ได้ไม่น่ายาก

แต่จะอธิบายคร่าวๆให้เข้าใจก็คือ `docker-compose` ที่รันไฟล์นี้มันจะสร้าง service มา 2 อย่าง ชื่อ `webapp` และ `redis` (เหมือนกับที่ใส่ใน `--name` ตอน `docker run`)

- service `webapp` จะใช้ image ที่ build เอง (ใน directory ต้องมี dockerfile อยู่ด้วย) แต่ของ `redis` จะใช้ image ที่ชื่อ `redis:alpine`
- service `webapp` มีการ map port 443 ของ host เข้ากับ 8043 ของ container (เหมือนกับ `-p` ใน `docker run` ) 
- volumes จะ map `.` กับ `/code` ของ container (เหมือนกับ `-v` ใน `docker run`) 
- environment ก็คือการกำหนด environment variable (เหมือนกับ `-e` ใน `docker run`)

สุดท้ายแล้ว พอเราเขียนไฟล์ `docker-compose.yml` เสร็จ ก็สามารถสั่งให้มันทำงานได้ด้วยคำสั่ง
```
docker-compose up -d
```
แล้วมันก็จะได้ container มา 2 อัน คือ `webapp` กับ `redis`

ลองเช็คว่ามี container รึยังด้วย
```
docker ps
```

หรือถ้าจะให้มันหยุดทำงานทั้ง 2 container ที่เราสร้างก็ใช้คำสั่ง
```
docker-compose stop
```

หรือจะเอาแบบถอนรากถอนโคน ลบ container ทิ้งไปด้วย ก็คำสั่งนี้
```
docker-compose down --volumes 
```
**--volumes นี่คือลบทิ้งทั้ง volumes ที่เราไป map ไว้ด้วยนะ ระวังกันดีๆ**

