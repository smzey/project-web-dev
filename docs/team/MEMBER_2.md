# คนที่ 2 — Activities และหน้าตาเว็บ

ชื่อสมาชิก: __________  
Branch แนะนำ: `member-2/activities-ui`

รับผิดชอบ Activity CRUD, หน้ากิจกรรมสาธารณะ และ layout/CSS ที่ทุกคนใช้ อ่าน [ข้อตกลงร่วม](README.md) ก่อนเริ่ม

## งานที่ต้องทำ

1. ทำ Activity model/migration เชื่อม Category และ organizer (`User`) และเตรียม `registrations()` ให้คนที่ 3 ใช้
2. ทำหน้าแสดงรายการและรายละเอียดกิจกรรมที่ Guest เข้าได้ พร้อมแสดงชื่อหมวดและผู้จัดผ่าน relationships จริง
3. ทำ CRUD กิจกรรมสำหรับ ADMIN: เพิ่ม, ดู, แก้ไข และลบ พร้อมเลือก Category; กำหนดผู้จัดเป็น ADMIN ที่สร้างกิจกรรมผ่าน relationship ไม่รับ organizer_id จากผู้ใช้ทั่วไป
4. ทำ validation เช่น code ไม่ซ้ำ, ชื่อ/สถานที่/วันเวลาที่จำเป็น และหมวดต้องมีจริง
5. เพิ่มค้นหาจาก code/name และ pagination โดยรักษา query string ตอนเปลี่ยนหน้า; ใช้ eager loading สำหรับหมวดและผู้จัด
6. ทำ layout, navigation, CSS และส่วนแสดง session `success` / `error` กับ validation errors ที่ทุกคนใช้ร่วมกัน
7. ใส่ปุ่มไปหน้าสมัครที่คนที่ 3 ทำ: Guest เห็นลิงก์ login; คนที่ login เห็นปุ่มสมัครตามสถานะ
8. กันการลบ Activity ที่มี registrations และแสดง error ที่เข้าใจได้

## ไฟล์ที่รับผิดชอบ

ตำแหน่งต่อไปนี้อ้างอิงจาก Laravel application root ที่จะสร้างใหม่

| กลุ่มไฟล์ | งาน |
|---|---|
| `app/Models/Activity.php` | `category()`, `organizer()`, `registrations()` |
| `database/migrations/*activities*` | schema และ category_id/organizer_id foreign keys |
| `database/seeders/ActivitySeeder.php` | กิจกรรมตัวอย่างหลัง User/Category พร้อม |
| `app/Http/Controllers/ActivityController.php` | หน้าสาธารณะ, CRUD, search/pagination |
| `app/Policies/ActivityPolicy.php` | public read และ ADMIN-only writes |
| `resources/views/activities/*` | list/detail/create/edit และฟอร์มร่วม |
| `resources/views/layouts/main.blade.php` | layout, เมนูตามสิทธิ์, flash messages |
| `public/css/app.css` | CSS ที่กลุ่มเขียนเองและใช้ทั้งเว็บ |

ใช้แนวทางค้นหาจากบทเรียนได้ แต่เขียนสำหรับโปรเจกต์ใหม่นี้; ไม่จำเป็นต้องเพิ่ม abstract controller ถ้าไม่ได้ช่วยให้งานง่ายขึ้น

## หน้าและ routes ที่ต้องมี

| Method / URL | ใช้ทำอะไร | สิทธิ์ |
|---|---|---|
| GET `/` | ไปหน้ารายการหรือแสดงกิจกรรม | Public |
| GET `/activities` | รายการ, search, pagination | Public |
| GET `/activities/create` | ฟอร์มเพิ่ม | ADMIN |
| POST `/activities` | บันทึกใหม่ | ADMIN |
| GET `/activities/{activity}` | รายละเอียดพร้อมหมวด/ผู้จัด | Public |
| GET `/activities/{activity}/edit` | ฟอร์มแก้ไข | ADMIN |
| PUT `/activities/{activity}` | บันทึกแก้ไข | ADMIN |
| DELETE `/activities/{activity}` | ลบกิจกรรมที่ไม่มี registrations | ADMIN |

`{activity}` ใช้ code; ถ้าใช้ policy สำหรับ public read ต้องรองรับผู้ใช้ที่ยังไม่ login ด้วย และห้ามครอบ public GET routes ด้วย `auth`

## สิ่งที่ต้องรับและส่งต่อ

- รับ User/Category fields, demo accounts และตัวอย่าง policy จากคนที่ 1
- ส่ง layout ที่ใช้งานได้ให้เพื่อนตั้งแต่ต้น พร้อมตัวอย่าง `@extends`, section และชื่อ session keys
- แจ้ง route names ของ activity list/detail และส่งกิจกรรมตัวอย่างให้คนที่ 3
- รับชื่อ route สมัคร/ยกเลิกจากคนที่ 3 เพื่อเชื่อมปุ่ม และส่ง routes ของตัวเองให้คนที่ 1 รวม
- ช่วยเพื่อนปรับหน้าให้ใช้ CSS กลาง โดยแต่ละคนยังรับผิดชอบฟอร์มของตัวเอง

## Checklist ว่างานเสร็จเมื่อไร

- [ ] Guest ดูรายการและรายละเอียดได้โดยไม่ต้อง login
- [ ] หน้าแสดงชื่อหมวดและผู้จัดจาก relationships จริง
- [ ] ADMIN เพิ่ม/ดู/แก้/ลบ Activity ได้ครบ
- [ ] MEMBER และ Guest ส่ง request เพิ่ม/แก้/ลบเองไม่ได้
- [ ] code ซ้ำ, หมวดที่ไม่มีจริง และข้อมูลจำเป็นที่หายไปมี validation errors
- [ ] ลบกิจกรรมที่มี registrations ไม่ได้; กิจกรรมว่างลบได้
- [ ] Search/pagination ทำงานและรักษาคำค้นเมื่อเปลี่ยนหน้า
- [ ] ทุกหน้าใช้ Blade และ CSS กลาง; เมนูและปุ่มตรงกับสิทธิ์
- [ ] ทุก CRUD แสดง success/error และฟอร์มคืนค่าด้วย `old()` อย่างเหมาะสม
- [ ] ลองหน้าจอมือถือและ desktop จริง พร้อมตรวจตาราง/ฟอร์มไม่ล้นหน้าจอ

## ส่วนที่ต้องอธิบายในรายงานและ demo

อธิบาย Activity → Category/User, validation, search/pagination, public routes เทียบกับ ADMIN actions และการใช้ Blade layout/session messages จากนั้นสาธิต Guest เปิดเว็บและ ADMIN ทำ CRUD

เขียนหัวข้อ “งานที่รับผิดชอบ”, “วิธีพัฒนา”, “ไฟล์สำคัญ” และ “ผลการทดสอบ” พร้อม commit ของตัวเอง
