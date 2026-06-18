# **FINAL PROJECT TEKNOLOGI KOMPUTASI AWAN 2026**


**KELOMPOK 4 KELAS B**

| Nama | NRP |
| --- | --- |
| Tiara Putri Prasetya | 5027241013 |
| Clarissa Aydin Rahmazea | 5027241014 |
| Muhammad Fatihul Qolbi | 5027241023 |
| Aslam Ahmad Usman | 5027241074 |
| Putu Yudi Nandanjaya | 5027241080 |
| Mutiara Diva Jaladitha | 5027241083 |
| I Gede Bagus Saka | 5027241088 |

# FINAL PROJECT TEKNOLOGI KOMPUTASI AWAN 2026

## Introduction

Proyek ini merupakan implementasi **Order Processing Service** berbasis cloud untuk platform e-commerce. Layanan ini menangani pembuatan pesanan, pengecekan status, dan riwayat transaksi menggunakan REST API berbasis **Python (Flask)** dengan database **MongoDB**, yang di-deploy di atas infrastruktur cloud Azure.

Arsitektur dirancang untuk mampu menangani lonjakan traffic (flash sale, promo, dsb.) secara andal dan efisien dengan budget maksimal **75 USD/bulan**.

## Spesifikasi VM

VM :

| VM | Hostname | Fungsi | CPU | RAM | SSH | Password |
|----|----------|--------|-----|-----|-----|----------|
| VM1 | fe-lb | Frontend + Nginx Load Balancer | 1 vCPU | 1 GB | ssh vm1@20.244.87.0 | Tka1234567890 |
| VM2 | backend-1 | Flask + Gunicorn | 2 vCPU | 4 GB | ssh azureuser@20.207.194.140 | Tka1234567890 |
| VM3 | backend-2 | Flask + Gunicorn | 2 vCPU | 4 GB | ssh VM3@40.81.233.30 | Tka1234567890 |
| VM4 | mongodb | MongoDB Server | 2 vCPU | 4 GB | ssh VM4@4.240.118.65 | Tka1234567890 |

---

# IMPLEMENTASI

# 1. Cloud Architect & Team Lead
### Putu Yudi Nandanjaya - 5027241080 

## Tugas Utama

Merancang seluruh arsitektur cloud.

## Job Desk

### 1. Menentukan Infrastruktur

```
VM1 = Nginx Load Balancer
VM2 = Backend Flask 1
VM3 = Backend Flask 2
VM4 = MongoDB
```

### 2. Membuat Diagram Topologi Arsitektur Cloud




### 3. Cost Analysis

![Azure VM Pricing]()

| VM | CPU | RAM | Harga |
|----|-----|-----|-------|
| vm1 | 1 CPU | 1 GB | $4,49 |
| vm2 | 2 CPU | 4 GB | $17,96 |
| vm3 | 2 CPU | 4 GB | $17,96 |
| vm4 | 2 CPU | 4 GB | $17,96 |

Total:

```
58,37 USD
```

Sudah sesuai dengan requirement soal, dimana maks untuk budget VM dibawah 75 USD.

**Alasan Pemilihan Konfigurasi:**

- **VM1 (1 vCPU, 1 GB)**: Cukup untuk Nginx yang hanya meneruskan request (tidak memproses logika bisnis). Spec kecil menekan biaya.
- **VM2 & VM3 (2 vCPU, 4 GB)**: Backend Flask + Gunicorn membutuhkan lebih banyak resource untuk menangani concurrent request. Dua instance memungkinkan horizontal scaling.
- **VM4 (2 vCPU, 4 GB)**: MongoDB dipisahkan dari app server untuk performa query yang lebih stabil dan tidak berkompetisi resource dengan backend.
- **Load Balancing Least Connection**: Lebih optimal dari round-robin karena mendistribusikan request ke backend dengan koneksi aktif paling sedikit, ideal saat beban tidak merata.


# 2. Database Engineer
###  I Gede Bagus Saka - 5027241088 

## Tugas Utama

Mengurus MongoDB.

## Job Desk

### 1. Install MongoDB

Ubuntu:

```bash
sudo apt install mongodb
```

### 2. Restore Database

```bash
mongorestore dump/orderdb
```

### 3. Verifikasi Collection

```js
show dbs

use orderdb

show collections
```

### 4. Membuat Index

```js
db.orders.createIndex({order_id: 1})
```

```js
db.orders.createIndex({created_at: -1})
```

Index pada `order_id` dan `created_at` mempercepat query history dan pencarian order secara signifikan.

### 5. Monitoring MongoDB

```bash
mongostat
```

