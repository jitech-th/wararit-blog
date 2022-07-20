## สรุปการใช้งาน Docker#2
ผมขอข้ามวิธีการ install docker ไปเลยนะครับ 

ไปดูกันได้ที่ https://docs.docker.com/engine/install/

พอติดตั้งเสร็จปุ๊ป ลองใช้คำสั่งดู version กันได้เลย
```
$ docker version
```

ดู container ที่มีได้จาก `docker ps` โดยที่ `-a` เป็น option สำหรับแสดงตัวที่ไม่ได้รันอยู่ด้วย
```
$ docker ps -a
```

ก็จะได้ประมาณนี้ เห็นได้ว่ามันจะบอกด้วยว่า container นี้สร้างจาก image ตัวไหน

![alt text](http://)

ดู image ที่มีอยู่ได้จาก
```
$ docker images
```

ซึ่งเราสามารถสั่งให้มันสร้าง container จาก image แล้วก็รัน command ได้ด้วยคำสั่ง
```
$ docker run -d --name abc ubuntu [COMMAND]
```

โดยที่คำสั่งนี้มันจะสร้าง container ชื่อ `abc` (หรือถ้าเราไม่ใส่ name มันก็จะตั้งชื่อให้) จาก image ubuntu โดยไม่จำเป็นต้องมี image นี้บนเครื่อง ถ้าไม่มีมันก็จะไปดึงมาจาก registry (คล้ายๆ github)

ถ้าใช้คำสั่ง `docker run` แล้ว container มันมี status เป็น exited ไม่ต้องแปลกใจ เพราะว่า docker มันจะทำงานเมื่อมันมี service ให้มันทำงาน

เช่นเราอาจจะสั่งให้มัน sleep 10 วินาที container มันก็จะ up อยู่ 10 วินาที แล้วถึงจะ exited ซึ่งถ้ามันเป็น service อื่นที่ทำงานตลอดเวลา มันก็จะทำงานอยู่แบบนั้นแหละ จนกว่าเราจะ stop มัน ดังนั้นมันก็เลยต้องมี `-d` สำหรับ detach ออกมาจาก container (รันเป็น backgound)

นอกจากนี้ container ที่มันกำลังรันอยู่ เราสามารถให้มันทำคำสั่งอื่นได้ โดยใช้คำสั่ง
```
$ docker exec -it abc bash
```

คำสั่งนี้มันก็จะให้ container abc (หรือใช้ id ก็ได้) รันคำสั่ง bash มองง่ายๆก็เหมือนก็เข้าไปข้างใน container แหละ โดยส่วนมาก ถ้ารัน bash ต้องมี `-i` (interactive) `-t` (TTY) ด้วย

เราสามารถสั่งให้ container หยุดทำงานด้วยคำสั่ง `stop` โดยที่ container id ดูได้จาก `docker ps` เกือบลืมบอกว่าเวลาที่เราพิมพ์ id เราสามารถใช้แค่ 2–3 ตัวแรกก็ได้ ถ้า 2–3 ตัวนั้นมันไม่ซ้ำกับตัวอื่น
```
$ docker stop [CONTAINER_ID/Name]
```

หรือสั่งให้รันอีกครั้งด้วยคำสั่ง start อย่าลืมว่าถ้าจะดู id ต้องใช้ docker ps -a
```
$ docker start [CONTAINER_ID/Name]
```

หรือจะลบ container นั้นทิ้งไปเลย ด้วยคำสั่ง
```
$ docker rm [CONTAINER_ID/Name]
```

หรือจะลบ image ทิ้งไปเลยก็ได้ แต่ต้อง stop container ที่ใช้ image นั้นอยู่ก่อนนะ
```
$ docker rmi [IMAGE_ID]
```

เท่านี้ก็น่าจะพอสำหรับคำสั่งพื้นฐาน ไว้เดี๋ยว part ต่อๆ ไปจะมาพูดถึงเรื่อง Dockerfile, docker compose, environment แล้วก็พวก network ต่างๆใน docker กันนะครับ
