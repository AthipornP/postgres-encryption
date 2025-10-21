# โครงการ: Postgres Column Encryption (pgcrypto) + Jupyter/ORM

โปรเจกต์ตัวอย่างที่แสดงการปกป้องข้อมูลระดับคอลัมน์ใน PostgreSQL โดยใช้ส่วนขยาย `pgcrypto` ทั้งแบบ symmetric (pgp_sym_encrypt/pgp_sym_decrypt) และแบบกุญแจสาธารณะ (pgp_pub_encrypt/pgp_pub_decrypt) — มีตัวอย่างทั้งแบบ SQL ดิบ และผ่าน ORM (SQLAlchemy) ด้วยชนิดข้อมูลกำหนดเอง (TypeDecorator)

## คุณสมบัติหลัก
- สาธิตการเข้ารหัส/ถอดรหัสในระดับคอลัมน์ของ PostgreSQL โดยใช้ `pgcrypto`
- ตัวอย่างการใช้งานทั้งจาก SQL ดิบ และผ่าน ORM (SQLAlchemy)
- เซ็ตอัพด้วย Docker Compose เพื่อรัน PostgreSQL ที่เปิดใช้ `pgcrypto` โดยอัตโนมัติ
- โน้ตบุ๊กสำหรับทดลอง: symmetric encryption, ORM integration และ public-key (PGP) flow

## โครงสร้างสำคัญของโปรเจกต์
- `docker-compose.yml` — คอนฟิก Docker Compose สำหรับรัน Postgres (service: `postgres`, container_name: `pg_enc_local`)
- `docker/Dockerfile` — อิมเมจ Postgres ที่คัดลอกสคริปต์เริ่มต้น
- `docker/initdb/01-pgcrypto.sql` — สคริปต์รันตอนสร้างฐานข้อมูลแรก เพื่อเปิดใช้ `pgcrypto`
- `requirements.txt` — ไลบรารีที่ใช้ในโน้ตบุ๊ก (`psycopg[binary]`, `SQLAlchemy`, `pandas`, `jupyter`, `pgpy` สำหรับตัวอย่างกุญแจสาธารณะ)
- `notebooks/postgres_column_encryption_test.ipynb` — เดโมการใช้ `pgp_sym_encrypt`/`pgp_sym_decrypt` ด้วย SQL ดิบ (symmetric)
- `notebooks/orm_postgres_encryption.ipynb` — เดโมการใช้งานผ่าน ORM โดยใช้ `TypeDecorator` เพื่อเข้ารหัส/ถอดรหัสแบบ symmetric
- `notebooks/orm_postgres_pubkey_encryption.ipynb` — เดโมการเข้ารหัสแบบกุญแจสาธารณะ (PGP) ร่วมกับ `pgpy` และ ORM

## ข้อกำหนดเบื้องต้น
- Docker และ Docker Compose
- Python 3.11+ (ทดสอบกับ 3.13)
- แนะนำให้ใช้ virtual environment เช่น `.venv`
- หากรันโน้ตบุ๊ก public-key (`orm_postgres_pubkey_encryption.ipynb`) จะต้องติดตั้ง `pgpy` และโน้ตบุ๊กนั้นมีการสร้างกุญแจตัวอย่างในหน่วยความจำเพื่อการสาธิต (อย่าสร้างกุญแจแบบนี้ในงานจริง). หมายเหตุ: ใน Python 3.13 มีการลบโมดูลมาตรฐาน `imghdr` — โน้ตบุ๊ก public-key มี shim เล็กน้อยเพื่อให้ `pgpy` ทำงานได้บนเวอร์ชันนั้น

## เริ่มต้นใช้งาน (สั้นๆ)
1) สร้างและเปิดใช้งาน virtual environment (PowerShell)
```
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
```
ติดตั้งไลบรารี
```
pip install -r requirements.txt
```

