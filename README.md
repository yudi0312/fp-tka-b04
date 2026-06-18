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

## Kelompok B04

| Anggota | Role | Nama |
|---------|------|------|
| 1 | Cloud Architect & Team Lead | Yudi |
| 2 | Database Engineer | Kaka |
| 3 | Backend Engineer | Obi |
| 4 | Frontend & Web Server Engineer | Tiara |
| 5 | DevOps & Load Balancer Engineer | Aslam |
| 6 | Performance Testing Engineer | Diva |
| 7 | Monitoring & Documentation Engineer | Clara |

---

## Introduction

Proyek ini merupakan implementasi **Order Processing Service** berbasis cloud untuk platform e-commerce. Layanan ini menangani pembuatan pesanan, pengecekan status, dan riwayat transaksi menggunakan REST API berbasis **Python (Flask)** dengan database **MongoDB**, yang di-deploy di atas infrastruktur cloud Azure.

Arsitektur dirancang untuk mampu menangani lonjakan traffic (flash sale, promo, dsb.) secara andal dan efisien dengan budget maksimal **75 USD/bulan**.

---

## Architecture

### Topologi Arsitektur Cloud

```
CLIENT LAYER
     │
     ▼ HTTPS (80/443)
┌─────────────────┐
│  FRONTEND SERVER │  ← VM1 (fe-lb) | 1 vCPU | 1 GB
│  Nginx + Static  │
└────────┬────────┘
         │ REST API HTTP /api/*
         ▼
┌─────────────────┐         ┌──────────────────┐
│  LOAD BALANCER  │         │  LOAD TESTING    │
│  Nginx          │◄────────│  HOST (Locust)   │
│  VM1 (fe-lb)    │  HTTP   │  Host Terpisah   │
│  1 vCPU | 1 GB  │ Requests└──────────────────┘
└────────┬────────┘
    HTTP :5000 Round Robin (Least Connection)
    ┌────┴────┐
    ▼         ▼
┌────────┐ ┌────────┐
│Backend1│ │Backend2│  APPLICATION LAYER
│ VM2    │ │ VM3    │
│2vCPU   │ │2vCPU   │
│4 GB    │ │4 GB    │
└────┬───┘ └───┬────┘
     └────┬────┘
          │ MongoDB Protocol :27017
          ▼
    ┌───────────┐
    │  MONGODB  │  DATA LAYER
    │  SERVER   │
    │   VM4     │
    │ 2 vCPU    │
    │   4 GB    │
    └───────────┘
```

### Spesifikasi VM

| VM | Hostname | Fungsi | CPU | RAM | Harga/Bulan |
|----|----------|--------|-----|-----|-------------|
| VM1 | fe-lb | Frontend + Nginx Load Balancer | 1 vCPU | 1 GB | $4.49 |
| VM2 | backend-1 | Flask + Gunicorn | 2 vCPU | 4 GB | $17.96 |
| VM3 | backend-2 | Flask + Gunicorn | 2 vCPU | 4 GB | $17.96 |
| VM4 | mongodb | MongoDB Server | 2 vCPU | 4 GB | $17.96 |
| **Total** | | | | | **$58.37** |

> ✅ Total biaya **$58.37/bulan** — di bawah batas maksimum **$75/bulan**.

### Alasan Pemilihan Konfigurasi

- **VM1 (1 vCPU, 1 GB)** — Cukup untuk Nginx yang hanya meneruskan request (tidak memproses logika bisnis). Spec kecil menekan biaya.
- **VM2 & VM3 (2 vCPU, 4 GB)** — Backend Flask + Gunicorn membutuhkan lebih banyak resource untuk menangani concurrent request. Dua instance memungkinkan horizontal scaling.
- **VM4 (2 vCPU, 4 GB)** — MongoDB dipisahkan dari app server untuk performa query yang lebih stabil dan tidak berkompetisi resource dengan backend.
- **Load Balancing Least Connection** — Lebih optimal dari round-robin karena mendistribusikan request ke backend dengan koneksi aktif paling sedikit, ideal saat beban tidak merata.

---

## Implementation

### 1. Database (MongoDB — VM4)

