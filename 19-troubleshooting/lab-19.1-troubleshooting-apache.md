# Lab 19.1 — Quiz: Troubleshooting Apache

**Course:** Linux System Administration (Adinusa)
**Topic:** Troubleshooting Apache yang gagal start & 403 Forbidden
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Menginvestigasi kenapa **Apache gagal start**
- Menerapkan **troubleshooting skill** untuk memperbaiki config/runtime issue
- Memverifikasi Apache berjalan normal dan bisa **serve index page**
- Memahami berbagai **kemungkinan penyebab kegagalan Apache** dan solusinya

---

## 📘 Skenario Lab

Environment lab sudah **disiapkan dengan error** yang harus ditemukan dan diperbaiki **tanpa reinstall Apache**.

**Aturan:**
1. Jangan reinstall Apache.
2. Investigasi kenapa `apache2` tidak bisa start.
3. Fix config/issue agar Apache jalan lagi.
4. Pastikan `curl http://localhost` berhasil — bukan `403 Forbidden`.

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-019-1
```

---

## 🔍 Alur Troubleshooting (Step by Step)

### 1. Cek status Apache

```bash
$ sudo systemctl status apache2
```

**Output saat error (kasus lab ini):**

```
× apache2.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/apache2.service; enabled)
     Active: failed (Result: exit-code) since ...
    Process: 13008 ExecStart=/usr/sbin/apachectl start (code=exited, status=1/FAILURE)

