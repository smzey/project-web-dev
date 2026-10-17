# คนที่ 1 — Accounts และ Categories

ชื่อสมาชิก: __________  
Branch แนะนำ: `member-1/accounts-categories`

รับผิดชอบ foundation ของโปรเจกต์, Login/Logout, role และ Category CRUD อ่าน [ข้อตกลงร่วม](README.md) ก่อนเริ่ม

## งานที่ต้องทำ

1. สร้าง Laravel application ใหม่ตามโครงสร้าง repository ที่กลุ่มตกลง ห้ามคัดลอกโปรเจกต์แล็บมาแก้ชื่อ
2. ทำ `.env.example` และวิธีติดตั้ง/รัน โดยไม่ใส่ secrets หรือ commit `.env`
3. เพิ่ม role `ADMIN` / `MEMBER` ใน User และสร้าง demo accounts ด้วย seeder ใช้รหัสที่ hash แล้ว; ไม่ต้องทำหน้าสมัครบัญชีหรือ CRUD User ในขอบเขตนี้
4. ทำหน้า login, ตรวจ credentials, regenerate session เมื่อสำเร็จ และ logout แบบ POST พร้อม invalidate session/regenerate CSRF token
5. กำหนด `auth` สำหรับ protected routes และตัวอย่าง policy ที่คนอื่นใช้ต่อได้ ไม่ปิด public activity pages ด้วย auth
6. ทำ Category CRUD ครบ พร้อม validation, policy, success/error session messages และจัดการกรณีลบหมวดที่ยังมีกิจกรรม
7. รวม routes/ไฟล์ตั้งค่ากลางจากเพื่อน และช่วยตรวจสิทธิ์หลังรวมงาน

## ไฟล์ที่รับผิดชอบ

ตำแหน่งต่อไปนี้อ้างอิงจาก Laravel application root ที่จะสร้างใหม่

| กลุ่มไฟล์ | งาน |
|---|---|
| `app/Models/User.php` | role, helper ตรวจ ADMIN, relationships ของ User โดยประสานคนที่ 2–3 |
| `app/Models/Category.php` | ข้อมูลหมวดและ `activities()` |
| `database/migrations/*users*`, `*categories*` | schema ของ User/Category และ FK ที่เกี่ยวข้องกับตารางตัวเอง |
| `database/seeders/*` สำหรับบัญชีและหมวด | ข้อมูลเริ่มต้นและการเรียก seeder ตามลำดับ |
| `app/Http/Controllers/LoginController.php` | login/logout |
| `app/Http/Controllers/CategoryController.php` | Category CRUD |
| `app/Policies/CategoryPolicy.php` | เฉพาะ ADMIN จัดการหมวดได้ |
| `resources/views/logins/*`, `resources/views/categories/*` | login และหน้า CRUD หมวด |
| `routes/web.php`, `bootstrap/app.php` | รวม routes และการตั้งค่าที่จำเป็น |

อย่าเปิดให้ฟอร์มทั่วไปแก้ role/email ผ่าน mass assignment และไม่คืน password ด้วย `withInput()`

## หน้าและ routes ที่ต้องมี

| Method / URL | ใช้ทำอะไร | สิทธิ์ |
|---|---|---|
| GET / POST `/auth/login` | ฟอร์มและตรวจ login | Guest |
| POST `/auth/logout` | ออกจากระบบ | ผู้ที่ login |
| GET `/categories` | รายการหมวด | ADMIN |
| GET `/categories/create` | ฟอร์มเพิ่ม | ADMIN |
| POST `/categories` | บันทึกหมวดใหม่ | ADMIN |
| GET `/categories/{category}` | รายละเอียดและกิจกรรมในหมวด | ADMIN |
| GET `/categories/{category}/edit` | ฟอร์มแก้ไข | ADMIN |
| PUT `/categories/{category}` | บันทึกแก้ไข | ADMIN |
| DELETE `/categories/{category}` | ลบหมวดที่ไม่มี Activity | ADMIN |

`{category}` ใช้ code; route login ต้องมีชื่อ `login` และ `/create` ต้องอยู่ก่อน route parameter

## สิ่งที่ต้องส่งให้เพื่อน

- Foundation ที่ติดตั้งและ migrate ได้ พร้อม User/Category และบัญชีสอง role
- รูปแบบ policy, route names และวิธีตรวจ ADMIN ที่ตกลงใช้ร่วมกัน
- วิธีเรียก seeders และวิธีรันในเครื่องเพื่อน
- Category ตัวอย่างสำหรับคนที่ 2 และ User สำหรับคนที่ 2–3

รับ layout/CSS จากคนที่ 2 มาใช้ ไม่แก้ไฟล์ layout พร้อมกันโดยไม่ประสาน

## Checklist ว่างานเสร็จเมื่อไร

- [ ] ติดตั้งตามเอกสารในเครื่องใหม่และสร้างฐานทดสอบได้
- [ ] บัญชีทั้งสอง role login/logout ได้; รหัสผิดมีข้อความและไม่เก็บ password กลับไป
- [ ] Guest ถูกพาไป login เมื่อเข้าหน้าหมวด; MEMBER ถูกปฏิเสธแม้เปิด URL หรือส่ง request เอง
- [ ] ADMIN เพิ่ม/ดู/แก้/ลบ Category ได้จริง
- [ ] ชื่อว่างหรือ code ซ้ำแสดง validation error และคงค่าฟอร์มที่เหมาะสม
- [ ] ลบหมวดที่มีกิจกรรมไม่ได้และมีข้อความแจ้ง; หมวดว่างลบได้
- [ ] ทุกการเปลี่ยนข้อมูลแสดงข้อความผลลัพธ์ผ่าน layout กลาง

## ส่วนที่ต้องอธิบายในรายงานและ demo

อธิบายการสร้างโปรเจกต์, migrations/seeders, การ hash password, session ของ login/logout, ความต่างระหว่าง authentication กับ authorization และ Category → Activity จากนั้นสาธิต ADMIN จัดการหมวดและ MEMBER ถูกปฏิเสธ

เขียนหัวข้อ “งานที่รับผิดชอบ”, “วิธีพัฒนา”, “ไฟล์สำคัญ” และ “ผลการทดสอบ” พร้อม commit ของตัวเอง
