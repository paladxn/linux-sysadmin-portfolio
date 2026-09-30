# Lab 6.2 — Menambahkan Repository Eksternal & Instalasi Versi Spesifik

**Course:** Linux System Administration (Adinusa)
**Topic:** External repository & version pinning
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Menambahkan **repository eksternal** ke sistem melalui `/etc/apt/sources.list.d/`
- Menginstal paket dengan **versi spesifik** dari repository tersebut
- Memverifikasi bahwa repository dan paket sudah terpasang sesuai kebutuhan

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-006-2
```

Login menggunakan kredensial Adinusa, lalu jalankan script persiapan lab.

---

## 📘 Guided Example

### 1. Memahami struktur repository APT

Repository APT di Debian/Ubuntu dikonfigurasi melalui file `.list` di:

- `/etc/apt/sources.list` → file utama (jarang diedit langsung di sistem modern)
- `/etc/apt/sources.list.d/` → folder untuk repository tambahan

Setiap file `.list` berisi baris dengan format:

```
deb [opsi] URL DISTRIBUTION COMPONENT
```

### 2. Menambahkan repository MariaDB 10.11

Ada dua cara yang bisa dipakai. **Cara pertama** (direkomendasikan untuk lab ini) menggunakan script resmi MariaDB:

```bash
# Download script setup
$ curl -LsS https://r.mariadb.com/downloads/mariadb_repo_setup -o mariadb_repo_setup

# Buat script executable
$ chmod +x mariadb_repo_setup

# Jalankan dengan versi yang diinginkan
$ sudo ./mariadb_repo_setup --mariadb-server-version="mariadb-10.11"
```

Script ini akan otomatis:
- Membuat file `/etc/apt/sources.list.d/mariadb.list`
- Menambahkan GPG key MariaDB
- Menjalankan `apt update`

**Cara kedua** (manual) — jika ingin membuat file sendiri:

```bash
$ sudo tee /etc/apt/sources.list.d/mariadb.list << EOF
deb [arch=amd64,arm64] https://dlm.mariadb.com/repo/mariadb-server/10.11/repo/ubuntu noble main
EOF
```

> **Catatan:** Ganti `noble` dengan codename distro yang kamu pakai (misal `jammy`, `focal`). Cek dengan `lsb_release -cs`.

### 3. Update package index

Setelah repository ditambahkan, update daftar paket:

```bash
$ sudo apt update
```

### 4. Cek versi yang tersedia

Sebelum instalasi, lihat versi apa saja yang tersedia dari repository:

```bash
$ apt-cache policy mariadb-server
```

Contoh output:

```
mariadb-server:
  Installed: (none)
  Candidate: 1:10.11.9+maria~ubu2404
  Version table:
     1:10.11.9+maria~ubu2404 500
        500 https://dlm.mariadb.com/repo/mariadb-server/10.11/repo/ubuntu noble/main amd64 Packages
```

### 5. Instal MariaDB versi 10.11

Instal dengan **versi spesifik**:

```bash
$ sudo apt install mariadb-server=1:10.11.* -y
```

Atau, jika hanya ingin memastikan berasal dari repository 10.11:

```bash
$ sudo apt install mariadb-server -y
```

Karena repository yang aktif hanya menyediakan versi 10.11, APT akan otomatis memilih versi tersebut.

### 6. Verifikasi instalasi

```bash
$ mariadb --version
```

Contoh output:

```
mariadb  Ver 15.1 Distrib 10.11.9-MariaDB, for debian-linux-gnu (x86_64)
```

---

## 🧪 Practice Task

### Soal

1. Tambahkan repository MariaDB 10.11 ke `/etc/apt/sources.list.d/` dengan nama file **`mariadb.list`**.
2. Update package index.
3. Instal **MariaDB versi 10.11** (pilih release di bawah 10.11.*, bukan versi lain).
4. Verifikasi bahwa repository sudah terpasang dan `mariadb-server` sudah terinstal.

### ✅ Solusi

```bash
# 1. Tambahkan repository (via script resmi)
$ curl -LsS https://r.mariadb.com/downloads/mariadb_repo_setup -o mariadb_repo_setup
$ chmod +x mariadb_repo_setup
$ sudo ./mariadb_repo_setup --mariadb-server-version="mariadb-10.11"

# 2. Update index
$ sudo apt update

# 3. Instal versi spesifik 10.11
$ sudo apt install mariadb-server=1:10.11.* -y