### Screenshot

- MongoDB running

![MongoDB Running]()

- Collection muncul

![Collection Muncul]()

- Index berhasil dibuat

![Index Berhasil Dibuat]()

### Informasi Database

- Private IP: `10.0.0.4`
- Port: `27017`
- Username: `admin`
- Password: `Admin@12345`
- Database: `orderdb`

---

# 3. Backend Engineer
### Muhammad Fatihul Qolbi - 5027241023 

Folder:

```
Resources/BE
```

## Tugas Utama

Deploy Flask API.

## Job Desk

### 1. Install Dependency

```bash
pip install -r requirements.txt
```

### 2. Menjalankan Flask

```bash
python app.py
```

### 3. Setup Gunicorn

```bash
gunicorn -w 4 -b 0.0.0.0:5000 app:app
```

### 4. Endpoint Testing

Test:

```
POST /order
```

```
GET /order/<id>
```

```
GET /orders
```

```
PUT /order/<id>
```

### Screenshot

- Backend sukses

```bash
sudo systemctl status tka-backend --no-pager
```

![Backend Sukses]()

- `sudo ss -tulpn | grep 5000`

![Port 5000 Listening]()

- Testing Health

![Health Check]()

Response:
```json
{
  "database": "connected",
  "status": "ok",
  "timestamp": "2026-06-18T08:13:51.624541+00:00"
}
```

- Test POST ORDER

![POST Order]()

Request:
```bash
ORDER_ID=$(curl -s -X POST http://127.0.0.1:5000/order \
  -H "Content-Type: application/json" \
  -d '{"product":"Laptop Gaming","quantity":2,"price":150000}' | jq -r '.order_id')
```

Response (201 Created):
```json
{
  "_id": "6a33a8cc9209be684ce4W497",
  "order_id": "f754d976-3b33-4a07-afde-770a0354fa22",
  "product": "Laptop Gaming",
  "quantity": 2,
  "price": 150000.0,
  "status": "pending",
  "payment_status": "unpaid",
  "total": 300000.0,
  "created_at": "2026-06-18T08:14:04.496000",
  "updated_at": "2026-06-18T08:14:04.496000"
}
```

- Test GET Detail Order

![GET Detail Order]()

Request:
```bash
curl -s http://127.0.0.1:5000/order/$ORDER_ID | jq
```

Response (200 OK):
```json
{
  "_id": "6a33a8cc9209be684ce4W497",
  "order_id": "f754d976-3b33-4a07-afde-770a0354fa22",
  "product": "Laptop Gaming",
  "quantity": 2,
  "price": 150000.0,
  "status": "pending",
  "payment_status": "unpaid",
  "total": 300000.0
}
```

- Test PUT Update Status

![PUT Update Status]()

Request:
```bash
curl -s -X PUT http://127.0.0.1:5000/order/$ORDER_ID \
  -H "Content-Type: application/json" \
  -d '{"status":"completed"}' | jq
```

Response (200 OK):
```json
{
  "order_id": "f754d976-3b33-4a07-afde-770a0354fa22",
  "status": "completed"
}
```

- Cek ulang order setelah di-update

![Order Setelah Update]()

Response (200 OK):
```json
{
  "_id": "6a33a8cc9209be684ce4W497",
  "order_id": "f754d976-3b33-4a07-afde-770a0354fa22",
  "product": "Laptop Gaming",
  "status": "completed",
  "payment_status": "paid",
  "total": 300000.0,
  "updated_at": "2026-06-18T08:14:58.046000"
}
```

- Test GET semua orders

![GET All Orders 1]()

![GET All Orders 2]()

Request:
```bash
curl -s "http://127.0.0.1:5000/orders?limit=5" | jq
```

Response (200 OK):
```json
{
  "data": [ "..." ],
  "limit": 5,
  "page": 1,
  "total": 10001,
  "total_pages": 2001
}
```

- Test not found

![Not Found]()

Response:
```json
{
  "error": "Order not found"
}
```

---

# 4. Frontend & Web Server Engineer
### Tiara Putri Prasetya - 5027241013

Folder:

```
Resources/FE
```

## Tugas Utama

Deploy frontend.

## Job Desk

### 1. Install Nginx

```bash
sudo apt install nginx
```

### 2. Hosting Frontend

Copy:

```
index.html
styles.css
```

ke

```
/var/www/html
```

### 3. Konfigurasi API URL

Sesuaikan URL backend.

### 4. Integrasi Frontend ↔ Backend

Test:

- Create Order
- View History
- Check Status

### Screenshot

- Frontend berjalan

![Frontend Berjalan]()