MongoDB di-install di VM4 dengan konfigurasi terpisah dari backend.

```bash
sudo apt install mongodb
```

**Konfigurasi Database:**
- Private IP: `10.0.0.4`
- Port: `27017`
- Username: `admin`
- Database: `orderdb`

**Membuat Index untuk optimasi query:**

```js
db.orders.createIndex({ order_id: 1 })
db.orders.createIndex({ created_at: -1 })
```

Index pada `order_id` dan `created_at` mempercepat query history dan pencarian order secara signifikan.

**Verifikasi:**

```bash
sudo systemctl status mongod
mongosh -u admin -p "Admin@12345" --authenticationDatabase admin
use orderdb
show collections
db.orders.getIndexes()
```

Screenshot konfirmasi: MongoDB running, collection `orders` muncul, index berhasil dibuat.

---

### 2. Backend (Flask + Gunicorn — VM2 & VM3)

Flask API di-deploy dengan Gunicorn sebagai WSGI server untuk performa produksi.

```bash
pip install -r requirements.txt
gunicorn -w 4 -b 0.0.0.0:5000 app:app
```

**Systemd service** dikonfigurasi agar backend berjalan otomatis saat reboot:

```bash
sudo systemctl status tka-backend --no-pager
sudo ss -tulpn | grep 5000
```

**Health check:**

```bash
curl -s http://127.0.0.1:5000/health | jq
# Output: { "database": "connected", "status": "ok", "timestamp": "..." }
```

---

### 3. Frontend & Web Server (Nginx — VM1)

Frontend statis di-hosting menggunakan Nginx di VM1.

```bash
sudo apt install nginx
# Copy file frontend
cp index.html styles.css /var/www/html/
```

Frontend dikonfigurasi untuk terhubung ke Load Balancer sebagai API endpoint.

---

### 4. Load Balancer (Nginx — VM1)

Nginx dikonfigurasi sebagai reverse proxy dengan strategi **Least Connection**:

```nginx
upstream backend_cluster {
    least_conn;
    server 20.207.194.140:5000;  # VM2 backend-1
    server 40.81.233.30:5000;    # VM3 backend-2
}

server {
    listen 80;
    server_name 20.244.87.0;

    # Frontend (static files)
    location / {
        root /var/www/html;
        index index.html;
        try_files $uri $uri/ =404;
    }

    # API load balanced ke backend
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

```bash
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl enable nginx
```

---

## Endpoint Testing

Seluruh endpoint diuji melalui VM backend langsung maupun melalui Load Balancer.

### 1. POST /order — Membuat Pesanan Baru

```bash
ORDER_ID=$(curl -s -X POST http://localhost/order \
  -H "Content-Type: application/json" \
  -d '{"product":"Laptop Gaming","quantity":2,"price":150000}' | jq -r '.order_id')
```

**Response (201 Created):**
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

---

### 2. GET /order/\<order_id\> — Cek Status Pesanan

```bash
curl -s http://localhost/order/$ORDER_ID | jq
```

**Response (200 OK):**
```json
{
  "_id": "6a33a8cc9209be684ce4W497",
  "order_id": "f754d976-3b33-4a07-afde-770a0354fa22",
  "product": "Laptop Gaming",
  "status": "pending",
  "payment_status": "unpaid",
  "total": 300000.0
}
```

---

### 3. PUT /order/\<order_id\> — Update Status Pesanan

```bash
curl -s -X PUT http://localhost/order/$ORDER_ID \
  -H "Content-Type: application/json" \
  -d '{"status":"completed"}' | jq
