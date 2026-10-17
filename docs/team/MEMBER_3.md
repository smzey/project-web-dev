# คนที่ 3 — Registrations และ Export

ชื่อสมาชิก: __________  
Branch แนะนำ: `member-3/registrations-export`

รับผิดชอบ Registration CRUD และ PHP library ที่ใช้ export รายชื่อผู้สมัคร อ่าน [ข้อตกลงร่วม](README.md) ก่อนเริ่ม

## งานที่ต้องทำ

1. ทำ Registration model/migration เชื่อม Activity และ User พร้อม code ไม่ซ้ำและ unique constraint คู่ `(activity_id, user_id)`
2. ทำฟอร์มสมัครกิจกรรมของผู้ใช้ที่ login แล้ว โดยผูก User จาก session ผ่าน relationship ห้ามเชื่อ user_id ที่ส่งจากฟอร์ม
3. ทำรายการและรายละเอียดการสมัคร: MEMBER เห็นของตัวเองเท่านั้น; ADMIN เห็นทั้งหมดและกรองตามกิจกรรมได้
4. ทำหน้าแก้ note และลบการสมัครเพื่อยกเลิก ให้ครบ Update และ Delete จริง ไม่ใช้แค่ปุ่มเปลี่ยนสถานะแทน Delete
5. ทำ policy ตรวจเจ้าของ record ทุกครั้งที่ view/update/delete และ query รายการต้องกรองสิทธิ์ด้วย ป้องกันการเดา code ไปดูของคนอื่น
6. ตรวจการสมัครซ้ำทั้งก่อนบันทึกและด้วย unique constraint; จับกรณี request ซ้ำแล้วแจ้ง error ผ่าน session
7. เลือก PHP library สำหรับสร้าง CSV ผ่าน Composer ตรวจว่าไม่ได้สอนในคาบก่อนเลือก รวม Week 01–06; ยังไม่ได้กำหนด package ในแผนนี้
8. ทำ ADMIN-only export รายชื่อผู้สมัครของกิจกรรมจริง โดยเรียกใช้ library นั้น ไม่ใช่แค่ติดตั้งไว้
9. ทำเอกสารติดตั้ง/ตั้งค่า/เรียกใช้ library และรวบรวมผลทดสอบส่วนของตัวเอง

## ไฟล์ที่รับผิดชอบ

ตำแหน่งต่อไปนี้อ้างอิงจาก Laravel application root ที่จะสร้างใหม่

| กลุ่มไฟล์ | งาน |
|---|---|
| `app/Models/Registration.php` | `activity()` และ `user()` |
| `database/migrations/*registrations*` | schema, FK, code unique และคู่ activity/user unique |
| `database/seeders/RegistrationSeeder.php` | การสมัครตัวอย่างหลัง Activity/User พร้อม |
| `app/Http/Controllers/RegistrationController.php` | สมัคร, list/detail, แก้ note, ยกเลิก |
| `app/Policies/RegistrationPolicy.php` | เจ้าของ record หรือ ADMIN |
| `app/Http/Controllers/RegistrationExportController.php` | ตรวจ ADMIN, query และดาวน์โหลด CSV |
| `resources/views/registrations/*` | หน้า CRUD และปุ่ม export ตามสิทธิ์ |
| `composer.json`, `composer.lock` | PHP library ที่เลือก โดยประสานคนที่ 1 |
| `docs/EXPORT_LIBRARY.md` ที่ต้องสร้างเมื่อเลือก library | วิธีติดตั้ง/ตั้งค่า/ใช้ พร้อมเหตุผลที่นับ R6 ได้ |

## หน้าและ routes ที่ต้องมี

| Method / URL | ใช้ทำอะไร | สิทธิ์ |
|---|---|---|
| GET `/activities/{activity}/register` | ฟอร์มสมัคร | ผู้ที่ login |
| POST `/activities/{activity}/registrations` | สมัครให้ตัวเอง | ผู้ที่ login |
| GET `/registrations` | MEMBER เห็นของตน; ADMIN เห็นทั้งหมด | ผู้ที่ login |
| GET `/registrations/{registration}` | รายละเอียด | เจ้าของหรือ ADMIN |
| GET `/registrations/{registration}/edit` | ฟอร์มแก้ note | เจ้าของหรือ ADMIN |
| PUT `/registrations/{registration}` | บันทึก note | เจ้าของหรือ ADMIN |
| DELETE `/registrations/{registration}` | ยกเลิกโดยลบ record | เจ้าของหรือ ADMIN |
| GET `/activities/{activity}/registrations/export` | ดาวน์โหลด CSV ของกิจกรรมนั้น | ADMIN |

