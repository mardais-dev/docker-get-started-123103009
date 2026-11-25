# Docker Get Started - Python (FastAPI)

Repositori ini berisi implementasi panduan **Python language-specific guide** dari dokumentasi resmi Docker untuk membungkus (_containerize_) dan menjalankan aplikasi REST API berbasis Python menggunakan **FastAPI**, **Docker Hardened Images (DHI)**, dan **Docker Compose**.

- **Guide Referensi:** [Python language-specific guide | Docker Docs](https://web.archive.org/web/20261024110830/https://docs.docker.com/guides/python/)
- **Repository GitHub:** `https://github.com/mardais-dev/docker-get-started-123103009`

---

## Struktur Direktori Proyek

```text
docker-get-started-123103009/
├── app.py              # Kode utama aplikasi REST API FastAPI
├── requirements.txt    # Daftar dependensi Python (fastapi & uvicorn)
├── Dockerfile          # Instruksi multi-stage build image Docker
├── compose.yaml        # Konfigurasi layanan dan port mapping Docker Compose
├── .dockerignore       # Daftar file/folder yang dikecualikan saat build image
├── .gitignore          # Daftar file/folder yang diabaikan oleh Git
└── README.md           # Dokumentasi proyek dan panduan penggunaan
```

---

## 1. Prasyarat (Prerequisites)

Sebelum memulai, pastikan perangkat lunak berikut sudah terpasang dan berjalan di komputer lokal:

1. **Docker Desktop** (pastikan status _Docker Engine_ dalam keadaan _Running_).
2. **Git** untuk pengelolaan versi dan pengiriman kode ke GitHub.
3. **Akun Docker Hub** yang memiliki akses ke registry Docker Hardened Images (`dhi.io`).

---

## 2. Tahap Persiapan & Inisialisasi Repository

1. Buat direktori proyek lokal sesuai format penamaan tugas, masuk ke dalamnya, lalu inisialisasi Git:

   ```bash
   mkdir docker-get-started-123103009
   cd docker-get-started-123103009
   git init
   ```

2. Hubungkan folder lokal dengan remote repository GitHub:
   ```bash
   git remote add origin [https://github.com/mardais-dev/docker-get-started-123103009.git](https://github.com/mardais-dev/docker-get-started-123103009.git)
   git remote -v
   ```

---

## 3. Tahap Pembuatan Aplikasi Python

Pada tahap ini disiapkan tiga file utama aplikasi sebelum proses containerisasi:

1. **`app.py`**: Mendefinisikan aplikasi FastAPI dengan satu endpoint `GET /` yang mengembalikan respons JSON `{"message": "Hello World"}`.
2. **`requirements.txt`**: Mengunci versi dependensi `fastapi==0.115.12` dan `uvicorn==0.34.3` agar proses instalasi di dalam container konsisten.
3. **`.gitignore`**: Mencegah file cache Python (`__pycache__/`), virtual environment (`.venv/`), dan file sensitif ikut tercatat ke dalam repository Git.

_(Opsional)_ Jika ingin menguji aplikasi secara lokal tanpa Docker terlebih dahulu (membutuhkan Python terpasang di host):

```bash
python -m venv .venv
# Aktifkan virtual environment (Windows PowerShell/CMD):
.venv\Scripts\activate
# Install dependensi dan jalankan server lokal:
pip install -r requirements.txt
uvicorn app:app --reload
```

---

## 4. Tahap Konfigurasi Docker & Login Registry

1. **Login ke Registry Docker Hardened Images (`dhi.io`)**
   Karena `Dockerfile` menggunakan base image `dhi.io/python:3.12-dev` dan `dhi.io/python:3.12`, lakukan autentikasi terlebih dahulu di terminal:

   ```bash
   docker login dhi.io
   ```

2. **Menyiapkan Asset Docker**
   - **`Dockerfile`**: Menggunakan pendekatan _multi-stage build_. Stage pertama (`builder`) membuat virtual environment `/venv` dan menginstal paket dari `requirements.txt`. Stage kedua menyalin `/venv` serta kode aplikasi ke runtime image minimal `dhi.io/python:3.12` yang berjalan sebagai user _non-root_ pada port `8000`.
   - **`compose.yaml`**: Mendefinisikan service bernama `server` yang di-build dari direktori saat ini (`.`) dan memetakan port host `8000` ke port container `8000`.
   - **`.dockerignore`**: Memastikan file lokal yang tidak diperlukan tidak ikut tersalin ke dalam _build context_ Docker.

---

## 5. Cara Build Image dan Menjalankan Container

Pastikan terminal berada di direktori root proyek (`docker-get-started-123103009`) yang berisi `Dockerfile` dan `compose.yaml`.

1. **Build Image dan Jalankan Container di Latar Belakang (`detached mode`)**

   ```bash
   docker compose up -d --build
   ```

   _Port `8000` pada host (komputer lokal) diteruskan ke port `8000` di dalam container (`8000:8000`)._

2. **Memeriksa Status Container**
   Pastikan service `server` sudah berjalan dengan status `Up` / `running`:

   ```bash
   docker compose ps
   ```

   Atau gunakan perintah standar Docker:

   ```bash
   docker ps
   ```

3. **Melihat Log Aplikasi di Dalam Container**
   Untuk memastikan server Uvicorn sudah aktif dan siap menerima permintaan:
   ```bash
   docker compose logs server
   ```

---

## 6. Cara Menguji / Mengakses Service REST API

### A. Akses dari Luar Container (Host / Komputer Lokal)

1. **Melalui Browser**
   - Akses endpoint utama: `http://localhost:8000/`
   - Akses dokumentasi interaktif bawaan FastAPI (Swagger UI): `http://localhost:8000/docs`

2. **Melalui Terminal (`curl`)**

   ```bash
   curl http://localhost:8000/
   ```

3. **Melalui Postman / Thunder Client**
   - Buat request baru dengan metode **`GET`** ke URL `http://localhost:8000/` lalu klik **Send**.

**Hasil Respons yang Diharapkan (Status `200 OK`):**

```json
{ "message": "Hello World" }
```

### B. Akses dari Dalam Container

1. **Menguji Endpoint Langsung Menggunakan Python di Dalam Container**
   Karena runtime image `dhi.io/python:3.12` merupakan _minimal hardened image_ (tidak menyertakan `curl` maupun shell interaktif secara default), pengujian dari dalam container dapat dijalankan langsung menggunakan interpreter Python di `/venv/bin/python3`:

   ```bash
   docker compose exec server /venv/bin/python3 -c "import urllib.request; print(urllib.request.urlopen('http://localhost:8000/').read().decode())"
   ```

2. **Alternatif Masuk ke Shell Container (Jika Menggunakan Image Standar yang Memiliki Shell)**
   ```bash
   docker compose exec server sh
   ```
   Setelah masuk ke dalam container, jalankan perintah pengujian berikut:
   ```bash
   python3 -c "import urllib.request; print(urllib.request.urlopen('http://localhost:8000/').read().decode())"
   ```

---

## 7. Menghentikan dan Membersihkan Container

Setelah pengujian selesai, container dapat dihentikan menggunakan salah satu perintah berikut:

- **Menghentikan container tanpa menghapusnya:**

  ```bash
  docker compose stop
  ```

- **Menghentikan sekaligus menghapus container dan network yang dibuat Compose:**
  ```bash
  docker compose down
  ```

---

## 8. Riwayat Git Commit & Push ke GitHub

Seluruh tahapan pengerjaan dicatat ke dalam riwayat commit Git (minimum 3 commit) sebelum diunggah ke GitHub:

```bash
# Commit 1: File aplikasi Python
git add app.py requirements.txt .gitignore
git commit -m "feat: create FastAPI app, requirements.txt, and .gitignore"

# Commit 2: Konfigurasi Docker & Docker Compose
git add Dockerfile compose.yaml .dockerignore
git commit -m "feat: add Dockerfile, compose.yaml, and .dockerignore"

# Commit 3: Dokumentasi proyek
git add README.md
git commit -m "docs: add detailed README.md from setup to testing"

# Verifikasi riwayat commit dan push ke branch main
git log --oneline
git branch -M main
git push -u origin main
```