```

**Response (200 OK):**
```json
{
  "order_id": "f754d976-3b33-4a07-afde-770a0354fa22",
  "status": "completed"
}
```

---

### 4. GET /orders — Riwayat Semua Pesanan

```bash
curl -s "http://localhost/orders?limit=5" | jq
```

**Response (200 OK):**
```json
{
  "data": [ ... ],
  "limit": 5,
  "page": 1,
  "total": 10001,
  "total_pages": 2001
}
```

---

### 5. GET /order/\<id-tidak-ada\> — Not Found

```bash
curl -s http://localhost/order/order-tidak-ada | jq
# Output: { "error": "Order not found" }
```

---

### Verifikasi Load Balancer

Request terbagi ke Backend 1 dan Backend 2 (terlihat dari Nginx access log):

```
TKA-Backend1 gunicorn[15911]: 182.5.243.39 -- [18/Jun/2026:14:59:26] "GET /orders HTTP/1.0" 200
TKA-Backend2 gunicorn[871]:   20.244.87.0 -- [18/Jun/2026:16:27:06] "GET /orders HTTP/1.0" 200
```

---

## Load Testing

Load testing dilakukan menggunakan **Locust** dengan target host `http://20.244.87.0`.

```bash
locust -f locustfile_order.py --host=http://20.244.87.0
# Buka: http://localhost:8089/
```

### Skenario 1 — Maximum RPS (Ramp 10)

> Tujuan: Mencari RPS tertinggi dengan failure 0%

| Run | Users | Ramp | RPS | Failure |
|-----|-------|------|-----|---------|
| #1 | 50 | 10 | ~55.5 | 0% |
| #2 | 100 | 10 | ~83.2 | 0% |
| #3 | 200 | 10 | ~191.3 | 45% ❌ |

**Kesimpulan:** Rata-rata RPS tertinggi dengan failure 0% = **83.2 RPS**

---

### Skenario 2 — Peak Concurrency (Ramp 50)

> Tujuan: Mencari jumlah user concurrent tertinggi dengan failure 0%

| Run | Users | Ramp | Failure |
|-----|-------|------|---------|
| #1 | 100 | 50 | 0% ✅ |
| #2 | 200 | 50 | 0% ✅ |
| #3 | 300 | 50 | 0% ✅ |
| #4 | 400 | 50 | 57% ❌ |

**Kesimpulan:** Concurrent user tertinggi dengan failure 0% = **300 users**

---

### Skenario 3 — Peak Concurrency (Ramp 100)

| Run | Users | Ramp | Failure |
|-----|-------|------|---------|
| #1 | 100 | 100 | 0% ✅ |
| #2 | 200 | 100 | 98% ❌ |

**Kesimpulan:** Concurrent user tertinggi dengan failure 0% = **100 users**

---

### Skenario 4 — Peak Concurrency (Ramp 200)

| Run | Users | Ramp | Failure |
|-----|-------|------|---------|
| #1 | 100 | 200 | 0% ✅ |
| #2 | 200 | 200 | 0% ✅ |
| #3 | 300 | 200 | 73% ❌ |

**Kesimpulan:** Concurrent user tertinggi dengan failure 0% = **200 users**

---

### Skenario 5 — Peak Concurrency (Ramp 500)

| Run | Users | Ramp | Failure |
|-----|-------|------|---------|
| #1 | 100 | 500 | 0% ✅ |
| #2 | 200 | 500 | 10% ❌ |

**Kesimpulan:** Concurrent user tertinggi dengan failure 0% = **100 users**

---

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

## Conclusion

Arsitektur cloud yang dirancang berhasil memenuhi seluruh requirement final project:

1. **Budget terkendali** — Total biaya $58.37/bulan, di bawah batas $75/bulan.
2. **Skalabilitas horizontal** — Dua backend server (VM2 & VM3) dengan Nginx Load Balancer (Least Connection) terbukti mendistribusikan traffic secara merata.
3. **Performa stabil** — Sistem mampu menangani hingga **300 concurrent users** dengan 0% failure (Skenario 2, Ramp 50), dan mencapai maksimum **83.2 RPS** tanpa error.
4. **Database terpisah** — MongoDB di VM terpisah (VM4) meningkatkan performa query dan mencegah resource contention dengan backend.
5. **Index MongoDB** — Index pada `order_id` dan `created_at` mengoptimalkan query GET /orders secara signifikan.

Bottleneck utama yang ditemukan adalah pada ramp rate tinggi (100+) — sistem tidak sempat warm up sehingga terjadi spike failure sementara. Untuk production, rekomendasi ke depan adalah menambah jumlah Gunicorn worker dan mengimplementasikan connection pooling MongoDB yang lebih agresif.

