## สรุปการใช้งาน Docker#3

ต่อจาก part แรกนะครับ ใน part ที่แล้วผมก็ได้อธิบายเรื่องของ microservices รวมถึง concept คร่าวๆแล้วก็คำสั่งพื้นฐาน อย่างเช่น docker ps, run, start, stop ของตัว docker ไปแล้ว 
ใน part นี้ก็จะเน้นไปที่เรื่องของ image เป็นหลัก แล้วก็พวก Dockerfile ที่เอาไว้สร้าง image ด้วย

### Dockerfile คืออะไร?
**Dockerfile** เป็นคล้ายๆกับ script ที่เอาไว้สร้าง docker image กำหนด spec ของ image ว่าจะให้มีลักษณะยังไง มี environment แบบไหน ตัวอย่างคร่าวๆ ด้านล่าง
```
FROM node:12-alpine
RUN apk add --no-cache python g++ make
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```
จากตัวอย่าง ก็จะเห็นว่าในแต่ละบรรทัด จะมีคำสั่งอยู่ข้างหน้า แล้วตามด้วย parameter เดี๋ยวจะอธิบายให้ละเอียดมากขึ้นครับ

Dockerfile instruction หรือคำสั่งที่เห็นบ่อยๆจะมีอยู่ไม่กี่ตัว เช่น
- `FROM` คำสั่งนี้เป็นคำสั่งที่กำหนดว่าจะใช้ base image ตัวไหน ซึ่งทุก Dockerfile ต้องมีคำสั่งนี้ แสดงว่า image ที่เราสร้างใหม่ต้องมาจาก base image อื่นนั่นเองครับ โดยปกติถ้าไม่รู้จะใช้ base image ตัวไหนอาจจะใช้เป็น alpine หรือเป็น ubuntuไรงี้ก็ได้
- `RUN` คำสั่งนี้ใช้สำหรับบอกว่า จะให้ทำอะไรก่อนจะเริ่มรัน container บ้าง อะไรประมาณนั้น สามารถเขียนหลายบรรทัดได้ เพื่อให้อ่านง่าย คั่นด้วย \
```
RUN apt-get update && apt-get install -y \
    package-bar \
    package-baz \
    package-foo=1.3.*
```
- `CMD` คำสั่งนี้ใช้ run ตอนที่ build image เสร็จแล้ว
- `EXPOSE` คำสั่งนี้ใช้สำหรับบอกว่า container ที่เราจะสร้าง จะให้มันเปิด port ไหน
- `ENV` คำสั่งนี้ใช้ประกาศตัวแปร environment variables ที่จะไปอยู่ใน container ที่เราสร้างจาก image ที่มาจาก dockerfileนี้
- `COPY` คำสั่งนี้ใช้ copy ไฟล์จาก directory ข้างนอก (ก่อนสร้าง image) ให้ไปอยู่ใน container (จาก image ที่เราสร้าง) ตาม path ที่เรากำหนด
- `ENTRYPOINT` คำสั่งนี้คล้ายๆ `CMD` เลย แต่จะมีส่วนที่ต่างกันตรงที่ `CMD` เป็นเหมือนคำสั่ง default สามารถโดนเขียนทับได้ตอนที่สั่ง docker run แต่ `ENTRYPOINT` เป็นเหมือนคำสั่งที่รอรับ parameter จากคำสั่ง docker run ทั้ง `ENTRYPOINT` และ `CMD` สามารถใช้ร่วมกันได้ ตัวอย่างเช่น

```
FROM centos:8.1.1911
CMD ["echo", "Hello Docker"]
```

แล้วเราสั่งรัน

```
$ docker run <image-id>
Hello Docker
$ docker run <image-id> hostname # hostname is exec to override CMD
244af....
```

แต่ถ้าเป็น ENTRYPOINT

```
FROM centos:8.1.1911
ENTRYPOINT ["echo", "Hello Docker"] 
```

แล้วสั่งรัน

```
$ docker run <image-id>
Hello Docker
$ docker run <image-id> hostname   # hostname as parameter to exec
Hello Docker hostname
```

ซึ่งทั้ง CMD แล้ว ENTRYPOINT มันสามารถใช้ร่วมกันได้

```
FROM centos:8.1.1911
ENTRYPOINT ["echo", "Hello"]
CMD ["Docker"]
```

แล้วสั่งรัน

```
$ docker run <image-id>
Hello Docker
$ docker run <image-id> Ben
Hello Ben
```

- `VOLUME` คำสั่งนี้เอาไว้สำหรับ map ไฟล์ระหว่างใน container กับนอก container ซึ่งมันมีประโยชน์มากๆ ในกรณีทีมันเป็น database 
สมมุติว่า container เราโดนลบ ปกติถ้าไม่ map ข้อมูลที่อยู่ใน container จะหายหมด แต่ถ้า map ต่อให้ลบกี่ครั้ง ถ้าเราสร้าง container จาก image เดิมหรือ map ไปที่เดิม ข้อมูลเราก็ไม่หาย

### แล้วเราจะ build ยังไงล่ะ?
ใช้คำสั่ง `docker build` โดย `-t` คือการกำหนดชื่อและ tag จะอยู่ในรูป `[name:tag]` เช่น `my_name:1.0` แต่ถ้าไม่ใส่ tag มันจะ default เป็น `lastest`
ส่วน path คือจะให้ build จากที่ไหน (บอกว่า Dockerfile เราอยู่ไหนนั่นแหละ)

```
$ docker build -t <name> [path]
```

พอเรา build เสร็จแล้วลองเช็คได้ด้วย docker images เช่น

```
$ docker images
REPOSITORY    TAG       IMAGE ID        CREATED             SIZE
my_name       lastest   308e519aac60    6 days ago          824.5 MB
```

แล้วทีนี้เราก็สามารถสร้างรัน container จาก images นั้นได้

### แถมนิดนึง เรื่อง registry

docker จะมีสิ่งที่เรียกว่า registry อยู่ ซึ่งมันคล้ายๆ github ที่ไว้เก็บโค้ด เราสามารถ push และ pull image มาใช้ได้
เวลาที่เราสั่ง docker run แล้วตามด้วย image ที่เราไม่ได้มีอยู่ในเครื่อง มันก็จะไป pull มาจาก registry (docker hub) นี่แหละ ทำให้การใช้งาน docker มันง่ายขึ้นเยอะ