Oct 12 01:38:59 apachectl[13010]: AH00526: Syntax error on line 148 of /etc/apache2/apache2.conf:
Oct 12 01:38:59 apachectl[13010]: Invalid command 'SSLEngine', perhaps misspelled or defined by a module not included in the server configuration
```

**Pelajaran:** Pesan error menunjukkan **file & baris** yang bermasalah — ini clue pertama.

### 2. Cek log detail dengan journalctl

```bash
$ sudo journalctl -u apache2 -n 50 --no-pager
```

Cari baris `AH00526` — itu adalah **kode error Apache** untuk syntax error.

### 3. Test syntax config

```bash
$ sudo apache2ctl configtest
```

**Output:**

```
AH00526: Syntax error on line 148 of /etc/apache2/apache2.conf:
Invalid command 'SSLEngine', perhaps misspelled or defined by a module not included in the server configuration
```

**Pelajaran:** `configtest` adalah **langkah wajib** sebelum restart Apache. Ini mencegah downtime karena config yang salah.

### 4. Cek isi baris yang error

```bash
$ sudo sed -n '140,155p' /etc/apache2/apache2.conf
```

**Output:**

```apache
IncludeOptional mods-enabled/*.conf
SSLEngine                              ← baris 148: tanpa argumen!

# Include list of ports to listen on
Include ports.conf
```

**Analisis:** Ada direktif `SSLEngine` **tanpa argumen** (`on`/`off`) di config global. Ini **salah** — `SSLEngine` seharusnya hanya ada di dalam blok `<VirtualHost *:443>`.

### 5. Cek port yang listen (untuk rule out port conflict)

```bash
$ sudo ss -tlnp | grep -E ':(80|443|8088)'
```

**Output:** (kosong) — karena Apache tidak jalan sama sekali.

### 6. Cek modul SSL

```bash
$ sudo a2enmod ssl
```

**Output:**

```
Considering dependency mime for ssl:
Module mime already enabled
Considering dependency socache_shmcb for ssl:
Enabling module socache_shmcb.
Enabling module ssl.
```

**Pelajaran:** Meskipun modul SSL diaktifkan, error tetap muncul karena masalah sebenarnya adalah **syntax error** (`SSLEngine` tanpa argumen), bukan modul yang hilang.

---

## ✅ Solusi yang Diterapkan

### Fix 1 — Hapus baris `SSLEngine` yang salah

**Backup dulu:**

```bash
$ sudo cp /etc/apache2/apache2.conf /etc/apache2/apache2.conf.bak
```

**Edit config:**

```bash
$ sudo nano /etc/apache2/apache2.conf
```

**Hapus baris 148** (`SSLEngine` tanpa argumen).

**Sebelum:**
```apache
IncludeOptional mods-enabled/*.conf
SSLEngine                              ← HAPUS
# Include list of ports to listen on
Include ports.conf
```

**Sesudah:**
```apache
IncludeOptional mods-enabled/*.conf

# Include list of ports to listen on
Include ports.conf
```

**Test ulang:**

```bash
$ sudo apache2ctl configtest
Syntax OK
```

**Restart:**

```bash
$ sudo systemctl restart apache2
$ sudo systemctl status apache2
```

Status: **active (running)** ✅

### Fix 2 — Perbaiki `Require all denied` (403 Forbidden)

**Setelah Apache jalan**, cek `curl`:

```bash
$ curl http://localhost
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
```

**Diagnosa — cek config:**

```bash
$ sudo grep -A 5 "<Directory /var/www/>" /etc/apache2/apache2.conf
```

**Output:**

```apache
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all denied          ← INI MASALAHNYA
</Directory>
```

**Fix — ubah `denied` ke `granted`:**

```bash
$ sudo nano /etc/apache2/apache2.conf
```

**Sebelum:**
```apache
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all denied
</Directory>
```

**Sesudah:**
```apache
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```

**Test & restart:**

```bash
$ sudo apache2ctl configtest
Syntax OK

$ sudo systemctl restart apache2

$ curl http://localhost
<!DOCTYPE html>
<html>
...halaman web...
```

**Verifikasi grading:**

```bash
$ nusactl grade linlab-019-1
[-] Grading lab -> linlab-019-1
    => case 1 - Check if apache2 web server is running [✓]
    => case 2 - Check curl access to Apache [✓]
[✓] Yeay! you're completed the task
[-] Score -> 100
```

---

## 📋 Ringkasan Perbaikan

| # | Masalah | Fix |
|---|---|---|
| 1 | `SSLEngine` tanpa argumen di `apache2.conf` line 148 | Hapus baris tersebut |
| 2 | `Require all denied` di `<Directory /var/www/>` | Ubah ke `Require all granted` |

**Urutan fix:**
1. Perbaiki syntax error dulu → Apache bisa start
2. Baru perbaiki permission/access → curl berhasil

---

## 🧪 Kemungkinan Error Apache & Solusinya

Berikut **semua kemungkinan penyebab Apache gagal start atau 403**, berdasarkan pengalaman lab dan dunia nyata.

### A. Apache Gagal Start

| # | Error Message | Penyebab | Solusi |
|---|---|---|---|
| 1 | `Invalid command 'SSLEngine'` | Modul SSL belum aktif, atau `SSLEngine` tanpa argumen | `sudo a2enmod ssl` + hapus baris `SSLEngine` yang salah |
| 2 | `SSLEngine takes one argument` | `SSLEngine` tanpa `on`/`off` di config global | Hapus dari `apache2.conf`, pindah ke vhost HTTPS |
| 3 | `AH00526: Syntax error on line X` | Config Apache ada typo/syntax salah | Perbaiki baris X sesuai pesan error |
| 4 | `AH00072: make_sock: could not bind to address` | Port 80 sudah dipakai proses lain | `sudo ss -tlnp \| grep :80`, stop service yang pakai |
| 5 | `Could not open configuration file` | File config hilang / path salah | Cek `ls /etc/apache2/`, restore dari backup |
| 6 | `Permission denied: AH00072` | Apache jalan sebagai non-root & port < 1024 | Pastikan Apache jalan sebagai root, atau pakai port > 1024 |
| 7 | `Module <x> not found` | Modul tidak terinstal | `sudo apt install libapache2-mod-<x>` atau `sudo a2enmod <x>` |
| 8 | `Failed to start: Unit apache2.service not found` | Apache belum terinstall | `sudo apt install apache2 -y` |
| 9 | `Address already in use` | Proses lain (nginx, apache lama) masih pakai port | `sudo killall nginx apache2`, restart |
| 10 | `No space left on device` | Disk penuh | `df -h` → hapus file besar → restart |
| 11 | `apache2: bad user name ${APACHE_RUN_USER}` | Env var hilang di `/etc/apache2/envvars` | Cek file `envvars`, pastikan `APACHE_RUN_USER=www-data` |
| 12 | `Cannot load modules/libphp.so` | PHP module tidak kompatibel | Disable module PHP atau install ulang versi yang cocok |

### B. Apache Jalan Tapi 403 Forbidden

| # | Penyebab | Cek | Solusi |
|---|---|---|---|
| 1 | `Require all denied` di config | `sudo grep -A 5 "<Directory /var/www/>" /etc/apache2/apache2.conf` | Ubah ke `Require all granted` |
| 2 | File `index.html` tidak ada | `ls /var/www/html/` | Buat index.html atau aktifkan `Options Indexes` |
| 3 | Permission file terlalu ketat | `ls -la /var/www/html/` | `sudo chmod 644 /var/www/html/index.html` |
| 4 | Permission direktori terlalu ketat | `ls -ld /var/www/html/` | `sudo chmod 755 /var/www/html` |
| 5 | Owner salah (bukan `www-data`) | `ls -la /var/www/html/` | `sudo chown -R www-data:www-data /var/www/html` |
| 6 | Directory listing disabled | Error log: `Directory index forbidden` | Buat `index.html`, atau set `Options Indexes` |
| 7 | `.htaccess` memblokir | Cek error log | Rename `.htaccess` sementara untuk debug |
| 8 | SELinux context salah (RHEL/CentOS) | `ls -Z /var/www/html/` | `sudo restorecon -Rv /var/www/html` |
| 9 | Vhost tidak match | `apache2ctl -S` | Cek `ServerName` / `ServerAlias` |
| 10 | Default vhost menimpa vhost custom | `ls /etc/apache2/sites-enabled/` | `sudo a2dissite 000-default.conf` |

### C. Apache Jalan Tapi 404 Not Found

| # | Penyebab | Solusi |
|---|---|---|
| 1 | File tidak ada di DocumentRoot | Cek `ls <DocumentRoot>` |
| 2 | DocumentRoot salah path | Cek config vhost |
| 3 | URL salah / typo | Cek `curl -v` untuk lihat request |
| 4 | Rewrite rule salah | Cek `mod_rewrite` config |

### D. Apache Jalan Tapi 500 Internal Server Error

| # | Penyebab | Solusi |
|---|---|---|
| 1 | Error di PHP script | Cek `/var/log/apache2/error.log` |
| 2 | `.htaccess` syntax salah | Cek `AllowOverride` + isi `.htaccess` |
| 3 | Permission CGI script | `chmod +x script.cgi` |
| 4 | Module konflik | Disable module yang dicurigai |

### E. Error Warning (Non-Fatal)

| # | Warning | Arti |
|---|---|---|
| 1 | `AH00558: Could not reliably determine FQDN` | Warning saja, Apache tetap jalan. Fix: tambah `ServerName localhost` di `apache2.conf` |
| 2 | `Could not open log file` | Directory log tidak ada, buat: `sudo mkdir -p /var/log/apache2 && sudo chown www-data /var/log/apache2` |

---

## 🛠️ Cheatsheet Perintah Troubleshooting Apache

### Cek Status & Log

| Perintah | Fungsi |
|---|---|
| `sudo systemctl status apache2` | Cek status + error terakhir |
| `sudo journalctl -u apache2 -n 50` | Log systemd |
| `sudo tail -50 /var/log/apache2/error.log` | Log error Apache |
| `sudo tail -50 /var/log/apache2/access.log` | Log akses |
| `sudo apache2ctl configtest` | Validasi syntax config |
| `sudo apache2ctl -S` | Ringkasan vhost & port |

### Cek Config & File

| Perintah | Fungsi |
|---|---|
| `sudo sed -n 'X,Yp' /etc/apache2/apache2.conf` | Lihat baris X-Y |
| `sudo grep -rn "SSLEngine" /etc/apache2/` | Cari direktif tertentu |
| `sudo grep -A 5 "<Directory /var/www/>" /etc/apache2/apache2.conf` | Cek blok Directory |
| `ls -la /var/www/html/` | Cek file web |
| `ls -ld /var/www/html/` | Cek permission direktori |

### Cek Port & Proses

| Perintah | Fungsi |
|---|---|
| `sudo ss -tlnp \| grep -E ':(80\|443\|8088)'` | Cek port yang listen |
| `sudo lsof -i :80` | Proses yang pakai port 80 |
| `ps aux \| grep apache` | Cek proses Apache |

### Fix Permission

| Perintah | Fungsi |
|---|---|
| `sudo chown -R www-data:www-data /var/www/html` | Fix owner |
| `sudo chmod 755 /var/www/html` | Fix permission direktori |
| `sudo chmod 644 /var/www/html/*.html` | Fix permission file |

### Enable/Disable Module & Site

| Perintah | Fungsi |
|---|---|
| `sudo a2enmod ssl` | Aktifkan modul SSL |
| `sudo a2enmod rewrite` | Aktifkan mod rewrite |
| `sudo a2ensite <site>.conf` | Aktifkan vhost |
| `sudo a2dissite 000-default.conf` | Nonaktifkan default vhost |
| `sudo a2query -m` | Cek modul aktif |
| `sudo a2query -s` | Cek site aktif |

### Debug Request

| Perintah | Fungsi |
|---|---|
| `curl -v http://localhost` | Debug HTTP request/response |
| `curl -I http://localhost` | Hanya lihat header |
| `curl -k https://localhost` | Ignore SSL warning |
| `wget --spider http://localhost` | Cek URL valid |

---

## 🎯 Studi Kasus: Lab Adinusa 19.1

### Konteks

Environment lab **sengaja dirusak** dengan 2 issue bertingkat:

1. **Syntax error** — `SSLEngine` tanpa argumen di `apache2.conf` line 148
2. **Access denied** — `Require all denied` di blok `<Directory /var/www/>`

Kedua issue ini harus di-fix **berurutan** — fix syntax dulu, baru fix akses.

### Timeline Investigasi

| Langkah | Perintah | Temuan |
|---|---|---|
| 1 | `systemctl status apache2` | `AH00526: Syntax error on line 148` |
| 2 | `journalctl -u apache2` | Konfirmasi error baris 148 |
| 3 | `apache2ctl configtest` | Error `Invalid command 'SSLEngine'` |
| 4 | `sed -n '140,155p' apache2.conf` | Baris 148 = `SSLEngine` (tanpa argumen) |
| 5 | `a2enmod ssl` | Modul SSL aktif, tapi error tetap |
| 6 | Edit config — hapus baris 148 | `Syntax OK` |
| 7 | `systemctl restart apache2` | Apache **running** ✅ |
| 8 | `curl http://localhost` | **403 Forbidden** ❌ |
| 9 | `grep -A 5 "<Directory /var/www/>"` | `Require all denied` |
| 10 | Edit config — ubah ke `granted` | — |
| 11 | Restart + curl | **Berhasil** ✅ |
| 12 | `nusactl grade` | **Score 100** 🎉 |

### Pelajaran dari Studi Kasus

1. **Error bisa bertingkat** — fix pertama membuka jalan ke masalah kedua.
2. **Pesan error sangat informatif** — selalu baca, jangan tebak.
3. **`configtest` wajib** — mencegah downtime.
4. **`Require all denied` adalah penyebab 403 paling umum** di Apache 2.4.
5. **Urutan fix penting** — syntax dulu, akses kemudian.

---

## 💡 Catatan & Insight Pribadi

### Prinsip Troubleshooting

1. **Baca error dulu** — jangan tebak-tebakan.
2. **Isolasi masalah** — config? permission? port? service lain?
3. **Ubah satu hal, test** — jangan ubah banyak hal sekaligus.
4. **Selalu `configtest`** sebelum restart.
5. **Cek log** kalau masih gagal.
6. **Error bertingkat** — fix satu, mungkin muncul berikutnya.

### Tiga Penyebab Utama Apache Gagal Start

| Kategori | Persentase | Contoh |
|---|---|---|
| **Config error** | 50% | Syntax salah, `;` hilang, direktif salah tempat |
| **Permission** | 30% | File/dir tidak bisa dibaca www-data |
| **Port conflict** | 20% | Nginx atau service lain pakai port 80 |

### Analogi Troubleshooting

Seperti **dokter**:
- **Anamnesis** = tanya pasien (`systemctl status`)
- **Diagnosis** = cek lab (`journalctl`, `configtest`)
- **Treatment** = perbaiki (`nano` config)
- **Verifikasi** = cek sembuh (`curl`)

### Perbedaan HTTP Error Code

| Code | Arti | Penyebab Umum |
|---|---|---|
| **403** | Forbidden | Permission, `Require` config |
| **404** | Not Found | File tidak ada |
| **500** | Internal Server Error | PHP/CGI crash, `.htaccess` |
| **502** | Bad Gateway | Reverse proxy backend down |
| **503** | Service Unavailable | Server overload / maintenance |

### Kapan Reinstall Apache?

**Hampir tidak pernah.** Reinstall hanya kalau:
- File binary apache rusak
- Upgrade gagal total
- Config hilang dan tidak bisa direstore

**Troubleshooting 99% cukup** dengan fix config/permission/port.

### Kasus Nyata

- **After update:** Config baru konflik dengan lama → `configtest` untuk cek.
- **After install PHP:** Module `php` belum enable → `a2enmod php`.
- **Setelah ganti domain:** Vhost lama masih aktif → `a2dissite 000-default`.
- **File permission berubah:** Karena `chmod -R` tanpa pikir → cek `ls -la`.
- **Sisa config lab lama:** `SSLEngine` di `apache2.conf` → hapus.

### Pelajaran Kunci dari Quiz Ini

1. **Jangan reinstall** — hampir selalu bisa di-fix.
2. **`systemctl status` + `journalctl`** = pintu masuk troubleshooting.
3. **`apache2ctl configtest`** wajib sebelum restart.
4. **Cek log** — error message kasih clue spesifik.
5. **403 Forbidden** = `Require all denied` atau permission.
6. **Config bisa bermasalah bertingkat** — fix satu, cek lagi.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `sudo systemctl status apache2` | Cek status |
| `sudo journalctl -u apache2 -n 50` | Log systemd |
| `sudo tail -50 /var/log/apache2/error.log` | Log error |
| `sudo apache2ctl configtest` | Validasi syntax |
| `sudo apache2ctl -S` | Ringkasan vhost |
| `sudo ss -tlnp \| grep :80` | Cek port 80 |
| `sudo sed -n 'X,Yp' /etc/apache2/apache2.conf` | Lihat baris config |
| `sudo grep -rn "pattern" /etc/apache2/` | Cari direktif |
| `sudo a2enmod <mod>` | Enable module |
| `sudo a2ensite <site>` | Enable site |
| `sudo a2dissite <site>` | Disable site |
| `sudo systemctl restart apache2` | Restart |
| `sudo chown -R www-data:www-data /var/www/html` | Fix owner |
| `sudo chmod 755 /var/www/html` | Fix permission dir |
| `sudo chmod 644 /var/www/html/index.html` | Fix permission file |
| `curl -v http://localhost` | Debug request |

**File penting:**

| File | Isi |
|---|---|
| `/etc/apache2/apache2.conf` | Config utama |
| `/etc/apache2/ports.conf` | Port listen |
| `/etc/apache2/sites-available/` | Config vhost (draft) |
| `/etc/apache2/sites-enabled/` | Config aktif (symlink) |
| `/var/www/html/index.html` | Homepage default |
| `/var/log/apache2/error.log` | Log error |
| `/var/log/apache2/access.log` | Log akses |

---

## 📌 Kesimpulan

Lab ini adalah **quiz troubleshooting** — menguji kemampuan mendiagnosa masalah Apache tanpa reinstall. Skill yang diuji:

- Membaca **error log** dan **status systemd**.
- **Validasi syntax** config sebelum restart.
- Mengidentifikasi penyebab umum: config error, permission, port conflict.
- Fix dengan **minimal changes** dan verifikasi bertahap.
- Memahami **error bertingkat** — fix satu, cek lagi.

Yang paling penting dipahami: **Apache jarang rusak permanen**. 99% masalah bisa di-fix dengan:

1. Baca error.
2. Cek config (`configtest`).
3. Cek permission (`ls -la`).
4. Cek port (`ss -tlnp`).
5. Restart & verify (`curl`).

Kemampuan troubleshooting ini adalah **inti dari sysadmin** — server produksi pasti akan mengalami masalah, dan yang membedakan sysadmin baik adalah cara mendiagnosanya.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