# 4. Verifikasi
$ apt-cache policy mariadb-server
$ mariadb --version
```

### 🔍 Hasil Akhir yang Diharapkan

**File repository sudah ada:**

```bash
$ cat /etc/apt/sources.list.d/mariadb.list
deb [arch=amd64,arm64] https://dlm.mariadb.com/repo/mariadb-server/10.11/repo/ubuntu noble main
```

**Versi MariaDB yang terinstal:**

```bash
$ mariadb --version
mariadb  Ver 15.1 Distrib 10.11.x-MariaDB, for debian-linux-gnu (x86_64)
```

---

## 🐛 Troubleshooting & Kesalahan

- **Salah codename distro:** Saat pertama membuat file manual, saya sempat menulis `jammy` padahal sistemnya Ubuntu 24.04 (`noble`). Akibatnya `apt update` gagal dengan error *"404 Not Found"*. Solusinya: cek codename dengan `lsb_release -cs` sebelum menulis file repository.

- **GPG key error:** Setelah menambahkan repository manual, muncul error *"NO_PUBKEY"*. Ternyata GPG key MariaDB belum ditambahkan. Solusinya: pakai script resmi `mariadb_repo_setup` yang otomatis menangani key, atau tambahkan key manual dengan `curl` + `apt-key add`.

- **Konflik dengan repository bawaan Ubuntu:** Ubuntu 24.04 sendiri sudah menyediakan MariaDB 10.11 di repository universe. Kalau repository eksternal dan repository bawaan sama-sama menyediakan versi yang sama, APT bisa bingung memilih. Solusinya: prioritaskan repository eksternal dengan **pinning** atau pastikan repository eksternal punya priority lebih tinggi.

- **Instalasi versi spesifik gagal:** Saat mencoba `sudo apt install mariadb-server=10.11.9`, muncul error *"Version '10.11.9' for 'mariadb-server' was not found"*. Ternyata format versi di repository MariaDB adalah `1:10.11.9+maria~ubu2404`. Solusinya: gunakan wildcard `1:10.11.*` atau cek dulu versi yang tersedia dengan `apt-cache policy mariadb-server`.

- **File repository tidak bernama `mariadb.list`:** Script resmi MariaDB membuat file bernama `mariadb.list`, tapi beberapa panduan lama membuat `MariaDB.list` (huruf besar). Di Linux, nama file **case-sensitive**. Pastikan namanya persis `mariadb.list` sesuai ketentuan lab.

---

## 💡 Catatan & Insight Pribadi

- **Kenapa perlu repository eksternal?** Repository bawaan distro sering kali menyediakan versi yang lebih lama atau tidak menyediakan versi spesifik yang dibutuhkan aplikasi. Repository eksternal memungkinkan kita mendapatkan versi yang lebih baru atau spesifik.

- **Risiko repository eksternal:**
  - Bisa menimpa paket sistem dengan versi yang tidak kompatibel.
  - Perlu update manual jika ingin pindah versi.
  - GPG key harus dikelola dengan benar.
  - Sebaiknya hanya gunakan repository resmi dari vendor (MariaDB, Docker, PostgreSQL, dll).

- **`sources.list.d/` vs `sources.list`:** File di `sources.list.d/` dibaca **setelah** `sources.list`. Jadi kalau ada konflik, file di `sources.list.d/` bisa menimpa. Ini memudahkan pengelolaan repository pihak ketiga tanpa mengedit file utama.

- **Version pinning:** Selain `apt install paket=versi`, kita juga bisa mengunci versi dengan `apt-mark hold paket` agar tidak ter-upgrade otomatis. Ini berguna di server produksi yang butuh stabilitas versi.

- **`apt-cache policy`:** Perintah ini sangat berguna untuk melihat:
  - Versi yang terinstal
  - Versi kandidat (yang akan diinstal)
  - Daftar repository yang menyediakan paket tersebut

- **Kasus nyata:** Di server produksi, sering kali kita perlu menginstal versi spesifik dari vendor (misal MySQL 8.0, PostgreSQL 15, Nginx 1.24) yang tidak tersedia di repository bawaan. Menguasai cara menambahkan repository eksternal adalah skill wajib sysadmin.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `curl -LsS <url> -o file` | Download file dari internet |
| `chmod +x script` | Buat script executable |
| `sudo ./mariadb_repo_setup --mariadb-server-version="mariadb-10.11"` | Setup repository MariaDB 10.11 |
| `sudo tee /etc/apt/sources.list.d/mariadb.list` | Buat file repository manual |
| `apt-cache policy <paket>` | Cek versi yang tersedia |
| `sudo apt install <paket>=<versi>` | Instal versi spesifik |
| `mariadb --version` | Cek versi MariaDB terinstal |
| `lsb_release -cs` | Cek codename distro |

---

## 📌 Kesimpulan

Lab ini memperkenalkan konsep **external repository** dan **version pinning** — dua hal yang sangat penting di dunia sysadmin. Menambahkan repository eksternal memungkinkan kita menginstal versi paket yang lebih baru atau spesifik yang tidak tersedia di repository bawaan distro. Namun, ini juga membawa risiko: konflik dependensi, GPG key, dan kompatibilitas. Memahami cara kerja `sources.list.d/`, `apt-cache policy`, dan version pinning adalah bekal wajib sebelum mengelola server produksi.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.