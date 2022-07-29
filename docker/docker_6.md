## สรุปการใช้งาน Docker แบบ basic#5

เราพูดถึงเรื่อง docker-compose ไปแล้ว โดย docker-compose ช่วยใฟ้เรา deploy docker หลายๆ container ได้ง่ายขึ้น 
หลายๆ container นั้นบาง container ก็ต้องมีการ communicate กัน หรือบาง container ต้องคุยกับ client ภายนอกโดยตรง
ใน part นี้ก็จะพูดถึงเรื่อง docker networking เป็นหลักว่ามัน communicate กันเองยังไง หรือคุยกับ client ข้างนอกยังไง

### Network drivers

จริงๆแล้ว driver มันมีหลายตัวมาก แต่จะขอพูดถึงแค่ 3 ตัวที่อาจจะได้เห็นบ่อยๆ คือ

- `bridge` คือค่า default ของ docker ถ้าเราไม่ได้ระบุ network driver โดยที่แต่ละ container จะสามารถ communicate กันได้ตามปกติ (ผ่าน ip วง 172 โดย default) การทำแบบนี้ทำให้ network ของ docker มีความ isolate
- `host` คือการ map ip ของเครื่อง host ที่ port ใด port หนึ่งเข้ากับ container โดยตรง ไม่จำเป็นต้องทำ NAT
- `none` container นั้นจะไม่สามารถ communicate กับใครได้เลย มีความ isolate สูงสุด (เพราะคุยกับใครไม่ได้)

ดูรูปข้างล่างเผื่อจะเข้าใจได้ง่ายขึ้น

![alt text](https://user-images.githubusercontent.com/31476202/181682799-ceaacde6-43e8-427a-b320-3854549035c7.png)

### Using bridge networks 

bridge network เป็นค่า default ของ docker ที่จะ attach container เข้ากับ bridge ที่ชื่อ `bridge` (default bridge) 
แต่ในกรณีที่เราต้องการสร้าง bridge เองก็สามารถทำได้ (user-defined bridge)

#### ความแตกต่างระหว่าง user-defined และ default bridge

- user-defined bridge จะมี dns resolution ระหว่าง container ให้ ขณะที่ default bridge จะใช้ได้เฉพาะ ip address เท่านั้น หรือสามารถใช้ `/etc/hosts` แทน dns ได้เช่นกัน (แต่ทำให้ debug ยากขึ้น)
- user-defined bridge ทำให้มีความ isolation มากกว่า เพราะโดย default container จะ attach เข้ากับ default bridge
- container สามารถ attach เข้ากับ user-defined bridge ตอนไหนก็ได้ ต่างกับ default bridge ที่จำเป็นต้อง stop container ก่อน จึงสามารถ recreate แล้ว attach เข้ากับ bridge อื่นได้
- การ share environment varaibles บน default bridge สามารถทำได้ด้วย `--link` แต่ถ้าต้องการ share variables บน user-defined bridge สามารถทำได้โดย
  - mount บน volume เดียวกัน
  - ใช้ `docker-compose` กำหนด shared variables
  - ใช้ `docker swarm` กำหนดค่า config และ secret

#### Bridge commands

- create a bridge
```
$ docker network create my-net
```
- delete a bridge
```
$ docker network rm my-net
```
- connect a container to a bridge
```
$ docker create --name my-nginx \
  --network my-net \
  --publish 8080:80 \
  nginx:latest
```
for a running container
```
$ docker network connect my-net my-nginx
```
- disconnect a bridge
```
$ docker network disconnect my-net my-nginx
```
