# **FINAL PROJECT TEKNOLOGI KOMPUTASI AWAN 2026**
## **KELOMPOK 4 KELAS B**

| Nama | NRP |
| --- | --- |
| Tiara Putri Prasetya | 5027241013 |
| Clarissa Aydin Rahmazea | 5027241014 |
| Muhammad Fatihul Qolbi | 5027241023 |
| Aslam Ahmad Usman | 5027241074 |
| Putu Yudi Nandanjaya | 5027241080 |
| Mutiara Diva Jaladitha | 5027241083 |
| I Gede Bagus Saka | 5027241088 |

## Introduction

Proyek ini merupakan implementasi **Order Processing Service** berbasis cloud untuk platform e-commerce. Layanan ini menangani pembuatan pesanan, pengecekan status, dan riwayat transaksi menggunakan REST API berbasis **Python (Flask)** dengan database **MongoDB**, yang di-deploy di atas infrastruktur cloud Azure.

Arsitektur dirancang untuk mampu menangani lonjakan traffic (flash sale, promo, dsb.) secara andal dan efisien dengan budget maksimal **75 USD/bulan**.

## Spesifikasi VM

| VM | Hostname | Fungsi | CPU | RAM | Harga/Bulan |
|----|----------|--------|-----|-----|-------------|
| VM1 | fe-lb | Frontend + Nginx Load Balancer | 1 vCPU | 1 GB | $4.49 |
| VM2 | backend-1 | Flask + Gunicorn | 2 vCPU | 4 GB | $17.96 |
| VM3 | backend-2 | Flask + Gunicorn | 2 vCPU | 4 GB | $17.96 |
| VM4 | mongodb | MongoDB Server | 2 vCPU | 4 GB | $17.96 |
| **Total** | | | | | **$58.37** |

---

# IMPLEMENTASI

# 1. Cloud Architect & Team Lead
### Putu Yudi Nandanjaya - 5027241080 

## Tugas Utama

Merancang seluruh arsitektur cloud yang akan digunakan tim, mencakup pemilihan jumlah VM, spesifikasi masing-masing VM, strategi load balancing, serta estimasi biaya keseluruhan agar tidak melebihi budget yang ditentukan.

## Job Desk

### 1. Menentukan Infrastruktur

Infrastruktur dirancang dengan memisahkan setiap layer ke VM yang berbeda agar tidak ada resource contention antar komponen. Pemisahan ini juga memudahkan scaling secara independen jika salah satu layer mengalami bottleneck.

```
VM1 = Nginx Load Balancer
VM2 = Backend Flask 1
VM3 = Backend Flask 2
VM4 = MongoDB
```

### 2. Membuat Diagram Topologi Arsitektur Cloud

Diagram arsitektur dibuat menggunakan draw.io untuk menggambarkan alur request dari user hingga ke database, termasuk komponen load balancer dan dua backend server.

<img width="575" height="790" alt="topologi1" src="https://github.com/user-attachments/assets/ed0de35f-6818-46a3-be20-372cea7072a5" />

Diagram memuat komponen:

- **Client Layer**: User mengakses sistem melalui web browser
- **Presentation Layer**: Frontend server (VM1) melayani static files
- **Load Balancing Layer**: Nginx di VM1 mendistribusikan request API ke backend
- **Application Layer**: Dua backend Flask (VM2 & VM3) memproses request secara paralel
- **Data Layer**: MongoDB di VM4 menyimpan seluruh data pesanan

### 3. Cost Analysis

Pemilihan spesifikasi VM disesuaikan dengan kebutuhan beban kerja masing-masing layer, sekaligus menjaga total biaya di bawah batas maksimum $75/bulan.

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

Total biaya **$58,37/bulan**, sudah sesuai dengan requirement soal dimana budget maksimal VM adalah $75/bulan.

**Alasan Pemilihan Konfigurasi:**

