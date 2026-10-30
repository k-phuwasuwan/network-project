# Network Project

เว็บไซต์ WordPress บน Azure VM (Ubuntu 24.04) เปิดใช้งานที่ https://compilerbuu.duckdns.org
สร้างด้วย Nginx, PHP-FPM, MySQL และ HTTPS จาก Let's Encrypt พร้อม Dashboard สำหรับตรวจความปลอดภัยของเซิร์ฟเวอร์
โปรเจกต์นี้เป็นส่วนหนึ่งของรายวิชา Network โดย
1. 67160050 กันตวิชญ์ ภูวะสุวรรณ์
2. 67160065 ณัฐพล เสาร์เพ็ง
3. 67160253 จิรวัฒน์ เลิศสันติคุปต์
4. 67160265 พีรพัฒน์ ภู่สำอางค์
5. 67160277 สิรภพ วิระวณิชย์ชัย
6. 67160423 ธนพล วัฒนสุข
7. 67160426 นที บุญญานนท์
8. 67160427 พงศ์พิเชษฐ์ มูลชาลี

## ภาพรวมระบบ
ขั้นตอนการสร้างระบบ เรียงตามที่ผู้พัฒนาทำ:

1. **Azure VM**: เครื่องบนคลาวด์ที่เก็บทุกอย่างไว้ เข้าด้วย SSH Key
2. **Firewall**: เลือกว่าใครเข้าทางพอร์ตไหนได้
3. **Nginx + PHP-FPM + WordPress + MySQL**: รับคำขอ ประมวลผล และเก็บเนื้อหาของเว็บ
4. **DuckDNS**: แปลงเลข IP ให้เป็นชื่อโดเมนที่จำง่าย
5. **HTTPS**: เข้ารหัสข้อมูลระหว่างเบราว์เซอร์กับเซิร์ฟเวอร์
6. **Security**: ตรวจ log แจ้งเตือนผ่าน Discord และสรุปด้วย AI (OpenRouter) แสดงบน Dashboard

## เทคโนโลยีที่ใช้
| ชั้น | เครื่องมือ | หน้าที่ |
| --- | --- | --- |
| เครื่อง | Azure VM, Ubuntu 24.04 | เครื่องเซิร์ฟเวอร์ |
| เครือข่าย | Azure NSG, UFW | กรองการเข้าถึงสองชั้น |
| โดเมน | DuckDNS | ชื่อโดเมนฟรี |
| เว็บเซิร์ฟเวอร์ | Nginx | รับคำขอ ส่งต่อให้ PHP ป้องกัน Dashboard |
| แอปพลิเคชัน | PHP-FPM 8.3, WordPress | สร้างหน้าเว็บ |
| ฐานข้อมูล | MySQL 8.0 (เฉพาะ localhost) | เก็บเนื้อหาและการตั้งค่า |
| ใบรับรอง | Let's Encrypt (Certbot) | HTTPS และต่ออายุอัตโนมัติ |
| เฝ้าระวัง | Python, Discord, OpenRouter | วิเคราะห์ log และแจ้งเตือน |

## ไฟล์ในโปรเจกต์
```text
network-project/
├─ README.md
└─ compilerbuu-wordpress/
   ├─ wordpress.html
   ├─ security-dashboard.html
   ├─ logs.html
   └─ ai.html
```

| ไฟล์ | คืออะไร |
| --- | --- |
| `compilerbuu-wordpress/wordpress.html` | โค้ดของหน้า WordPress (หน้าเว็บหลักที่ผู้ใช้เห็น) |
| `compilerbuu-wordpress/security-dashboard.html` | Dashboard ภาพรวม (ต้องล็อกอิน) |
| `compilerbuu-wordpress/logs.html` | เหตุการณ์ที่ต้องสงสัย กรองได้ตามความเสี่ยง ประเภท และ IP |
| `compilerbuu-wordpress/ai.html` | สรุปผลวิเคราะห์จาก AI และสถานะการอ่าน log |

## เส้นทางของคำขอ
เมื่อผู้ใช้เปิดเว็บ:

1. ผู้ใช้พิมพ์ `https://compilerbuu.duckdns.org` ในเบราว์เซอร์
2. เบราว์เซอร์ถาม DuckDNS ว่าชื่อนี้อยู่ที่ IP ใด
3. DuckDNS ตอบเลข IP ของเซิร์ฟเวอร์
4. เบราว์เซอร์ส่งคำขอไปที่ IP นั้นทางพอร์ต 443
5. Firewall ตรวจแล้วอนุญาตให้ถึง Nginx
6. Nginx ส่งคำขอที่ต้องประมวลผลให้ PHP-FPM (WordPress)
7. WordPress ขอข้อมูลหน้าเว็บจาก MySQL
8. MySQL ส่งข้อมูลกลับให้ WordPress
9. WordPress ส่งหน้าเว็บให้ Nginx
10. Nginx ส่งหน้าเว็บกลับถึงผู้ใช้ผ่าน HTTPS

## Configuration

| ส่วน | ไฟล์ในเครื่อง | หน้าที่ |
| --- | --- | --- |
| SSH | `/etc/ssh/sshd_config` | กำหนดวิธีเข้าเครื่อง |
| Firewall | UFW (`ufw status`) + Azure NSG | เลือกว่าใครเข้าพอร์ตไหนได้ |
| Nginx | `/etc/nginx/sites-enabled/compilerbuu` | รับคำขอ ส่งต่อให้ PHP ป้องกัน Dashboard |
| HTTPS | `/etc/letsencrypt/` (Certbot) | ใบรับรองและการต่ออายุอัตโนมัติ |

### SSH
```conf
PermitRootLogin no
PasswordAuthentication no
```
- ปิดล็อกอินด้วย root และรหัสผ่าน ใช้ SSH Key เท่านั้น
- ตรวจ: `sudo sshd -T | grep -E 'permitrootlogin|passwordauthentication'`

### Firewall
| พอร์ต | ใคร | ใช้ทำอะไร |
| --- | --- | --- |
| 22/tcp | IP ของทีม | SSH |
| 80/tcp | ทุกคน | เปลี่ยนเส้นทางไป HTTPS และให้ Let's Encrypt ตรวจโดเมน |
| 443/tcp | ทุกคน | HTTPS |

- พอร์ตอื่นปฏิเสธทั้งหมด (default deny)
- ตรวจ: `sudo ufw status verbose`

### Nginx
```nginx
location /security-dashboard {
    auth_basic "Restricted Security Dashboard";
    auth_basic_user_file /etc/nginx/.htpasswd;
    try_files /security-dashboard.html =404;
}

location = /logs.html {
    auth_basic "Restricted Security Dashboard";
    auth_basic_user_file /etc/nginx/.htpasswd;
}

location = /ai.html {
    auth_basic "Restricted Security Dashboard";
    auth_basic_user_file /etc/nginx/.htpasswd;
}

location = /api/security_data.json {
    alias /var/www/compilerbuu-wordpress/security_data.json;
    auth_basic "Restricted Security Dashboard";
    auth_basic_user_file /etc/nginx/.htpasswd;
    add_header Cache-Control "no-store" always;
}

add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```
- ทุกหน้าของ Dashboard และไฟล์ข้อมูลต้องล็อกอินด้วย Basic Auth
- เปิด HSTS ให้เบราว์เซอร์ใช้ HTTPS เสมอ
- ตรวจ: `sudo nginx -t` จากนั้น `curl -I https://compilerbuu.duckdns.org/security-dashboard` ควรได้ 401

### HTTPS
- ใบรับรองจาก Let's Encrypt ผ่าน Certbot ต่ออายุอัตโนมัติ
- ตรวจ: `sudo certbot renew --dry-run`

## Security Dashboard
Dashboard มี 3 หน้า เปิดที่ `/security-dashboard` และต้องล็อกอิน

| หน้า | แสดงอะไร |
| --- | --- |
| ภาพรวม | สรุปความเสี่ยง ตัวเลขคำขอ สัดส่วน HTTP status path ที่ถูกเรียกมากที่สุด ประเภทและ IP ที่ก่อเหตุ |
| เหตุการณ์ | รายการเหตุการณ์ที่ต้องสงสัย กรองตามความเสี่ยง ประเภท และค้นหา IP หรือ path |
| AI วิเคราะห์ | ผลสรุปจาก AI และสถานะการอ่าน log |

### ความปลอดภัยของหน้าเว็บ
- ข้อความจาก log แสดงเป็นข้อความธรรมดาเสมอ ไม่ถูกแปลงเป็น HTML จึงกันการแทรกสคริปต์ผ่านบรรทัด log ได้
- ไม่โหลดไฟล์จากเว็บภายนอก จึงไม่ต้องเปิดโดเมนเพิ่มใน CSP