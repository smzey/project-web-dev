# แบ่งงาน 3 คน — ระบบสมัครเข้าร่วมกิจกรรม

เอกสารนี้ขยายแผนที่แบ่งงานในกลุ่ม ยังไม่ใช่หลักฐานว่าพัฒนาฟีเจอร์เสร็จแล้ว ปัจจุบัน repository มีเอกสารอ้างอิงและแผน ยังไม่มี Laravel application

ใช้ Laravel + Blade และสร้างโปรเจกต์ใหม่ ห้ามนำโปรเจกต์จากแล็บมาแก้ต่อ ถ้าขัดกับ [กฎโปรเจกต์](../../source/PROJECT_RULES.md) ให้ยึดกฎและคำสั่งอาจารย์

## ใครทำอะไร

| สมาชิก | งานหลัก | เอกสารประจำตัว |
|---|---|---|
| คนที่ 1 — ชื่อ: __________ | ตั้งโปรเจกต์, Login/Logout, สิทธิ์, Category CRUD | [งานคนที่ 1](MEMBER_1.md) |
| คนที่ 2 — ชื่อ: __________ | Activity CRUD, หน้าสาธารณะ, Blade layout, CSS | [งานคนที่ 2](MEMBER_2.md) |
| คนที่ 3 — ชื่อ: __________ | Registration CRUD, Export, เอกสาร PHP library | [งานคนที่ 3](MEMBER_3.md) |

ทุกคนทำ model, migration, controller, routes, policy และ Blade pages ของฟีเจอร์ตัวเอง ทุกคนต้องอธิบายงานตัวเองและภาพรวมของระบบได้

## ข้อตกลงร่วมสำหรับเริ่มพัฒนา

รายละเอียดต่อไปนี้เป็นแบบเริ่มต้นจากแผนแบ่งงาน ไม่ใช่ requirement เพิ่มจากอาจารย์ หากกลุ่มปรับชื่อหรือฟิลด์ ให้แก้เอกสารนี้และแจ้งอีกสองคนก่อน

| Model / เจ้าของ | ฟิลด์หลัก นอกเหนือจาก id และ timestamps | Relationships |
|---|---|---|
| User / คนที่ 1 | name, email unique, password, role (`ADMIN` หรือ `MEMBER`) | hasMany activities ผ่าน organizer_id; hasMany registrations |
| Category / คนที่ 1 | code unique, name, description nullable | hasMany activities |
| Activity / คนที่ 2 | code unique, name, description, location, starts_at, category_id, organizer_id | belongsTo category; belongsTo organizer (`User`); hasMany registrations |
| Registration / คนที่ 3 | code unique, activity_id, user_id, note nullable | belongsTo activity; belongsTo user |

- Model ที่นับเป็นงาน custom สามตัวคือ Category, Activity และ Registration; User ไม่นับ
- ใช้ code ใน URL ของทั้งสาม resource เช่น `/activities/AC001` ไม่ใช้ id; FK ในฐานข้อมูลยังเป็น id
- ตั้ง FK ผ่าน relationship `associate()` ตามแนวทางที่เรียน
- Registration มี unique constraint คู่ `(activity_id, user_id)` และตรวจซ้ำก่อนบันทึก เพื่อไม่ให้คนเดียวสมัครกิจกรรมเดียวซ้ำ
- ห้ามลบ Category ที่ยังมี Activity และห้ามลบ Activity ที่ยังมี Registration ใช้ทั้งการตรวจในระบบและ FK แบบ restrict พร้อมข้อความ error ที่เข้าใจได้
- การลบ Registration หมายถึงยกเลิกการสมัคร ลบจริงเพื่อให้สาธิต Delete ได้
- การแก้ไข Registration หมายถึงแก้ note ไม่เปิดให้เปลี่ยนเจ้าของหรือกิจกรรมผ่านฟอร์มแก้ไข
- ยังไม่เพิ่มระบบชำระเงิน, QR, จำกัดที่นั่ง หรือ CRUD ผู้ใช้ เพราะไม่จำเป็นต่อขอบเขตนี้

## สิทธิ์ที่ทุกคนต้องใช้ตรงกัน

| การใช้งาน | Guest | MEMBER | ADMIN |
|---|---|---|---|
| ดูหน้ารายการ/รายละเอียดกิจกรรม | ได้ | ได้ | ได้ |
| จัดการ Category และ Activity | ไม่ได้ | ไม่ได้ | ได้ |
| สมัครกิจกรรม และดู/แก้ note/ลบการสมัครของตัวเอง | ต้อง login | ได้ | ได้ |
| ดู/แก้ note/ลบการสมัครของผู้อื่น | ไม่ได้ | ไม่ได้ | ได้ |
| Export รายชื่อผู้สมัครของกิจกรรม | ไม่ได้ | ไม่ได้ | ได้ |

ADMIN สร้างการสมัครให้ตัวเองได้เหมือน MEMBER; ไม่ต้องเพิ่มฟอร์มสมัครแทนคนอื่น หน้า login ไม่ใช่ public page ที่ใช้สาธิตเพียงหน้าเดียว ต้องมีหน้ากิจกรรมสาธารณะด้วย

คนที่ 1 ดูแลกลไก auth และรูปแบบ policy ร่วม ส่วนเจ้าของ resource เขียน policy ของตัวเอง ห้ามตรวจสิทธิ์เฉพาะการซ่อนปุ่ม ต้องตรวจที่ server ด้วย

## ข้อตกลงเรื่องไฟล์ร่วม

