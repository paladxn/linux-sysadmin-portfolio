# Lab 16.3 — Quiz: Securing Port Webserver (Nginx)

**Course:** Linux System Administration (Adinusa)
**Topic:** Nginx + custom port + document root + firewalld
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Menginstall dan mengonfigurasi **Nginx** web server
- Mengubah **port default** Nginx
- Mengatur **document root** custom
- Mengganti halaman index default dengan konten kustom
- Mengonfigurasi **firewalld** untuk port non-standard
- Memverifikasi akses via browser

---

## 📘 Konsep Dasar

**Nginx** adalah web server populer selain Apache. Lebih ringan, cepat untuk static content, dan sering dipakai sebagai reverse proxy.

| Aspek | Apache | Nginx |
|---|---|---|
| Model | Process-based | Event-driven |
| Config | `.htaccess` + vhost | Block server |
| Performa | Baik | Sangat baik untuk static |
| Port default | 80 | 80 |

**Struktur direktori Nginx (Ubuntu):**

```
/etc/nginx/
├── nginx.conf                  ← config utama
├── sites-available/            ← config site (draft)
├── sites-enabled/              ← config aktif (symlink)
└── conf.d/                     ← config tambahan
/var/www/html/                  ← document root default
```

**Kenapa port custom?**
- Menghindari konflik dengan Apache (kalau ada).
- Isolasi aplikasi.
- Firewall bisa atur per-port.

