# lerning haproxy

การใช้งาน HAProxy แบบทีละขั้นตอนสำหรับผู้เริ่มต้น:

**1. ทำความเข้าใจ HAProxy เบื้องต้น:**

  * HAProxy ย่อมาจาก High Availability Proxy เป็นซอฟต์แวร์โอเพนซอร์สที่ทำหน้าที่เป็นตัวกระจายโหลด (Load Balancer) และพร็อกซีสำหรับแอปพลิเคชัน TCP และ HTTP
  * หน้าที่หลักของ HAProxy คือการรับคำขอจากไคลเอนต์และส่งต่อคำขอนั้นไปยังเซิร์ฟเวอร์แบ็กเอนด์หลายตัว ทำให้สามารถปรับปรุงประสิทธิภาพ ความน่าเชื่อถือ และความพร้อมใช้งานของแอปพลิเคชันของคุณได้

**2. การติดตั้ง HAProxy:**

ขั้นตอนการติดตั้งจะแตกต่างกันไปขึ้นอยู่กับระบบปฏิบัติการที่คุณใช้งาน นี่คือตัวอย่างสำหรับระบบปฏิบัติการยอดนิยม:

  * **Ubuntu/Debian:**
    เปิด Terminal และรันคำสั่ง:
    ```bash
    sudo apt update
    sudo apt install haproxy
    ```
  * **CentOS/RHEL:**
    เปิด Terminal และรันคำสั่ง:
    ```bash
    sudo yum install haproxy
    ```
  * **RHEL 9:** ตามที่ระบุในผลการค้นหา คุณสามารถติดตั้งได้ด้วยคำสั่งเดียวกันกับ CentOS/RHEL หรืออาจมีขั้นตอนเพิ่มเติมเกี่ยวกับการตั้งค่า Hostname และ `/etc/hosts` ซึ่งโดยทั่วไปแล้วไม่ซับซ้อน

**3. การกำหนดค่า HAProxy:**

ไฟล์กำหนดค่าหลักของ HAProxy คือ `haproxy.cfg` ซึ่งโดยทั่วไปจะอยู่ที่ `/etc/haproxy/haproxy.cfg` คุณจะต้องแก้ไขไฟล์นี้เพื่อกำหนดวิธีการทำงานของ HAProxy

  * **โครงสร้างพื้นฐานของไฟล์ `haproxy.cfg`:**
    ไฟล์นี้แบ่งออกเป็นส่วนต่างๆ หลักๆ ได้แก่:

      * **`global`**: การตั้งค่าระดับโลก เช่น ผู้ใช้ กลุ่มผู้ใช้ และการตั้งค่าความปลอดภัย
      * **`defaults`**: การตั้งค่าเริ่มต้นสำหรับส่วน `frontend`, `backend`, และ `listen`
      * **`frontend`**: กำหนดวิธีการที่ HAProxy รับคำขอจากไคลเอนต์ (เช่น IP address และ port ที่ HAProxy จะรอรับฟัง)
      * **`backend`**: กำหนดกลุ่มของเซิร์ฟเวอร์แบ็กเอนด์ที่จะรับคำขอ
      * **`listen`**: เป็นการรวมเอาการตั้งค่าของ `frontend` และ `backend` ไว้ในส่วนเดียวกัน เหมาะสำหรับกรณีที่ไม่ซับซ้อน

  * **ตัวอย่างการกำหนดค่าพื้นฐาน:**
    สมมติว่าคุณต้องการกระจายโหลด HTTP ไปยังเซิร์ฟเวอร์สองเครื่องที่ IP `192.168.1.10:80` และ `192.168.1.11:80` นี่คือตัวอย่างการกำหนดค่า:

    ```cfg
    global
        log     127.0.0.1 local2
        user    haproxy
        group   haproxy
        daemon

    defaults
        mode    http
        log     global
        option  httplog
        option  dontlognull
        retries 3
        timeout client 10s
        timeout connect 5s
        timeout server 10s
        timeout queue 1m

    frontend http_frontend
        bind *:80
        default_backend http_backend

    backend http_backend
        balance roundrobin
        server server1 192.168.1.10:80 check
        server server2 192.168.1.11:80 check
    ```

    **คำอธิบาย:**

      * **`frontend http_frontend`**: กำหนดส่วน Frontend ชื่อ `http_frontend`
          * **`bind *:80`**: ให้ HAProxy รอรับฟังคำขอที่พอร์ต 80 บนทุก IP address ของเครื่อง
          * **`default_backend http_backend`**: กำหนด Backend เริ่มต้นที่จะใช้คือ `http_backend`
      * **`backend http_backend`**: กำหนดส่วน Backend ชื่อ `http_backend`
          * **`balance roundrobin`**: กำหนดอัลกอริธึมการกระจายโหลดเป็นแบบ Round Robin (สลับเซิร์ฟเวอร์ตามลำดับ)
          * **`server server1 192.168.1.10:80 check`**: กำหนดเซิร์ฟเวอร์แบ็กเอนด์ชื่อ `server1` ที่ IP `192.168.1.10` พอร์ต `80` และใช้คำสั่ง `check` เพื่อตรวจสอบสถานะของเซิร์ฟเวอร์
          * **`server server2 192.168.1.11:80 check`**: กำหนดเซิร์ฟเวอร์แบ็กเอนด์ชื่อ `server2` ที่ IP `192.168.1.11` พอร์ต `80` และใช้คำสั่ง `check` เพื่อตรวจสอบสถานะของเซิร์ฟเวอร์