- คนที่ 1 สร้าง Laravel foundation และดูแลการรวม `routes/web.php`, `bootstrap/app.php` และไฟล์ตั้งค่าร่วม
- คนที่ 2 ดูแล `resources/views/layouts/main.blade.php` และ `public/css/app.css`; อีกสองคนใช้ layout นี้และส่งคำขอแก้ไขให้เจ้าของ
- ทุก controller ใช้ session key `success` และ `error` ให้ตรงกัน; validation ใช้ `$errors`, `old()` และ `withInput()` โดยไม่เก็บ password กลับไป
- คนที่ 3 ดูแลการเพิ่ม PHP library ใน `composer.json` และ `composer.lock` หลัง foundation พร้อม
- เจ้าของแต่ละ module ส่ง route definitions ให้คนที่ 1 รวม หรือแก้ใน branch แล้วให้คนที่ 1ตรวจตอน merge; วาง route คงที่ก่อน `{code}` เสมอ
- ทุกฟอร์มที่เปลี่ยนข้อมูลมี `@csrf`; Delete ใช้ form พร้อม `@method('DELETE')`
- ใช้ `ServerRequestInterface` ตามแนวทางในเอกสารเรียน และ validation ฝั่ง server ทุกครั้งที่รับข้อมูลเพื่อบันทึก

## ลำดับทำงานและส่งต่องาน

1. ใส่ชื่อเจ้าของงาน และทบทวนตาราง fields/สิทธิ์ร่วมกัน
2. คนที่ 1 สร้างโปรเจกต์ใหม่, ตั้งฐานข้อมูล, User/Category migrations และ demo accounts แล้วแชร์วิธีติดตั้ง โดยไม่ commit `.env`
3. คนที่ 2 เพิ่ม Activity migration หลัง User/Category; คนที่ 3 เพิ่ม Registration migration หลัง Activity ทุกคนเริ่มเขียนหน้าและ logic ของตัวเองคู่ขนานได้เมื่อชื่อตารางและฟิลด์ตรงกัน
4. คนที่ 2 ส่ง layout และตัวอย่าง form/message ให้ทุกคนใช้; คนที่ 1 ส่งตัวอย่าง policy และวิธีเรียก auth
5. รวม Category → Activity → Registration → Export ตรวจระบบหลังรวมแต่ละส่วน และให้แต่ละคน commit งานตัวเอง
6. ทั้งกลุ่มทดสอบจากฐานข้อมูลใหม่ที่เป็นฐานทดสอบเท่านั้น สร้างข้อมูลตามลำดับ และซ้อม demo ตั้งแต่ Guest → MEMBER → ADMIN

แนะนำชื่อ branch: `member-1/accounts-categories`, `member-2/activities-ui`, `member-3/registrations-export`

## ใครรับผิดชอบ requirement ไหน

| Requirement | ผู้รับผิดชอบ |
|---|---|
| R1 — 3 custom models และ relationships ใช้จริง | ทุกคนทำของตัวเอง; คนที่ 1 ช่วยตรวจ schema รวม |
| R2 — CRUD ครบ 3 models | คนที่ 1 Category; คนที่ 2 Activity; คนที่ 3 Registration |
| R3 — Session notifications และ errors | ทุกคนส่งผลจาก controller; คนที่ 2 ทำส่วนแสดงผลกลาง |
| R4 — Authentication, 2 roles, public/protected routes | คนที่ 1 auth; คนที่ 2 public pages; ทุกคนป้องกัน resource ของตัวเอง |
| R5 — CSS | คนที่ 2 ทำรูปแบบกลาง; ทุกคนใช้กับหน้าตัวเอง |
| R6 — PHP third-party library | คนที่ 3 เลือกและใช้จริง พร้อมตรวจว่าไม่ได้สอนในคาบ |

## ตรวจร่วมกันก่อนส่ง

- [ ] CRUD ครบทั้งสาม model และเห็น relationships ผ่านหน้าเว็บจริง
- [ ] มีข้อความสำเร็จทุก create/update/delete และข้อความเมื่อ validation/ฐานข้อมูลผิดพลาด
- [ ] Guest เปิดหน้ากิจกรรมได้ แต่เข้า protected routes ไม่ได้
- [ ] MEMBER เข้า URL จัดการของ ADMIN และแก้ registration ของคนอื่นไม่ได้ แม้ส่ง request เอง
- [ ] ADMIN จัดการข้อมูลและ export รายชื่อได้
- [ ] ทุกหน้าเป็น Blade โหลดผ่าน route ปกติ และใช้ CSS กลาง
- [ ] Library เป็น PHP ติดตั้งผ่าน Composer และไม่ได้สอน รวมถึงตรวจ Week 01–06 ที่เอกสารปัจจุบันยังไม่ครอบคลุม
- [ ] ทุกคนเขียนส่วนงานและวิธีพัฒนาในรายงาน พร้อมซ้อมอธิบายโค้ดตัวเอง

## เอกสารอ้างอิง

- [แผนโปรเจกต์](../../source/PROJECT_PLAN.md)
- [กฎและเกณฑ์คะแนน](../../source/PROJECT_RULES.md)
- [สรุปเว็บตัวอย่าง my-db](../../source/my-db-project-summary.md)
- [Database / CRUD / Relationships — Week 07–09](../../source/954348-week07-08-09-database-combined.md)
- [Relationships / Session / Auth / Errors — Week 10–13](../../source/Laravel_Week10-11-12-13_Combined.md)
