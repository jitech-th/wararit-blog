## Setting up CA, certificates and keys for an OpenVPN server and clients

ก่อนที่เราจะไปดูการ install และ config openvpn server และ client สิ่งที่จำเป็นต้องมีก่อนเลยคือ certificate และ key
ดังนั้นในตอนนี้จะพูดถึงเรื่องการ create และการ sign certificate เป็นหลัก 

อ่านเพิ่มเติมได้ที่ [PKI คืออะไร?](https://www.etda.or.th/th/Useful-Resource/Knowledge-Sharing/articles/Public-Key-Infrastructure.aspx)

### easyrsa 

ในตอนนี้เพื่อไม่ให้การสร้าง CA, certificate และ key ซับซ้อนเกินไป จะใช้ easyrsa เป็น tool ช่วยสร้าง CA, certificate และ key
- ก่อนอื่นอย่าลืม check ว่า ntp enable แล้วหรือยัง ไม่เช่นนั้นจะเกิดปัญหาทีหลัง
```
$ timedatectl show
Timezone=Asia/Bangkok
LocalRTC=no
CanNTP=yes
NTP=yes
NTPSynchronized=yes
TimeUSec=Tue 2022-08-09 16:00:03 +07
RTCTimeUSec=Tue 2022-08-09 16:00:03 +07
```

- ถ้า `NTP=no` ให้ตั้งเป็น yes ด้วย
```
$ timedatectl set-ntp yes
```

- download easyrsa
```
$ wget https://github.com/OpenVPN/easy-rsa/releases/download/v3.1.0/EasyRSA-3.1.0.tgz
$ tar -xf EasyRSA-3.1.0.tgz
$ mv EasyRSA-3.1.0/ easy-rsa/; rm -f EasyRSA-3.1.0.tgz
```

- ตั้งค่า vars ของ easy-rsa (ด้านล่างเป็นแค่ตัวอย่างเท่านั้น)
```
$ cd easy-rsa
$ vim vars
set_var EASYRSA                 "$PWD"
set_var EASYRSA_PKI             "$EASYRSA/pki"
set_var EASYRSA_DN              "cn_only"
set_var EASYRSA_REQ_COUNTRY     "TH"
set_var EASYRSA_REQ_PROVINCE    "Bangkok"
set_var EASYRSA_REQ_CITY        "Bangkok"
set_var EASYRSA_REQ_ORG         "My ORG"
set_var EASYRSA_REQ_EMAIL       "mymail@myorg.org"
set_var EASYRSA_REQ_OU          ""
set_var EASYRSA_KEY_SIZE        2048
set_var EASYRSA_ALGO            rsa
set_var EASYRSA_CA_EXPIRE       7500
set_var EASYRSA_CERT_EXPIRE     365
set_var EASYRSA_NS_SUPPORT      "no"
set_var EASYRSA_NS_COMMENT      ""
set_var EASYRSA_EXT_DIR         "$EASYRSA/x509-types"
set_var EASYRSA_SSL_CONF        "$EASYRSA/openssl-easyrsa.cnf"
set_var EASYRSA_DIGEST          "sha256"
```

- ทำให้ vars สามารถ executable ได้
```
$ chmod +x vars
```

- **init and build CA**
```
$ ./easyrsa init-pki
$ ./easyrsa build-ca
```

- **build server key**
```
$ ./easyrsa gen-req <server_name> nopass
$ ./easyrsa sign-req server <server_name>
```

- verify
```
$ openssl verify -CAfile pki/ca.crt pki/issued/<server_name>.crt
pki/issued/<server_name>.crt: OK
```

- **build client key**
```
$ ./easyrsa gen-req <client_name> nopass
$ ./easyrsa sign-req client <client_name>
```

- **build DH key**
```
$ ./easyrsa gen-dh
```