- **VM1 (1 vCPU, 1 GB)**: Cukup untuk Nginx yang hanya meneruskan request (tidak memproses logika bisnis). Spec kecil menekan biaya.
- **VM2 & VM3 (2 vCPU, 4 GB)**: Backend Flask + Gunicorn membutuhkan lebih banyak resource untuk menangani concurrent request. Dua instance memungkinkan horizontal scaling.
- **VM4 (2 vCPU, 4 GB)**: MongoDB dipisahkan dari app server untuk performa query yang lebih stabil dan tidak berkompetisi resource dengan backend.
- **Load Balancing Least Connection**: Lebih optimal dari round-robin karena mendistribusikan request ke backend dengan koneksi aktif paling sedikit, ideal saat beban tidak merata.


# 2. Database Engineer
###  I Gede Bagus Saka - 5027241088 

## Tugas Utama

Menyiapkan dan mengelola database MongoDB di VM4, mulai dari instalasi, konfigurasi koneksi, restore data awal, pembuatan index untuk optimasi performa query, hingga monitoring kondisi database saat load testing berlangsung.

## Job Desk

### 1. Install MongoDB

MongoDB di-install di VM4 yang terpisah dari backend server. Pemisahan ini bertujuan agar resource database (CPU, RAM, I/O) tidak berkompetisi dengan proses aplikasi Flask.

```bash
sudo apt install mongodb
```
Setelah instalasi, MongoDB dijalankan sebagai service agar otomatis aktif saat VM restart:

```bash
sudo systemctl enable mongod
sudo systemctl start mongod
```

### 2. Restore Database

Data awal di-restore dari dump yang sudah disiapkan agar collection `orders` memiliki data untuk keperluan pengujian endpoint dan load testing.

```bash
mongorestore dump/orderdb
```

### 3. Verifikasi Collection

Setelah restore, dilakukan verifikasi untuk memastikan database dan collection berhasil terbentuk dengan benar.

```js
show dbs

use orderdb

show collections
```

### 4. Membuat Index

Index dibuat pada field yang paling sering digunakan dalam query agar performa GET /orders dan pencarian order by ID lebih cepat, terutama saat collection sudah berisi ribuan dokumen.

```js
db.orders.createIndex({order_id: 1})
```

```js
db.orders.createIndex({created_at: -1})
```

Index pada `order_id` dan `created_at` mempercepat query history dan pencarian order secara signifikan.

### 5. Monitoring MongoDB

Selama load testing berlangsung, kondisi MongoDB dipantau menggunakan `mongostat` untuk memastikan tidak ada bottleneck di sisi database.

```bash
mongostat
```

### Screenshot

- MongoDB running

<img width="2880" height="528" alt="Screenshot 2026-06-17 225948" src="https://github.com/user-attachments/assets/7b6be576-2fcd-40db-a502-96213968bcfb" />

- Collection muncul

<img width="2876" height="624" alt="2mongodb" src="https://github.com/user-attachments/assets/b4b25fcb-6bb2-4266-a0ad-7b65b2c3d8c6" />

- Index berhasil dibuat

<img width="2880" height="1196" alt="3mongo" src="https://github.com/user-attachments/assets/ae8d25e3-78f6-4fcf-bbbb-c009a65418da" />

---

# 3. Backend Engineer
### Muhammad Fatihul Qolbi - 5027241023 

Folder:

```
Resources/BE
```

## Tugas Utama

Men-deploy aplikasi Flask REST API di VM2 dan VM3, mengkonfigurasi Gunicorn sebagai WSGI server produksi, serta memastikan seluruh endpoint berjalan dengan benar dan dapat terhubung ke MongoDB di VM4.

## Job Desk

### 1. Install Dependency

Seluruh dependency Python yang dibutuhkan aplikasi Flask di-install dari file `requirements.txt` yang sudah disediakan di repository.

```bash
pip install -r requirements.txt
```

### 2. Menjalankan Flask

Untuk keperluan testing awal, Flask dijalankan langsung dengan development server:

```bash
python app.py
```

### 3. Setup Gunicorn

Untuk environment produksi, Flask dijalankan menggunakan Gunicorn dengan 4 worker process. Gunicorn dipilih karena mampu menangani multiple concurrent request secara efisien dibanding Flask development server yang single-threaded.

