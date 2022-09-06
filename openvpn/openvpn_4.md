## Configure Openvpn client

- **Client confguration**
```
client
dev tun
proto udp
remote 100.100.1.1 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ns-cert-type server
comp-lzo
verb 3
#auth-user-pass
<ca>
-----BEGIN CERTIFICATE-----
MIIDnDCCAwWg....
-----END CERTIFICATE-----
</ca>
<cert>
-----BEGIN CERTIFICATE-----
MIID1zCCA0CgA....
-----END CERTIFICATE-----
</cert>
<key>
-----BEGIN PRIVATE KEY-----
MIICdwIBADANB....
-----END PRIVATE KEY-----
</key>
```
จะเห็นได้ว่าหลายๆ อย่างต้องปรับให้ตรงกับของ server

``remote <ip> <port>``  - ip และ port ของเครื่อง server openvpn

``authen-user-pass``    - ให้มีหน้าต่างสำหรับใส่ username/password กรณีที่ฝั่ง server ต้องการการทำ authentication

- จะมี key และ certificate ที่ต้อง copy มาใส่ใน client configuration คือ
  - ``ca.crt`` ใส่ใน ``<ca>``
  - ``client.crt`` ใส่ใน ``<cert>``
  - ``client.key`` ใส่ใน ``<key>``
 
