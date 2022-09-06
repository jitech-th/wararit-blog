## Advance Openvpn configuration with BIRD routing
ในส่วนที่ผ่านมา เราทำ Openvpn แบบ NAT ที่มีวง private IP สำหรับ vpn client โดยหากต้องการออกไปยัง public ต้องมีการทำ NAT แปลง IP จาก private IP เป็น public IP ของ server
แต่ยังมีวิธีอื่นที่ไม่จำเป็นต้องทำ NAT (นอกจากการใช้ public IP เป็นวง vpn client ไปเลย) นั่นคือการใช้ BIRD routing เพื่อประกาศ routing table ให้ router อื่นมองเห็นว่า private IP วงนี้
สามารถเข้าถึงได้ผ่าน tun interface ของเครื่อง vpn server

### What is BIRD routing?
BIRD เป็นโปรแกรมที่ทำให้ linux server ทำตัวเสมือนเป็น router นั่นคือสามารถทำให้ linux server สามารถรัน ospf, bgp, rip protocol และอื่นๆที่ router ใช้ทำ route ได้
