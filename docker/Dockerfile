FROM postgres:16-alpine

# คัดลอกสคริปต์เริ่มต้นเพื่อเปิดใช้ส่วนขยาย pgcrypto สำหรับการเข้ารหัสคอลัมน์
COPY initdb/ /docker-entrypoint-initdb.d/