```bash
gunicorn -w 4 -b 0.0.0.0:5000 app:app
```

### 4. Endpoint Testing

Seluruh endpoint diuji langsung dari VM backend menggunakan `curl` sebelum diintegrasikan dengan Load Balancer.

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

<img width="2145" height="478" alt="Screenshot 2026-06-18 151652" src="https://github.com/user-attachments/assets/515332ea-0502-4eda-835b-4e26a3d3d580" />

- `sudo ss -tulpn | grep 5000`

<img width="1959" height="45" alt="be2" src="https://github.com/user-attachments/assets/6dd174c5-ed08-4a73-a866-c59629627313" />


- Testing Health

<img width="872" height="120" alt="be3" src="https://github.com/user-attachments/assets/70fc8bbf-2dad-4c52-abec-166e913482e8" />

- Test POST ORDER

<img width="979" height="121" alt="be4" src="https://github.com/user-attachments/assets/da60c73f-bf59-470c-906e-f9e52cac09a1" />

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

<img width="955" height="272" alt="get detail" src="https://github.com/user-attachments/assets/1b10612b-dc3e-43c1-982a-64142fd37621" />


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

<img width="963" height="136" alt="put update status" src="https://github.com/user-attachments/assets/bee10aa5-2c7b-4250-b902-7291d1a3e6e2" />

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

<img width="991" height="273" alt="cek ulang order" src="https://github.com/user-attachments/assets/4c9bb631-5001-4f0a-a4d0-f5fec98214fb" />


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

<img width="1005" height="1307" alt="get all order" src="https://github.com/user-attachments/assets/d997922c-5df1-41ca-9772-4a52bc9f4ea9" />

<img width="973" height="992" alt="get all order2" src="https://github.com/user-attachments/assets/02895eed-30c8-40e3-9d71-73667718c61b" />

- Test not found

<img width="1024" height="82" alt="not found" src="https://github.com/user-attachments/assets/0b179bb4-0568-4232-a41b-5298e334ebf2" />

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

Men-deploy tampilan antarmuka frontend yang disediakan di repository ke VM1, mengkonfigurasi Nginx agar dapat menyajikan static file, serta memastikan frontend dapat terhubung dan berkomunikasi dengan backend melalui Load Balancer.

## Job Desk

### 1. Install Nginx

Nginx di-install di VM1 untuk berfungsi sebagai web server yang menyajikan halaman frontend statis kepada user.


```bash
sudo apt install nginx
```

### 2. Hosting Frontend

File frontend yang disediakan di repository (`index.html` dan `styles.css`) di-copy ke direktori default Nginx agar dapat diakses melalui browser.

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

URL backend di dalam file frontend disesuaikan agar mengarah ke Load Balancer, bukan langsung ke salah satu backend. Ini memastikan seluruh request API dari frontend ikut melewati proses load balancing.

### 4. Integrasi Frontend ↔ Backend

Setelah deployment, dilakukan pengujian fungsionalitas antarmuka secara end-to-end:

- **Create Order** — Form pembuatan pesanan berhasil mengirim request ke backend dan menampilkan response
- **View History** — Halaman riwayat pesanan berhasil menampilkan data dari endpoint GET /orders
- **Check Status** — Pencarian status pesanan berdasarkan order_id berjalan dengan benar

### Screenshot

- Frontend berjalan

![Frontend Berjalan]()

- Frontend terhubung API

![Frontend Terhubung API]()

---

# 5. DevOps & Load Balancer Engineer
###  Aslam Ahmad Usman - 5027241074 |

## Tugas Utama

Mengonfigurasi Nginx di VM1 sebagai Load Balancer yang mendistribusikan request API ke dua backend server (VM2 dan VM3), memastikan request terbagi secara merata, serta melakukan verifikasi distribusi traffic melalui log Nginx.

## Job Desk

### 1. Install Nginx Load Balancer