**4. การเริ่มต้นและเปิดใช้งาน HAProxy:**

หลังจากแก้ไขไฟล์ `haproxy.cfg` แล้ว คุณจะต้องเริ่มต้นหรือรีสตาร์ทบริการ HAProxy เพื่อให้การตั้งค่าใหม่มีผล

  * **Ubuntu/Debian:**
    ```bash
    sudo systemctl start haproxy
    sudo systemctl enable haproxy  # เพื่อให้ HAProxy เริ่มทำงานอัตโนมัติเมื่อบูตเครื่อง
    ```
    หากต้องการรีสตาร์ท:
    ```bash
    sudo systemctl restart haproxy
    ```
  * **CentOS/RHEL:**
    ```bash
    sudo systemctl start haproxy
    sudo systemctl enable haproxy  # เพื่อให้ HAProxy เริ่มทำงานอัตโนมัติเมื่อบูตเครื่อง
    ```
    หากต้องการรีสตาร์ท:
    ```bash
    sudo systemctl restart haproxy
    ```

**5. การตรวจสอบสถานะ HAProxy:**

คุณสามารถตรวจสอบสถานะของ HAProxy เพื่อดูว่าทำงานถูกต้องหรือไม่ และเซิร์ฟเวอร์แบ็กเอนด์มีสถานะเป็นอย่างไร

  * **ผ่าน `systemctl`:**

    ```bash
    sudo systemctl status haproxy
    ```

  * **หน้าสถิติ (Statistics Page):**
    HAProxy สามารถแสดงหน้าสถิติที่มีประโยชน์ ซึ่งคุณสามารถเปิดใช้งานได้ในการตั้งค่าไฟล์ `haproxy.cfg` ในส่วน `listen` หรือ `frontend` เพิ่มบล็อกการกำหนดค่าคล้ายกับนี้:

    ```cfg
    listen stats
        bind *:8080  # หรือพอร์ตอื่นๆ ที่คุณต้องการ
        mode http
        stats enable
        stats uri /stats  # URL ที่จะเข้าถึงหน้าสถิติ (เช่น your_haproxy_ip:8080/stats)
        stats realm HAPROXY\ Statistics
        stats auth admin:password  # เปลี่ยน admin:password เป็นชื่อผู้ใช้และรหัสผ่านของคุณ (ควรตั้งรหัสผ่านที่ปลอดภัย)
    ```

    หลังจากรีสตาร์ท HAProxy แล้ว คุณจะสามารถเข้าถึงหน้าสถิติผ่านเบราว์เซอร์ของคุณได้

**6. การทดสอบ:**

เมื่อ HAProxy ทำงานแล้ว คุณสามารถทดสอบโดยการส่งคำขอไปยัง IP address ของเครื่องที่ติดตั้ง HAProxy (บนพอร์ตที่คุณกำหนดใน `frontend`) HAProxy ควรจะกระจายคำขอไปยังเซิร์ฟเวอร์แบ็กเอนด์ของคุณ

**ขั้นตอนถัดไป:**

เมื่อคุณเข้าใจพื้นฐานแล้ว คุณสามารถศึกษาเพิ่มเติมเกี่ยวกับ:

  * **Algorithm การกระจายโหลดอื่นๆ**: เช่น `leastconn`, `first`, `uri` เป็นต้น
  * **การตรวจสอบสถานะเซิร์ฟเวอร์ (Health Checks) ขั้นสูง**: เพื่อให้ HAProxy ตรวจสอบสถานะของเซิร์ฟเวอร์แบ็กเอนด์ได้อย่างละเอียดมากขึ้น
  * **การจัดการ SSL/TLS**: สำหรับแอปพลิเคชันที่ใช้ HTTPS
  * **Access Control Lists (ACLs)**: เพื่อกำหนดเงื่อนไขในการส่งต่อคำขอ
  * **การตั้งค่า Logging**: เพื่อบันทึกการทำงานของ HAProxy

คู่มือนี้เป็นเพียงจุดเริ่มต้น หวังว่าจะเป็นประโยชน์ในการเริ่มต้นใช้งาน HAProxy นะครับ หากมีคำถามเพิ่มเติม ถามได้เลยครับ