- Frontend terhubung API

![Frontend Terhubung API]()

---

# 5. DevOps & Load Balancer Engineer
###  Aslam Ahmad Usman - 5027241074 |

## Tugas Utama

Mengatur scaling dan load balancing.

## Job Desk

### 1. Install Nginx Load Balancer

Konfigurasi:

```nginx
upstream backend_cluster {
    least_conn;
    server 20.207.194.140:5000;  # VM2 backend-1
    server 40.81.233.30:5000;    # VM3 backend-2
}
```

### 2. Reverse Proxy

```nginx
server {
    listen 80;
    server_name 20.244.87.0;

    # Frontend (static files)
    location / {
        root /var/www/html;
        index index.html;
        try_files $uri $uri/ =404;
    }

    # API ngeload balanced ke backend
    location /order {
        proxy_pass http://backend_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_connect_timeout 30s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
    }

    location /orders {
        proxy_pass http://backend_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_connect_timeout 30s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
    }
}
```

### 3. Aktivasi Config

```bash
# Enable config
sudo ln -s /etc/nginx/sites-available/loadbalancer /etc/nginx/sites-enabled/

# Hapus default biar tidak conflict
sudo rm /etc/nginx/sites-enabled/default

# Test config
sudo nginx -t

# Kalau berhasil reload
sudo systemctl reload nginx
sudo systemctl enable nginx
```

### Testing Load Balancer

Pastikan request terbagi ke backend.

Verifikasi dari Nginx access log:

```
TKA-Backend1 gunicorn[15911]: 182.5.243.39 -- [18/Jun/2026:14:59:26] "GET /orders HTTP/1.0" 200
TKA-Backend2 gunicorn[871]:   20.244.87.0 -- [18/Jun/2026:16:27:06] "GET /orders HTTP/1.0" 200
```

### Screenshot

- Status Nginx

![Status Nginx]()

- Config Load Balancer

![Config Load Balancer]()

- Backend cluster

![Backend Cluster]()

- Log Nginx

![Log Nginx]()

- Request muncul di Backend 1 dan Backend 2

![Request Backend 1]()

![Request Backend 2]()

---

# Performance Testing Engineer
### Mutiara Diva Jaladitha - 5027241083

Folder:

```
Resources/Test
```

## Tugas Utama

Locust Testing.

## Job Desk

### 1. Setup Locust

```bash
pip install locust

#atau

sudo apt install python3-locust
```

```bash
cd /mnt/d/ITS/4\ TKA/fp-tka-b04/Resources/Test

locust -f locustfile_order.py --host=http://20.244.87.0
```

buka http://localhost:8089/

![Locust UI]()

### Skenario 1

Maximum RPS. Durasi: 60 detik

**User: 50 | Ramp: 10**

![Skenario 1 - 50 Users Stats]()

![Skenario 1 - 50 Users Chart]()

**User: 100 | Ramp: 10**

![Skenario 1 - 100 Users Stats]()

![Skenario 1 - 100 Users Chart]()

**User: 200 | Ramp: 10**

![Skenario 1 - 200 Users Stats]()

![Skenario 1 - 200 Users Chart]()

![Skenario 1 - Summary]()

**Rata-rata RPS** tertinggi dengan tingkat kegagalan 0%: **83.2 RPS**

| Run | Users | Ramp | RPS | Failure |
|-----|-------|------|-----|---------|
| #1 | 50 | 10 | ~55.5 | 0% |
| #2 | 100 | 10 | ~83.2 | 0% |
| #3 | 200 | 10 | ~191.3 | 45% ❌ |

---

### Skenario 2

**User: 100 | Ramp: 50**

![Skenario 2 - 100 Users]()

**User: 200 | Ramp: 50**

![Skenario 2 - 200 Users]()

**User: 300 | Ramp: 50**

![Skenario 2 - 300 Users]()

**User: 400 | Ramp: 50**

![Skenario 2 - 400 Users]()

| Run | Users | Ramp | Failure |
|-----|-------|------|---------|
| #1 | 100 | 50 | 0% ✅ |
| #2 | 200 | 50 | 0% ✅ |
| #3 | 300 | 50 | 0% ✅ |
| #4 | 400 | 50 | 57% ❌ |

Kesimpulan
Jumlah **concurrent user** tertinggi yang masih dapat dilayani dengan failure 0%: **300**

---

### Skenario 3

**User: 100 | Ramp: 100**

![Skenario 3 - 100 Users]()

**User: 200 | Ramp: 100**

![Skenario 3 - 200 Users]()

