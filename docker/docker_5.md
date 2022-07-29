## สรุปการใช้งาน Docker แบบ basic#4

หลังจากที่พูดเรื่อง Dockerfile และ docker-compose ไปบ้างแล้ว ที่สำคัญคือ ด้วยความที่ docker ออกแบบมาให้เรา create หรือ destroy มันได้อย่างง่ายๆ
แล้วจะเกิดอะไรขึ้นถ้าเราเก็บ data สำคัญไว้ข้างใน? data เราจะหายไปด้วยหรือเปล่านะ เราต้อง copy data ของเราทุกครั้งก่อน destory หรือเปล่า 

เพราะงั้น part นี้จะพูดถึงเรื่อง volume ของ Docker ว่ามันสร้าง volume หรือว่าเรียกใช้ volume แล้วเราจะสามารถ mount หรือ bind volume ที่เรามีอยู่ยังไงได้บ้าง

### The copy-on-write (CoW) strategy

ลองคิดดูว่าบนเครื่องเรามี docker image เยอะแยะไปหมด หลายๆ image เรา build ขึ้นจาก base image เดียวกัน นั่นหมายความว่าเราต้องใช้ storage สำหรับแต่ละ image
ดังนั้น CoW จะเข้ามาช่วยแก้ปัญหานี้ คือแทนที่ docker จะใช้ storage เท่ากับจำนวน image * พื้นที่ของแต่ละ image แต่ docker มันจะเก็บเฉพาะ change ที่ต่างไปจาก base image
สั้นๆคือ เก็บ base image แค่อันเดียว แล้ว image ที่เราสร้างจาก base image นั้นก็จะมาอ่านที่ base image นั้น ส่วนอะไรที่ต่างกันในแต่ละ image จึงเก็บแยก ทำให้ประหยัด storage ไปได้เยอะ
ซึ่ง docker มันก็จะเก็บเป็น layer ซ้อนไปเรื่อยๆ นั่นแหละ

โดยปกติแล้ว data หรือไฟล์ต่างๆที่สร้างขึ้นใน container จะถูกเก็บไว้ใน writable container layer (layer หลังจากที่รัน image นั้นขึ้นมาแล้ว) ตามรูปข้างล่าง

![alt text](https://docs.docker.com/storage/storagedriver/images/container-layers.jpg)

ส่วน read-only layer จะเป็นส่วนของ image layer ที่เราไม่สามารถเขียน data อะไรลงไปได้

ดังนั้นถ้าเราสร้าง container และเขียน data ลงไปที่ writable layer data จะยังอยู่บน container จนกว่าเราจะ destroy มัน
เมื่อเราจะ run container ใหม่ data ที่เราสร้างไว้ก็จะหายไป เพราะ container ถูก run ขึ้นมาจาก image ที่เราไม่ได้เขียน data อะไรลงไป (อย่าลืมว่าเราเขียนที่บน container layer)

แล้วเราจะทำยังไงถึงจะทำให้ data มันอยู่ตลอดล่ะ? คำตอบมี 2 วิธีคือ 

1. การ mount หรือ bind volume
2. การเอา data ไปใส่ตอน build ด้วยเลย

แต่ที่จะมาพูดถึงในตอนนี้คือวิธีแรก

### Type of mount

การ mount มีด้วยกัน 3 วิธี คือ 
1. **volume** ถูกเก็บไว้ใน filesystem ที่ถูกจัดการด้วย docker (`/var/lib/docker/volumes/`) ซึ่งเป็นวิธีที่ดีที่สุดในการจะทำ persist data 
2. **bind** mount เก็บไว้ที่ในก็ได้ในระบบ เหมือนเป็นการชี้ path ให้ docker ไปอ่านใน directory ที่เรากำหนดไว้
3. **tmpfs** เก็บทุกอย่างไว้ใน memory
ดูความต่างได้จากรูปข้างล่าง 

![alt text](https://docs.docker.com/storage/images/types-of-mounts.png)

### Docker volume commands

สร้าง volume
```
$ docker volume create my-vol
```

list volume
```
$ docker volume ls
```

ลบ volume 
```
$ docker volume rm my-vol
```

### Start a container with volume or bind mount

สามารถทำได้ด้วย `-v` หรือ `--mount` ต่างกันเล็กน้อยตรงที่ `--mount` จะมีความ verbose กว่า

ถ้าใช้ volume ก็ใส่ชื่อ volume ได้เลย หรือถ้า bind mount ก็ใส่เป็น absolute path เช่น `-v <volume_name>:/app` หรือ `-v <path>:/app`

ตัวอย่าง `--mount`
```
$  docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  nginx:latest
```
ตัวอย่าง `-v`
```
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app \ 
  nginx:latest
```