ทั้ง `{activity}` และ `{registration}` ใช้ code การแก้ไขรับเฉพาะ note; ไม่อนุญาตให้เปลี่ยน user_id/activity_id หรือ code ผ่าน request แก้ไข

## ขอบเขต Export

- ข้อมูลขั้นต่ำ: registration code, activity code/name, participant name/email และวันเวลาสมัคร
- Export เฉพาะกิจกรรมที่เลือก โดยใช้ relationships จริงและ eager loading ตามเหมาะสม
- ตรวจสิทธิ์ที่ server แม้ซ่อนปุ่มจาก MEMBER แล้ว
- CSV เปิดได้และภาษาไทยอ่านได้; ทดสอบชื่อที่มี comma, quote และขึ้นบรรทัดใหม่
- ใช้วิธีเขียน CSV ของ library และจัดการข้อความที่ spreadsheet อาจตีความเป็นสูตรเมื่อ export ข้อมูลผู้ใช้
- เอกสาร library ต้องระบุ package/version จาก `composer.lock`, คำสั่งติดตั้งจริง, การตั้งค่าที่ใช้ (หรือระบุว่าไม่ต้องตั้งค่า), จุดเรียกในโค้ด และขั้นตอน demo
- อย่านับ PSR-7 bridge, Laravel built-ins หรือ package ที่สอนแล้วเป็น R6

## สิ่งที่ต้องรับและส่งต่อ

- รับ User/demo accounts จากคนที่ 1 และ Activity/schema/layout จากคนที่ 2
- ประสานคนที่ 1–2 ให้ User/Activity มี `registrations()` และลำดับ migrations/seeders ถูกต้อง
- ส่ง route names และสถานะสมัครแล้วให้คนที่ 2 เชื่อมกับหน้ารายละเอียดกิจกรรม
- ส่ง routes ให้คนที่ 1 รวม พร้อมแจ้ง Composer dependency ที่เพิ่ม
- ส่งตัวอย่าง CSV และเอกสาร library ให้กลุ่มใช้ซ้อมนำเสนอ

## Checklist ว่างานเสร็จเมื่อไร

- [ ] Guest สมัครไม่ได้และถูกพาไป login
- [ ] MEMBER สมัคร ดูรายละเอียด แก้ note และลบการสมัครของตัวเองได้ครบ
- [ ] MEMBER ดู/แก้/ลบของคนอื่นไม่ได้ แม้เปลี่ยน URL หรือส่ง request เอง
- [ ] ADMIN ดูและจัดการการสมัครทั้งหมดได้
- [ ] สมัครซ้ำไม่ได้ รวมถึงการส่ง request ซ้ำ; ยกเลิกแล้วสมัครใหม่ได้
- [ ] เปลี่ยน user_id/activity_id ผ่าน request ไม่ได้
- [ ] ทุกการเปลี่ยนข้อมูลแสดง success/error และ validation ทำงาน
- [ ] MEMBER export ไม่ได้แม้เรียก URL โดยตรง; ADMIN export ข้อมูลตรงกับฐานข้อมูล
- [ ] ไฟล์ CSV เปิดอ่านได้ รวมชื่อภาษาไทยและตัวอักษรพิเศษ
- [ ] ใช้ PHP library จริง ตรวจว่าไม่ได้สอน และมีเอกสารอธิบายครบ

## ส่วนที่ต้องอธิบายในรายงานและ demo

อธิบาย Registration → Activity/User, unique constraint, การตรวจเจ้าของ record และ CRUD ของการสมัคร จากนั้นสาธิตสองบัญชี MEMBER เพื่อพิสูจน์สิทธิ์ และใช้ ADMIN export CSV พร้อมชี้จุดเรียก library ในโค้ด

เขียนหัวข้อ “งานที่รับผิดชอบ”, “วิธีพัฒนา”, “ไฟล์สำคัญ” และ “ผลการทดสอบ” พร้อม commit ของตัวเอง
