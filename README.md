# โครงการ: Postgres Column Encryption (pgcrypto) + Jupyter/ORM

โปรเจกต์นี้สาธิตการเข้ารหัสข้อมูลระดับคอลัมน์ใน PostgreSQL ด้วยส่วนขยาย `pgcrypto` ทั้งในรูปแบบ SQL ตรงๆ และผ่าน ORM (SQLAlchemy) โดยเตรียมสภาพแวดล้อมด้วย Docker และตัวอย่างโน้ตบุ๊ก Jupyter สำหรับทดลองใช้งาน

## คุณสมบัติหลัก
- Docker Compose สำหรับรัน PostgreSQL ในเครื่อง (เปิดใช้ `pgcrypto` อัตโนมัติ)
- โน้ตบุ๊ก Jupyter สาธิตการเข้ารหัส/ถอดรหัสด้วย SQL โดยตรง
- โน้ตบุ๊ก Jupyter สาธิตการใช้งานผ่าน ORM (SQLAlchemy) ด้วย `TypeDecorator`

## โครงสร้างสำคัญของโปรเจกต์
- `docker-compose.yml` — คอนฟิก Docker Compose สำหรับฐานข้อมูล
- `docker/Dockerfile` — อิมเมจ PostgreSQL พร้อมสคริปต์ init
- `docker/initdb/01-pgcrypto.sql` — เปิดใช้ส่วนขยาย `pgcrypto` ตอนสร้างฐานข้อมูลครั้งแรก
- `requirements.txt` — ไลบรารีจำเป็นสำหรับรันโน้ตบุ๊กและเชื่อมต่อฐานข้อมูล
- `notebooks/postgres_column_encryption_test.ipynb` — เดโม SQL เข้ารหัส/ถอดรหัสด้วย `pgcrypto`
- `notebooks/orm_postgres_encryption.ipynb` — เดโม ORM (SQLAlchemy 2.x) เข้ารหัส/ถอดรหัสอัตโนมัติ

## ข้อกำหนดเบื้องต้น
- Docker และ Docker Compose
- Python 3.11+ (โครงการนี้ทดสอบกับ 3.13)
- แนะนำให้ใช้ virtual environment (เช่น `.venv`)

## เริ่มต้นใช้งาน
1) สร้างและเปิดใช้งาน virtual environment (ตัวอย่าง PowerShell)
```
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
```
ติดตั้งไลบรารี
```
pip install -r requirements.txt
```

2) เริ่มฐานข้อมูล PostgreSQL
```
docker-compose up -d
```
ค่าเริ่มต้นที่ใช้ในคอนเทนเนอร์:
- User: `encuser`
- Password: `encpass`
- Database: `encdb`
- Port: `5432` (แมปกับเครื่อง)

3) เปิด Jupyter Notebook
```
jupyter notebook
```
แล้วเปิดไฟล์ในโฟลเดอร์ `notebooks/`:
- `postgres_column_encryption_test.ipynb` — เดโม SQL:
  - สร้างตารางตัวอย่าง
  - แทรกข้อมูลด้วย `pgp_sym_encrypt`
  - อ่านข้อมูลด้วย `pgp_sym_decrypt`
- `orm_postgres_encryption.ipynb` — เดโม ORM (SQLAlchemy):
  - ใช้ `TypeDecorator` เพื่อเข้ารหัสก่อนบันทึกและถอดรหัสเมื่ออ่าน
  - มีเซกชันแสดงข้อมูลที่ถูกเข้ารหัส (ciphertext) และข้อมูลที่ถอดรหัสแล้ว

## การตั้งค่าผ่าน Environment Variables (โน้ตบุ๊กทั้งสองไฟล์รองรับ)
- `PG_HOST` ค่าเริ่มต้น `localhost`
- `PG_PORT` ค่าเริ่มต้น `5432`
- `PG_DB` ค่าเริ่มต้น `encdb`
- `PG_USER` ค่าเริ่มต้น `encuser`
- `PG_PASSWORD` ค่าเริ่มต้น `encpass`
- `PG_PASSPHRASE` ค่าเริ่มต้น `my-strong-demo-passphrase` (เพื่อการสาธิต)

## หมายเหตุด้านความปลอดภัย
- ตัวอย่างนี้ใช้ passphrase แบบกำหนดใน ENV เพื่อความง่ายในการทดสอบเท่านั้น
- ในระบบจริงควรจัดเก็บคีย์ใน Secret Manager/Key Management และจำกัดสิทธิ์อย่างเหมาะสม
- การเข้ารหัสด้วย `pgp_sym_encrypt/pgp_sym_decrypt` เก็บข้อมูลเป็นชนิด `bytea` ในตาราง
- การทำดัชนี (index) บนข้อมูลที่เข้ารหัสโดยตรงทำได้จำกัด — หากต้องค้นหาแบบเท่ากัน/ช่วง ให้พิจารณาเก็บแฮชแยกต่างหาก

## คำสั่งที่เป็นประโยชน์
- ดูสถานะคอนเทนเนอร์: `docker ps`
- ดู log ของ Postgres: `docker-compose logs -f postgres`
- หยุดและลบคอนเทนเนอร์: `docker-compose down`

## ใบอนุญาต
โปรเจกต์ตัวอย่างเพื่อการศึกษา ไม่ระบุสัญญาอนุญาตอย่างเป็นทางการ