| Run | Users | Ramp | Failure |
|-----|-------|------|---------|
| #1 | 100 | 100 | 0% ✅ |
| #2 | 200 | 100 | 98% ❌ |

Kesimpulan
Jumlah **concurrent user** tertinggi yang masih dapat dilayani dengan failure 0%: **100**

---

### Skenario 4

**User: 100 | Ramp: 200**

![Skenario 4 - 100 Users]()

**User: 200 | Ramp: 200**

![Skenario 4 - 200 Users]()

**User: 300 | Ramp: 200**

![Skenario 4 - 300 Users]()

| Run | Users | Ramp | Failure |
|-----|-------|------|---------|
| #1 | 100 | 200 | 0% ✅ |
| #2 | 200 | 200 | 0% ✅ |
| #3 | 300 | 200 | 73% ❌ |

Kesimpulan
Jumlah **concurrent user** tertinggi yang masih dapat dilayani dengan failure 0%: **200**

---

### Skenario 5

**User: 100 | Ramp: 500**

![Skenario 5 - 100 Users]()

**User: 200 | Ramp: 500**

![Skenario 5 - 200 Users]()

| Run | Users | Ramp | Failure |
|-----|-------|------|---------|
| #1 | 100 | 500 | 0% ✅ |
| #2 | 200 | 500 | 10% ❌ |

Kesimpulan
Jumlah **concurrent user** tertinggi yang masih dapat dilayani dengan failure 0%: **100**

### Screenshot

- Statistics

![Statistics]()

- Charts

![Charts]()

- RPS

![RPS]()

- Failure

![Failure]()

### Ringkasan Hasil Load Testing

| Skenario | Konfigurasi | Hasil |
|----------|-------------|-------|
| 1 — Max RPS | Ramp 10, escalating users | **83.2 RPS** (failure 0%) |
| 2 — Peak Concurrency | Ramp 50 | **300 users** (failure 0%) |
| 3 — Peak Concurrency | Ramp 100 | **100 users** (failure 0%) |
| 4 — Peak Concurrency | Ramp 200 | **200 users** (failure 0%) |
| 5 — Peak Concurrency | Ramp 500 | **100 users** (failure 0%) |

> **Catatan Penilaian RPS:** Nilai = (83.2 / 200) × 30 = **12.48 poin**

---

# 7. Monitoring & Documentation Engineer
### Clarissa Aydin Rahmazea - 5027241014

## Tugas Utama

Mengumpulkan seluruh hasil dan membuat laporan.

## Job Desk

### Monitoring Resource

Saat Locust berjalan.

Gunakan:

```bash
htop
```

```bash
vmstat 1
```

atau

```bash
docker stats
```

### Screenshot

Untuk setiap skenario:

- CPU usage & RAM usage — Skenario 1

![Monitor Skenario 1]()

- CPU usage & RAM usage — Skenario 2

![Monitor Skenario 2]()

- CPU usage & RAM usage — Skenario 3

![Monitor Skenario 3]()

- CPU usage & RAM usage — Skenario 4

![Monitor Skenario 4]()

- CPU usage & RAM usage — Skenario 5

![Monitor Skenario 5]()

### Menulis README

Bagian:

```
Introduction
```

```
Architecture
```

```
Implementation
```

```
Endpoint Testing
```

```
Load Testing
```

```
Conclusion
```

---

## Conclusion

Arsitektur cloud yang dirancang berhasil memenuhi seluruh requirement final project:

1. **Budget terkendali** — Total biaya $58,37/bulan, di bawah batas maksimum $75/bulan.
2. **Skalabilitas horizontal** — Dua backend server (VM2 & VM3) dengan Nginx Load Balancer strategi Least Connection terbukti mendistribusikan traffic secara merata, dibuktikan dari log Nginx yang mencatat request masuk ke Backend 1 dan Backend 2.
3. **Performa stabil** — Sistem mampu menangani hingga **300 concurrent users** dengan 0% failure (Skenario 2, Ramp 50), dan mencapai maksimum **83.2 RPS** tanpa error (Skenario 1).
4. **Database terpisah** — MongoDB di VM terpisah (VM4) meningkatkan performa query dan mencegah resource contention dengan backend.
5. **Index MongoDB** — Index pada `order_id` dan `created_at` mengoptimalkan kecepatan query `GET /orders` secara signifikan.

Bottleneck utama yang ditemukan adalah pada ramp rate tinggi (100+) — sistem tidak sempat warm up sehingga terjadi spike failure sementara. Rekomendasi ke depan: tambah jumlah Gunicorn worker dan implementasikan connection pooling MongoDB yang lebih agresif.