Nginx dikonfigurasi menggunakan strategi **Least Connection** (`least_conn`) yang akan mengarahkan setiap request baru ke backend dengan jumlah koneksi aktif paling sedikit. Strategi ini lebih optimal dibanding round-robin sederhana karena mempertimbangkan beban aktual masing-masing backend.

Konfigurasi:

```nginx
upstream backend_cluster {
    least_conn;
    server 20.207.194.140:5000;  # VM2 backend-1
    server 40.81.233.30:5000;    # VM3 backend-2
}
```

### 2. Reverse Proxy

Nginx di VM1 dikonfigurasi untuk melayani dua fungsi sekaligus: menyajikan frontend statis dan meneruskan request API ke backend cluster.

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

Config Load Balancer diaktifkan dengan membuat symlink ke `sites-enabled`, kemudian default config Nginx dihapus untuk menghindari konflik.

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

<img width="1639" height="538" alt="status nginx" src="https://github.com/user-attachments/assets/ecddbe28-720a-4148-b69d-6318a4732547" />

- Config Load Balancer

<img width="818" height="851" alt="config load balancer" src="https://github.com/user-attachments/assets/2144a0fe-969e-4bfd-a341-1d04ce329755" />

- Log Nginx

<img width="1698" height="171" alt="Screenshot 2026-06-18 233007" src="https://github.com/user-attachments/assets/1f534940-3ff7-4f17-9714-aef61514e57e" />


- Request muncul di Backend 1 dan Backend 2

<img width="1693" height="54" alt="Screenshot 2026-06-18 234116" src="https://github.com/user-attachments/assets/a6b4cc5d-baf2-4f78-a729-124e7301f5e0" />

<img width="1689" height="31" alt="Screenshot 2026-06-18 233316" src="https://github.com/user-attachments/assets/da0e6921-3d1d-4e68-bfc5-99d0105d7991" />

---

# Performance Testing Engineer
### Mutiara Diva Jaladitha - 5027241083

Folder:

```
Resources/Test
```

## Tugas Utama

Melakukan load testing menggunakan Locust untuk mengukur performa sistem secara keseluruhan, mencakup pencarian RPS (Request Per Second) tertinggi dengan failure 0% dan jumlah concurrent user maksimum yang dapat dilayani tanpa kegagalan pada berbagai konfigurasi ramp rate.

## Job Desk

### 1. Setup Locust

Locust di-install di mesin terpisah (bukan di VM cloud) agar proses load testing tidak mengkonsumsi resource VM yang sedang diuji.

```bash
pip install locust

#atau

sudo apt install python3-locust
```
Locust dijalankan dengan locustfile yang sudah disiapkan, mengarah ke IP Load Balancer sebagai target:

```bash
cd /mnt/d/ITS/4\ TKA/fp-tka-b04/Resources/Test

locust -f locustfile.py --host=http://20.244.87.0
```

Buka dashboard Locust di browser: http://localhost:8089/

<img width="1920" height="663" alt="image" src="https://github.com/user-attachments/assets/e061776a-f18a-46d1-935c-d29f8638e4f5" />


### Skenario 1

Tujuan skenario ini adalah mencari nilai **RPS (Request Per Second) tertinggi** yang dapat dicapai sistem dengan **tingkat kegagalan 0%**. Pengujian dilakukan dengan menaikkan jumlah user secara bertahap menggunakan ramp rate 10 user/detik.

**User: 50 | Ramp: 10**

![Skenario 1 - 50 Users Stats]()

![Skenario 1 - 50 Users Chart]()

**User: 100 | Ramp: 10**

![Skenario 1 - 100 Users Stats]()

![Skenario 1 - 100 Users Chart]()

**User: 200 | Ramp: 10**


Pada 200 user, sistem mulai mengalami kegagalan sehingga angka ini tidak dipakai sebagai hasil akhir.


![Skenario 1 - 200 Users Stats]()

![Skenario 1 - 200 Users Chart]()

![Skenario 1 - Summary]()

**Rata-rata RPS** tertinggi dengan tingkat kegagalan 0%: **83.2 RPS**

