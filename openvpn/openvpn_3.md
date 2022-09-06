## Configure OpenVPN server

### Copy Certificate Files ( ไม่จำเป็น )
เริ่มจากการ copy certificates ที่สร้างไว้แล้วไว้ใน path ที่ต้องการ
- **Copy ca.crt, server.crt และ server.key**
```
cp pki/ca.crt /etc/openvpn/certs/
cp pki/issued/server.crt /etc/openvpn/certs/
cp pki/private/server.key /etc/openvpn/certs/
```
- **Copy client.crt และ client.key**
```
cp pki/issued/client.crt /etc/openvpn/certs/
cp pki/private/client.key /etc/openvpn/certs/
```
- **Copy dh.pem**
```
cp pki/dh.pem /etc/openvpn/certs/
```

**สมมุติต้องการ environment แบบรูปข้างล่าง**
<img width="886" alt="image" src="https://user-images.githubusercontent.com/31476202/188553532-abe0d849-9b2a-470c-a576-d1fb1d3c5c80.png">

### Openvpn server configuration
```
port 1194
proto udp
dev tun
ca   /etc/openvpn/certs/ca.crt
cert /etc/openvpn/certs/server.crt
key /etc/openvpn/certs/server.key
dh /etc/openvpn/certs/dh.pem

mode server
tls-server
topology subnet
push "topology subnet"
ifconfig 10.6.38.1 255.255.255.192
ifconfig-pool  10.6.38.2 10.6.38.62 255.255.255.192
push "route-gateway 10.6.38.1"
push "redirect-gateway"
push "route 100.100.0.0 255.255.0.0"
push "dhcp-option DNS 100.100.0.1"

duplicate-cn
keepalive 10 120
cipher AES-256-CBC
auth SHA256
comp-lzo
max-clients 127
#script-security 3
#username-as-common-name
#auth-user-pass-verify /usr/local/bin/checkpass via-env
#learn-address /etc/openvpn/login
user nobody
group nobody
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
verb 3
daemon
```

`port <number>`  - จะให้เปิด openvpn server นี้ที่ port ไหน

`proto <tcp/udp>` - จะใช้ protocol อะไรในการเปิด openvpn tcp หรือ udp

`dev <tun/tap>`   - virtual interface ที่ต้องการเปิด

`ca <ca.crt path>`      - path ของ ca.crt ที่ให้ openvpn ใช้

`cert <server.crt path>`- path ของ server.crt ที่ให้ openvpn ใช้

`key <server.key path>` - path ของ server.key ที่ให้ openvpn ใช้

`dh <dh.pem path>`      - path ของ dh.pem ที่ให้ openvpn ใช้

`push`                  - จะส่งอะไรให้ client บ้าง
  - `push route <network> <subnet>` - push route ให้ client เอาไว้บอกว่า สามาถเข้า network วงนี้่ผ่าน vpn ได้
  - `push dhcp-option DNS <ip>`     - ในกรณีที่มี local dns ของตัวเอง สามารถบอก client ให้สามารถเข้ามาที่ local dns นี้ได้
  - `push route-gateway <ip>`       - บอก client ว่า gateway คือ ip นี้ เป็น gateway ปกติจะเป็น ip ของขาฝั่ง openvpn server

`ifconfig`              - config ip ของ interface tun/tap

`ifconfig-pool`         - วง ip ของ openvpn client

`user <username>`       - รัน openvpn ในชื่อที่กำหนด

`group <groupname>`     - รัน openvpn ใน group ที่กำหนด

`duplicate-cn`          - ให้ client หลายๆ client สามารถใช้ certificate เดียวกันได้

`max-clients <numeber>` - กำหนดจำนวน client สูงสุดที่สามารถใช้ vpn นี้ได้

`auth-user-pass-verify <authen script path> via-env`  - ใช้ในกรณีที่ต้องการให้มีการทำ authentication โดยเขียน script เพื่อเช็ค username/password แล้ว return ค่าออกมาจาก script นั้น

`daemon`                - รันเป็น daemon mode

### Enable Port-Forwarding and Configure Routing

- **Port forwarding**
```
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
sysctl -p
```

- **Iptables routing**

```
iptables -t nat -A POSTROUTING -s 10.6.38.0/26 -j MASQUERADE
```

or

```
iptables -t nat -A POSTROUTING -s 10.6.38.0/26 --to-source 100.100.1.1 -j SNAT
```

[ความแตกต่างระหว่าง Masquerade และ SNAT](https://www.terrywang.net/2016/02/02/new-iptables-gotchas.html)


- ในกรณีที่ policy ของ FORWARD chain เป็น DROP จำเป็นต้องเพิ่ม rule เพื่อ allow vpn forwarding
```
iptables -A FORWARD -s 10.6.38.0/26 -j ACCEPT
```