2) เริ่มฐานข้อมูล PostgreSQL (จะ build image และใช้สคริปต์ใน `docker/initdb/` ครั้งแรกที่สร้างฐาน)
```
docker-compose up -d
```
ค่าเริ่มต้นในคอนเทนเนอร์
- User: `encuser`
- Password: `encpass`
- Database: `encdb`
- Port: `5432` (แมปกับเครื่อง)

3) เปิด Jupyter Notebook และเปิดไฟล์ในโฟลเดอร์ `notebooks/`
```
jupyter notebook
```

ตัวอย่างโน้ตบุ๊ก
- `postgres_column_encryption_test.ipynb` — เดโม SQL (symmetric): สร้างตาราง, เพิ่มข้อมูลด้วย `pgp_sym_encrypt`, อ่านด้วย `pgp_sym_decrypt`
- `orm_postgres_encryption.ipynb` — เดโม ORM (symmetric): ใช้ `TypeDecorator` เพื่อเข้ารหัสก่อนบันทึกและถอดรหัสเมื่ออ่าน
- `orm_postgres_pubkey_encryption.ipynb` — เดโม public-key (PGP) ร่วมกับ `pgpy`: สาธิตการสร้างคู่กุญแจ (เพื่อการเรียนรู้เท่านั้น) และการใช้ `pgp_pub_encrypt`/`pgp_pub_decrypt`

## การตั้งค่าผ่าน Environment Variables
โน้ตบุ๊กรองรับการปรับค่าผ่านตัวแปรแวดล้อมต่อไปนี้ (มีค่าเริ่มต้นในโน้ตบุ๊กเพื่อความสะดวก)
- `PG_HOST` (default: `localhost`)
- `PG_PORT` (default: `5432`)
- `PG_DB` (default: `encdb`)
- `PG_USER` (default: `encuser`)
- `PG_PASSWORD` (default: `encpass`)
- `PG_PASSPHRASE` (default: `my-strong-demo-passphrase`) — ใช้สำหรับ symmetric encryption demo
- `PG_PRIVATE_PASSPHRASE` (default: `demo-private-passphrase`) — รหัสผ่านตัวอย่างสำหรับกุญแจส่วนตัวในโน้ตบุ๊ก public-key (ใช้เพื่อการสาธิตเท่านั้น)

## หมายเหตุด้านความปลอดภัย
- ข้อมูลตัวอย่างนี้เก็บ passphrase / กุญแจใน ENV/โน้ตบุ๊กเพื่อความง่ายในการทดสอบเท่านั้น — ในสภาพแวดล้อมจริง ควรเก็บกุญแจ/ความลับใน Secret Manager หรือ KMS และควบคุมการเข้าถึงอย่างเข้มงวด
- การเข้ารหัสแบบ symmetric และ public-key เก็บผลลัพธ์เป็นชนิด `bytea` ในตาราง ดังนั้นคอลัมน์ที่เข้ารหัสจะไม่สามารถใช้ indexing/search แบบปกติได้
- หากต้องการค้นหา (equality/range) ให้พิจารณาจัดเก็บค่าแฮช (hash) แยกต่างหาก เพื่อใช้เป็นดัชนีในการค้นหา
- โน้ตบุ๊ก public-key จะสร้างกุญแจแบบชั่วคราวเพื่อการสาธิต — ห้ามใช้กุญแจชุดทดลองนี้ในงานจริง หากใช้ public-key จริง ต้องเก็บกุญแจส่วนตัวอย่างปลอดภัยและมีการสำรอง

## คำสั่งที่เป็นประโยชน์
- ดูสถานะคอนเทนเนอร์: `docker ps`
- ดู log ของ Postgres: `docker-compose logs -f postgres`
- หยุดและลบคอนเทนเนอร์: `docker-compose down`

## ใบอนุญาต
โปรเจกต์ตัวอย่างเพื่อการศึกษา — ไม่มีสัญญาอนุญาตอย่างเป็นทางการ