**Kenapa firewalld?**
- Zone-based firewall.
- Kontrol interface per zone.
- Runtime vs permanent config terpisah.

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-016-3
```

> **Catatan:** Ganti `USERNAME` dengan username Adinusa masing-masing.

---

## 🧪 Quiz Task

### Soal

1. Install **Nginx** di sistem.
2. Konfigurasi Nginx agar listen di port **8080** (bukan default 80).
3. Ubah **document root** ke `/opt/lab162`.
4. Ganti `index.html` di `/opt/lab162` dengan konten:
   ```
   Hello Adinusa, I am USERNAME
   ```
   *(tanpa tag HTML)*
5. Konfigurasi **firewalld** agar mengizinkan port **8080** di zone **public**.
6. Verifikasi akses via browser.

### ✅ Solusi

#### 1. Install Nginx

```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl enable --now nginx
```

**Verifikasi:**
```bash
sudo systemctl status nginx
```

#### 2. Buat document root & index.html

```bash
sudo mkdir -p /opt/lab162
sudo nano /opt/lab162/index.html
```

Isi (tanpa tag HTML):
```
Hello Adinusa, I am USERNAME
```

#### 3. Ubah config Nginx

Edit file default site:

```bash
sudo nano /etc/nginx/sites-available/default
```

Ubah menjadi:

```nginx
server {
    listen 8080 default_server;
    listen [::]:8080 default_server;

    root /opt/lab162;
    index index.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Penjelasan perubahan:**

| Sebelum | Sesudah |
|---|---|
| `listen 80 default_server;` | `listen 8080 default_server;` |
| `listen [::]:80 default_server;` | `listen [::]:8080 default_server;` |
| `root /var/www/html;` | `root /opt/lab162;` |

#### 4. Test config & reload Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

**Output `nginx -t` yang diharapkan:**
```
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

#### 5. Konfigurasi firewalld

```bash
# Install firewalld (kalau belum)
sudo apt install firewalld -y

# Pastikan firewalld aktif
sudo systemctl enable --now firewalld

# Buka port 8080 di zone public
sudo firewall-cmd --add-port=8080/tcp --zone=public --permanent

# Reload
sudo firewall-cmd --reload

# Verifikasi
sudo firewall-cmd --list-all --zone=public
```

Output harus menampilkan `ports: 8080/tcp`.

#### 6. Verifikasi

**Dari VM:**
```bash
curl http://localhost:8080
```

Output: `Hello Adinusa, I am USERNAME`

**Dari browser Windows:**
```
http://172.17.10.10:8080
```

---

## 🔍 Verifikasi Akhir

### 1. Cek Nginx listen di port 8080

```bash
sudo ss -tlnp | grep :8080
```

Harus muncul `LISTEN`.

### 2. Cek config Nginx

```bash
sudo nginx -T | grep -E 'listen|root'
```

Harus menampilkan `listen 8080` dan `root /opt/lab162`.

### 3. Cek firewalld aktif

```bash
sudo firewall-cmd --state
running

sudo firewall-cmd --list-all --zone=public
```

### 4. Cek konten index.html

```bash
cat /opt/lab162/index.html
```

Harus: `Hello Adinusa, I am USERNAME`

---

## 🐛 Troubleshooting & Kesalahan

- **`nginx -t` gagal syntax error:** Cek indentasi dan titik koma. Nginx sangat strict — setiap direktif harus diakhiri `;`. Jalankan `sudo nginx -t` lagi setelah perbaikan.

- **Port 8080 sudah dipakai proses lain:** Cek:
  ```bash
  sudo ss -tlnp | grep :8080
  ```
  Kalau ada, matikan proses itu atau ganti port.

- **`403 Forbidden`:** Permission `/opt/lab162` salah. Fix:
  ```bash
  sudo chown -R www-data:www-data /opt/lab162
  sudo chmod -R 755 /opt/lab162
  ```
  Nginx default pakai user `www-data`.

- **`404 Not Found`:** File `index.html` tidak ada atau salah nama. Cek:
  ```bash
  ls -la /opt/lab162/
  ```

- **Nginx gagal start:** Cek log:
  ```bash
  sudo journalctl -u nginx --no-pager | tail -20
  sudo tail -20 /var/log/nginx/error.log
  ```

- **Akses dari Windows timeout:** Cek ping dari Windows:
  ```cmd
  ping 172.17.10.10
  ```
  Kalau timeout, masalah di adapter VirtualBox / Windows hosts — bukan Nginx.

- **Firewalld block akses:** Cek port sudah dibuka:
  ```bash
  sudo firewall-cmd --list-ports --zone=public
  ```

- **SELinux block akses ke `/opt`:** Di Ubuntu biasanya SELinux tidak aktif. Kalau di RHEL/CentOS, perlu:
  ```bash
  sudo semanage fcontext -a -t httpd_sys_content_t "/opt/lab162(/.*)?"
  sudo restorecon -Rv /opt/lab162
  ```

- **`nginx: [emerg] bind() to 0.0.0.0:8080 failed`:** Port sudah dipakai atau permission denied. Cek `ss -tlnp | grep :8080`.

- **Config di `/etc/nginx/nginx.conf` vs `sites-available/default`:** Nginx pakai config di `sites-enabled/`, bukan `sites-available/`. Kalau edit `sites-available/default`, pastikan symlink-nya aktif:
  ```bash
  ls -la /etc/nginx/sites-enabled/
  ```

- **Lupa reload Nginx:** Setelah edit config, harus `sudo systemctl reload nginx`. Tanpa reload, perubahan tidak berlaku.

- **Firewalld tidak aktif:** Cek konflik dengan `ufw`:
  ```bash
  sudo systemctl disable --now ufw
  sudo systemctl enable --now firewalld
  ```

- **Port 8080 dibuka tapi akses dari luar tetap gagal:** Firewalld hanya mengizinkan akses dari interface yang di-assign ke zone public. Cek:
  ```bash
  sudo firewall-cmd --list-all --zone=public
  ```
  Kalau `interfaces:` kosong, tambahkan:
  ```bash
  sudo firewall-cmd --add-interface=enp0s8 --permanent
  sudo firewall-cmd --reload
  ```

---

## 💡 Catatan & Insight Pribadi

### Apache vs Nginx

| Aspek | Apache | Nginx |
|---|---|---|
| Model | Process/thread per request | Event-driven async |
| Config | `.htaccess`, vhost | Block server |
| Static content | Bagus | **Sangat bagus** |
| Dynamic (PHP) | Native via mod_php | Butuh PHP-FPM |
| Memory | Lebih besar | Lebih kecil |
| Reverse proxy | Bisa | **Excellent** |
| Cocok untuk | PHP app tradisional | Static, high-traffic, reverse proxy |

**Kapan pakai Nginx:**
- Static website / SPA (React, Vue build).
- Reverse proxy di depan Node.js, Python, Go.
- High-traffic server.
- Load balancer.

**Kapan pakai Apache:**
- PHP app tradisional (WordPress, Laravel).
- Butuh `.htaccess`.
- Butuh modul Apache spesifik.

### Struktur Config Nginx

```nginx
server {
    listen 8080 default_server;       # ← port
    listen [::]:8080 default_server;  # ← IPv6

    root /opt/lab162;                 # ← document root
    index index.html;                 # ← file default

    server_name _;                    # ← catch-all

    location / {
        try_files $uri $uri/ =404;    # ← fallback
    }
}
```

### Nginx Config Test — Wajib Sebelum Reload

```bash
sudo nginx -t
```

Ini akan cek syntax config **tanpa** apply. Kalau OK, baru reload. Tanpa `nginx -t`, kalau ada typo, Nginx bisa mati.

### Firewalld Zone — Konsep

| Zone | Kepercayaan |
|---|---|
| `public` | Rendah — default |
| `trusted` | Tinggi |
| `home` | Sedang |
| `work` | Sedang |
| `drop` | Block semua |

Interface yang di-assign ke zone mengikuti aturan zone itu. Default, interface baru masuk ke `public`.

### Runtime vs Permanent di Firewalld

| Perintah | Efek |
|---|---|
| `firewall-cmd --add-port=8080/tcp` | Runtime saja (hilang setelah reboot) |
| `firewall-cmd --add-port=8080/tcp --permanent` | Permanent (perlu reload) |
| `firewall-cmd --reload` | Terapkan permanent ke runtime |

**Selalu pakai `--permanent` lalu `--reload`** untuk perubahan yang ingin persist.

### Kasus Nyata

- **Reverse proxy:** Nginx di port 80 → forward ke app di port 8080.
- **Multi-app server:** Nginx di 80, 443, 8080 untuk aplikasi berbeda.
- **Static hosting:** Nginx serve file statis dengan performa tinggi.
- **Security:** Port non-standard mengurangi serangan otomatis (scanner biasanya scan port 80, 443 saja).

### Best Practice

1. **Selalu `nginx -t`** sebelum reload.
2. **Pakai `www-data:www-data`** untuk file web.
3. **Batasi akses firewalld** — hanya port yang diperlukan.
4. **Pakai port non-standard** untuk admin panel.
5. **Pakai HTTPS** untuk production.
6. **Jangan pakai `root /`** — bahaya.
7. **Log ke `/var/log/nginx/`** untuk debugging.
8. **Pisah config per-site** di `sites-available/` untuk maintainability.

### Pelajaran Kunci dari Lab Ini

1. **Nginx** = web server ringan, event-driven.
2. **`listen 8080`** mengubah port.
3. **`root`** menentukan document root.
4. **`nginx -t`** wajib sebelum reload.
5. **Firewalld** kontrol akses per-port & per-interface.
6. **`--permanent` + `--reload`** = commit perubahan firewalld.
7. **Permission `www-data`** untuk file web.
8. **Port non-standard** = sedikit mengurangi attack surface.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `sudo apt install nginx -y` | Install Nginx |
| `sudo systemctl enable --now nginx` | Enable + start Nginx |
| `sudo systemctl status nginx` | Cek status |
| `sudo nginx -t` | Test syntax config |
| `sudo systemctl reload nginx` | Reload config |
| `sudo systemctl restart nginx` | Restart service |
| `sudo nano /etc/nginx/sites-available/default` | Edit config default |
| `sudo ss -tlnp \| grep :8080` | Cek port listen |
| `sudo firewall-cmd --add-port=8080/tcp --zone=public --permanent` | Buka port |
| `sudo firewall-cmd --reload` | Reload firewalld |
| `sudo firewall-cmd --list-all --zone=public` | Lihat config zone |
| `curl http://localhost:8080` | Verifikasi web |

**File penting:**

| File | Isi |
|---|---|
| `/etc/nginx/nginx.conf` | Config utama |
| `/etc/nginx/sites-available/default` | Config site default |
| `/etc/nginx/sites-enabled/default` | Symlink config aktif |
| `/opt/lab162/index.html` | Konten homepage |
| `/var/log/nginx/access.log` | Log akses |
| `/var/log/nginx/error.log` | Log error |

**Contoh config server block:**

```nginx
server {
    listen 8080 default_server;
    listen [::]:8080 default_server;

    root /opt/lab162;
    index index.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Firewalld commands:**

| Perintah | Fungsi |
|---|---|
| `firewall-cmd --state` | Cek status |
| `firewall-cmd --list-all --zone=public` | Lihat config zone |
| `firewall-cmd --add-port=8080/tcp --zone=public --permanent` | Buka port |
| `firewall-cmd --remove-port=8080/tcp --zone=public --permanent` | Tutup port |
| `firewall-cmd --add-interface=enp0s8 --permanent` | Assign interface |
| `firewall-cmd --reload` | Reload |

---

## 📌 Kesimpulan

Lab ini adalah **quiz gabungan** dari topik web server (Nginx) + konfigurasi port + document root + firewalld. Kemampuan yang diuji:

- Menginstall & mengonfigurasi **Nginx**.
- Mengubah **port listening** dan **document root**.
- Mengganti **index.html** kustom.
- Mengonfigurasi **firewalld** untuk port non-standard.
- Verifikasi akses dari browser.

Yang paling penting dipahami: **Nginx sebagai alternatif Apache**. Nginx lebih ringan dan cepat, terutama untuk static content. Di production, Nginx sering dipakai sebagai **reverse proxy** di depan aplikasi Node.js/Python/Go. Firewalld memberi kontrol granular per-port & per-interface.

Kemampuan ini sangat relevan di dunia kerja — web server adalah tulang punggung hampir semua layanan modern.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