| Run | Users | Ramp | RPS | Failure |
|-----|-------|------|-----|---------|
| #1 | 50 | 10 | ~55.5 | 0% |
| #2 | 100 | 10 | ~83.2 | 0% |
| #3 | 200 | 10 | ~191.3 | 45% |

**Rata-rata RPS tertinggi dengan tingkat kegagalan 0%: 83.2 RPS**

---

### Skenario 2

Tujuan skenario ini adalah mencari jumlah **concurrent user tertinggi** yang masih dapat dilayani sistem dengan **failure 0%**, menggunakan ramp rate 50 user/detik.

**User: 100 | Ramp: 50**

<img width="579" height="396" alt="s2 - 1" src="https://github.com/user-attachments/assets/d2f35540-374c-444e-9b21-2f6f6dfe3881" />


**User: 200 | Ramp: 50**

<img width="571" height="392" alt="s2 - 2" src="https://github.com/user-attachments/assets/8cfbe8c6-8c81-4678-8ac9-fc1043110ab2" />


**User: 300 | Ramp: 50**

<img width="571" height="389" alt="s2 - 300" src="https://github.com/user-attachments/assets/b16274c0-6c35-41f9-bb90-a68578179c43" />


**User: 400 | Ramp: 50**

Pada 400 user, sistem mulai mengalami kegagalan sebesar 57%, menandakan batas kapasitas terlampaui.

<img width="573" height="392" alt="s2 - 400" src="https://github.com/user-attachments/assets/09695af5-b9d1-4653-a928-47e9c3949c2d" />

| Run | Users | Ramp | Failure |
|-----|-------|------|---------|
| #1 | 100 | 50 | 0% |
| #2 | 200 | 50 | 0% |
| #3 | 300 | 50 | 0% |
| #4 | 400 | 50 | 57% |

Kesimpulan
Jumlah **concurrent user** tertinggi yang masih dapat dilayani dengan failure 0%: **300 users**

---

### Skenario 3

Pengujian diulang dengan ramp rate lebih agresif (100 user/detik) untuk melihat apakah sistem tetap stabil saat jumlah user naik lebih cepat.

**User: 100 | Ramp: 100**

<img width="572" height="392" alt="s3 - 100" src="https://github.com/user-attachments/assets/d98f3639-bef4-4dfd-bede-54b983e07c1e" />


**User: 200 | Ramp: 100**

Pada ramp 100, sistem sudah gagal di 200 user karena koneksi datang terlalu cepat sebelum backend sempat warm up.

<img width="572" height="391" alt="s3 - 200" src="https://github.com/user-attachments/assets/10866145-4959-4958-960d-1b112e187af2" />


| Run | Users | Ramp | Failure |
|-----|-------|------|---------|
| #1 | 100 | 100 | 0% |
| #2 | 200 | 100 | 98% |

Kesimpulan
Jumlah **concurrent user** tertinggi yang masih dapat dilayani dengan failure 0%: **100 users**

---

### Skenario 4

Ramp rate dinaikkan ke 200 user/detik untuk mensimulasikan lonjakan traffic mendadak seperti saat flash sale.

**User: 100 | Ramp: 200**

<img width="576" height="394" alt="s4 - 100" src="https://github.com/user-attachments/assets/58c69f82-2b28-4b71-a2a4-100a55a16e3d" />


**User: 200 | Ramp: 200**

<img width="574" height="391" alt="s4 - 200" src="https://github.com/user-attachments/assets/339d0886-16fe-4b54-8fee-68beed3d3dd8" />


**User: 300 | Ramp: 200**

Pada 300 user dengan ramp 200, sistem tidak mampu menampung lonjakan dan mengalami 73% failure.

<img width="574" height="387" alt="s4 - 300" src="https://github.com/user-attachments/assets/c00dceb6-ea15-44c0-b2b7-ff942e512acf" />


| Run | Users | Ramp | Failure |
|-----|-------|------|---------|
| #1 | 100 | 200 | 0% |
| #2 | 200 | 200 | 0% |
| #3 | 300 | 200 | 73% |

