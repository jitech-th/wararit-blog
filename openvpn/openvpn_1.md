## Intro to openvpn

VPN (virtual private network) เป็นระบบ network ที่ทำให้เสมือนกัยว่าเราใช้ internet จากภายในองค์กร โดยปกติเวลาที่เราเข้าเว็บไซต์อะไรซักอย่าง
เราก็จะคุยกับ web server นั้นโดยตรง มี source IP เป็นคอมพิวเตอร์ของเรา destination IP เป็นเครื่อง web server
โดยมี ISP (internet service provider) คอยให้บริการส่งข้อมูล

แต่เมื่อเราใช้ VPN ข้อมูลที่เราส่งจะถูก encrypt (ทำให้มี privacy เพิ่มขึ้น) โดยมี source IP เป็นคอมพิวเตอร์ของเรา แต่ destination IP เป็นเครื่อง VPN server 
หลังจากนั้น VPN server จึงส่งข้อมูลของเราต่อ เสมือนกับว่าเราอยู่ภายในเครือข่ายขององค์กร เดี๋ยวผมจะอธิบายละเอียดอีกที ตอนนี้อยากให้เห็นการทำงานคร่าวๆก่อน

ทั้งหมดที่อธิบายมาข้างบนก็จะได้หน้าตาประมาณนี้

![image](https://user-images.githubusercontent.com/31476202/181870501-2d97444e-fdb3-485b-bcb9-d8f32217693d.png)

### How does a VPN work?

ส่วนนี้ก็จะมาพูดว่า จริงๆแล้วมันทำงานยังไงกันแน่ ทำไมถึงได้ privacy เพิ่มขึ้น ก่อนอื่นอยากให้ดูรูปนี้ก่อนเลย

![image](https://user-images.githubusercontent.com/31476202/181870794-53351fdb-41f5-4698-923e-1c44373e5152.png)

สมมุติให้ iperf ฝั่ง server มี IP เป็น 10.0.1.2 และ iperf ฝั่งเรามี IP เป็น 10.0.1.1 ลองคิดดูเล่นๆว่าในเมื่อ IP เป็น private IP
แล้วเราจะส่ง packet ออกไปยังไง? VPN ก็สามารถเอามาใช้ได้โดยตรงเลย

อธิบายจากรูป สมมุติว่าเรามีโปรแกรมที่ชื่อ iperf ซึ่งโปรแกรมนี้ืเป็นโปรแกรมที่ไปคุยกับ iperf อีกตัวที่อยู่ฝั่ง server
iperf ของเราก็จะสร้าง data เพื่อไปคุยกับ server แล้ว data ของเราก็จะถูกแปะ header ในชั้น TCP(L4) และ IP(L3)
IP ที่ถูกแปะตรงนี้จะเป็น IP ของวง VPN 10.0.1.0/30 (ในกรณีนี้คือ IP ของ iperf ฝ่ังเราและฝั่ง server)

หลังจากนั้น kernel ก็จะไปดูใน routing table ว่า packet ที่จะไปวง VPN 10.0.1.0/30 ต้องไปออก interface ไหน 
ถ้าตามรูปก็จะได้ว่าต้องออก tun0 (มีทั้ง tun และ tap มีความแตกต่างกันนิดหน่อย เดี๋ยวจะอธิบายอีกที) ซึ่งเป็น virtual interface ที่ถูกเปิดด้วยโปรแกรม OpenVPN
เมื่อ packet เข้า tun0 แล้ว packet ก็จะไปที่โปรแกรม OpenVPN ที่มีหน้าที่ encrypt แล้วส่ง packet นั้นไปแปะ header ที่เป็น IP public จริงๆของเรา โดยมีปลายทางเป็น VPN server
แล้วค่อยส่งออก interface จริงๆของเครื่องเรา

พอมาฝั่งรับ ก็ทำกลับกัน kernel แกะ header แล้ว OpenVPN decrypt packet หลังจากนั้น route table ส่ง packet ให้ tun0 ที่มี iperf รันอยู่ 

### TUN vs. TAP

#### TAP mode 

OpenVPN TAP mode จะใช้ TAP interface ที่ทำงานในระดับ layer 2 ทำงานคล้าย switch นั่นหมายความว่า packet ที่ออกจาก TAP interface จะมี header ของ layer 2 อยู่ด้วย
เหมาะกับในกรณีที่เราต้องการให้วง LAN กับ VPN clients อยู่ broadcast domain เดียวกัน หรือต้องการทำงานบย non-IP based traffic แต่ข้อเสียคือ จะเสีย performance ไปกับ header ที่เพิ่มขึ่้น **และไม่สามารถใช้กับ IOS และ android ได้** 

#### TUN mode 

OpenVPN TUN mode จะใช้ TUN interface ที่ทำงานในระดับ layer 3 ทำงานคล้ายกับ router นั่นหมายความว่า packet ที่ออกจาก TUN interface จะมี header ถึงแค่ layer 3 จึงทำให้มี overhead น้อยกว่า แต่จะใช้ได้เฉพาะกับ IP based traffic เท่านั้น และ broadcast traffic จะไม่ถูกส่งไปด้วย