Kesimpulan
Jumlah **concurrent user** tertinggi yang masih dapat dilayani dengan failure 0%: **200 users**

---

### Skenario 5

Skenario paling ekstrem — seluruh user masuk hampir serentak (ramp 500 user/detik) untuk mengukur ketahanan sistem terhadap spike traffic yang sangat tiba-tiba.

**User: 100 | Ramp: 500**

<img width="572" height="390" alt="s5 - 100" src="https://github.com/user-attachments/assets/50d25339-e3ba-4e3a-8518-5f4cd44738d7" />


**User: 200 | Ramp: 500**

Pada ramp 500, bahkan 200 user sudah menyebabkan kegagalan karena sistem tidak punya waktu untuk mendistribusikan koneksi secara merata.

<img width="575" height="386" alt="s5 - 200" src="https://github.com/user-attachments/assets/2ea4e9b4-2272-4145-a015-a9ab357dafad" />


| Run | Users | Ramp | Failure |
|-----|-------|------|---------|
| #1 | 100 | 500 | 0% |
| #2 | 200 | 500 | 10% |

Kesimpulan
Jumlah **concurrent user** tertinggi yang masih dapat dilayani dengan failure 0%: **100 users**

### Ringkasan Hasil Load Testing

| Skenario | Konfigurasi | Hasil |
|----------|-------------|-------|
| 1 (Max RPS) | Ramp 10, escalating users | **83.2 RPS** (failure 0%) |
| 2 (Peak Concurrency) | Ramp 50 | **300 users** (failure 0%) |
| 3 (Peak Concurrency) | Ramp 100 | **100 users** (failure 0%) |
| 4 (Peak Concurrency) | Ramp 200 | **200 users** (failure 0%) |
| 5 (Peak Concurrency) | Ramp 500 | **100 users** (failure 0%) |

Dari hasil di atas terlihat bahwa **ramp rate sangat mempengaruhi kapasitas sistem**. Semakin agresif ramp rate, semakin rendah jumlah user yang dapat dilayani tanpa kegagalan. Ini menunjukkan bahwa sistem cukup stabil saat traffic naik perlahan, namun perlu optimasi tambahan (seperti connection pooling dan pre-warming) untuk menghadapi spike traffic mendadak.

---

# 7. Monitoring & Documentation Engineer
### Clarissa Aydin Rahmazea - 5027241014

## Tugas Utama

Memantau penggunaan resource (CPU dan RAM) di seluruh VM selama load testing berlangsung untuk mengidentifikasi bottleneck, serta mengumpulkan seluruh hasil pengerjaan anggota tim dan menyusunnya menjadi laporan README yang lengkap dan terstruktur.

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

1. **Budget terkendali**: Total biaya $58,37/bulan, di bawah batas maksimum $75/bulan.
2. **Skalabilitas horizontal**: Dua backend server (VM2 & VM3) dengan Nginx Load Balancer strategi Least Connection terbukti mendistribusikan traffic secara merata, dibuktikan dari log Nginx yang mencatat request masuk ke Backend 1 dan Backend 2.
3. **Performa stabil**: Sistem mampu menangani hingga **300 concurrent users** dengan 0% failure (Skenario 2, Ramp 50), dan mencapai maksimum **83.2 RPS** tanpa error (Skenario 1).
4. **Database terpisah**: MongoDB di VM terpisah (VM4) meningkatkan performa query dan mencegah resource contention dengan backend.
5. **Index MongoDB**: Index pada `order_id` dan `created_at` mengoptimalkan kecepatan query `GET /orders` secara signifikan.

**Analisis bottleneck:** Dari hasil load testing, terlihat bahwa ramp rate sangat mempengaruhi stabilitas sistem. Pada ramp rate tinggi (100+), sistem mengalami spike failure karena backend tidak sempat mendistribusikan koneksi sebelum gelombang request berikutnya tiba. Ini mengindikasikan bahwa bottleneck utama ada di kapasitas connection handling Gunicorn, bukan di MongoDB.
