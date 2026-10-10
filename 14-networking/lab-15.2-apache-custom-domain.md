# Lab 15.2 — Apache Custom Domain Configuration

**Course:** Linux System Administration (Adinusa)
**Topic:** Apache web server & virtual host dengan custom domain
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Menginstal dan mengonfigurasi **Apache web server**
- Membuat **document root** dan homepage custom
- Mengonfigurasi **virtual host** untuk domain custom
- Memverifikasi virtual host dengan `/etc/hosts` dan `curl`

---

## 📘 Konsep Dasar

**Apache HTTP Server** adalah web server paling populer di Linux. Mendukung **virtual host** — memungkinkan satu server melayani banyak domain.

| Konsep | Fungsi |
|---|---|
| **DocumentRoot** | Direktori tempat file web disimpan |
| **VirtualHost** | Konfigurasi per domain |
| **a2ensite / a2dissite** | Enable/disable site config |
| **a2enmod / a2dismod** | Enable/disable module |
| **sites-available** | Config site yang tersedia |
| **sites-enabled** | Config site yang aktif (symlink) |

> **Analogi:** Virtual host seperti "pintu berbeda" di satu gedung. Meski alamatnya sama, isinya bisa berbeda tergantung pintu mana yang dimasuki.

**Struktur direktori Apache (Debian/Ubuntu):**

```
/etc/apache2/
├── apache2.conf          # config utama
├── ports.conf            # port listening
├── sites-available/      # config site (belum aktif)
├── sites-enabled/        # config site aktif (symlink)
├── mods-available/       # module tersedia
├── mods-enabled/         # module aktif
├── conf-available/       # config tambahan
└── conf-enabled/         # config tambahan aktif
```

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-015-2
```

> **Catatan:** Ganti `USERNAME` dengan username Adinusa masing-masing.

---

## 📘 Guided Example

### 1. Install Apache2

```bash
$ sudo apt-get install apache2
```

Konfirmasi instalasi sukses:

```bash
$ sudo systemctl status apache2
```

### 2. Buat document root dan homepage

```bash
$ sudo mkdir /var/www/USERNAME-adinusa.id
$ sudo vim /var/www/USERNAME-adinusa.id/index.html
```

Isi `index.html` — hanya teks ini, **tanpa tag HTML**:

```
Hello Adinusa, USERNAME is here!
```

### 3. Masuk ke direktori sites-available

```bash
$ cd /etc/apache2/sites-available/
```

### 4. Buat file virtual host

```bash
$ sudo vim USERNAME-adinusa.id.conf
```

Isi file:

```apache
<VirtualHost *:80>
        ServerName www.USERNAME-adinusa.id
        ServerAlias USERNAME-adinusa.id
        DocumentRoot /var/www/USERNAME-adinusa.id

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

**Penjelasan direktif:**

| Direktif | Fungsi |
|---|---|
| `VirtualHost *:80` | Berlaku untuk semua IP, port 80 |
| `ServerName` | Nama domain utama |
| `ServerAlias` | Nama domain alternatif |
| `DocumentRoot` | Lokasi file web |
| `ErrorLog` | File log error |
| `CustomLog` | File log akses |

### 5. Enable virtual host

```bash
$ sudo a2ensite USERNAME-adinusa.id.conf
```

Perintah ini membuat **symlink** dari `sites-available/` ke `sites-enabled/`.

### 6. Reload & restart Apache

```bash
$ sudo systemctl reload apache2
$ sudo systemctl restart apache2
```

- `reload` → baca ulang konfigurasi tanpa matikan service.
- `restart` → matikan lalu nyalakan ulang.

### 7. Update `/etc/hosts`

```bash
$ sudo vim /etc/hosts
```

Tambahkan `USERNAME-adinusa.id` ke baris localhost:

```
127.0.0.1 localhost USERNAME-adinusa.id
```

**Penjelasan:** Ini membuat sistem resolve `USERNAME-adinusa.id` ke `127.0.0.1` (localhost) — supaya domain bisa diakses secara lokal tanpa DNS publik.

### 8. Verifikasi

**Opsi 1 — curl:**

```bash
$ curl http://USERNAME-adinusa.id
Hello Adinusa, USERNAME is here!
```

**Opsi 2 — browser:**

Buka `http://USERNAME-adinusa.id` — harus muncul teks yang sama.

---

## 🧪 Practice Task

### Soal

1. Buat document root baru `/var/www/test-domain.local`.
2. Buat `index.html` dengan isi `Selamat datang di test-domain.local`.
3. Buat virtual host `test-domain.local.conf` di `sites-available/`.
4. Enable site, reload Apache.
5. Tambahkan `test-domain.local` ke `/etc/hosts`.
6. Verifikasi dengan `curl`.

### ✅ Solusi

```bash
# 1. Document root + homepage
$ sudo mkdir /var/www/test-domain.local
$ sudo vim /var/www/test-domain.local/index.html
# Isi: Selamat datang di test-domain.local

# 2. Buat vhost
$ sudo vim /etc/apache2/sites-available/test-domain.local.conf
```

Isi:

```apache
<VirtualHost *:80>
        ServerName test-domain.local
        DocumentRoot /var/www/test-domain.local

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

```bash
# 3. Enable + reload
$ sudo a2ensite test-domain.local.conf
$ sudo systemctl reload apache2

# 4. Update /etc/hosts
$ sudo vim /etc/hosts
# Tambah: 127.0.0.1 test-domain.local

# 5. Verifikasi
$ curl http://test-domain.local
Selamat datang di test-domain.local
```

### 🔍 Hasil yang Diharapkan

`curl http://test-domain.local` menampilkan teks homepage.

---

## 🐛 Troubleshooting & Kesalahan

- **`curl: (7) Failed to connect`:** Apache tidak jalan. Cek dengan `sudo systemctl status apache2`. Nyalakan dengan `sudo systemctl start apache2`.

- **Muncul halaman default Apache (bukan homepage kita):** Virtual host tidak aktif atau `/etc/hosts` belum diupdate. Cek dengan `sudo a2query -s` (lihat site aktif) dan `cat /etc/hosts`.

- **`403 Forbidden`:** Permission document root salah. Fix:
  ```bash
  sudo chown -R www-data:www-data /var/www/USERNAME-adinusa.id
  sudo chmod -R 755 /var/www/USERNAME-adinusa.id
  ```

- **`404 Not Found`:** File `index.html` tidak ada atau salah nama. Cek dengan `ls /var/www/USERNAME-adinusa.id/`.

- **Site tidak aktif meskipun sudah `a2ensite`:** Cek symlink di `sites-enabled/`:
  ```bash
  ls -l /etc/apache2/sites-enabled/
  ```

- **Konflik dengan default site:** Site `000-default.conf` aktif di port 80. Bisa dimatikan dengan `sudo a2dissite 000-default.conf`.

- **Syntax error konfigurasi:** Cek dengan:
  ```bash
  sudo apache2ctl configtest
  ```
  Output harus `Syntax OK`. Kalau ada error, perbaiki dulu sebelum reload.

- **`/etc/hosts` tidak berubah efeknya:** Cek format — harus `127.0.0.1 localhost USERNAME-adinusa.id`. Ada spasi sebagai pemisah, bukan koma.

- **`a2ensite` error "site does not exist":** Nama file harus **persis** dengan yang di `sites-available/`. Cek dengan `ls /etc/apache2/sites-available/`.

- **Index.html ada tag HTML:** Lab minta teks **polos tanpa tag**. Kalau pakai `<html>` atau `<body>`, browser tetap render tapi isi mentah bisa muncul. Ikuti instruksi: teks saja.

- **Lupa `sudo` saat edit `/etc/hosts`:** Tidak akan bisa disave. Wajib `sudo vim`.

- **Reload tidak cukup:** Kadang perlu `restart`. Coba `sudo systemctl restart apache2` kalau `reload` tidak bekerja.

---

## 💡 Catatan & Insight Pribadi

### Kenapa Butuh Virtual Host?

Satu server fisik bisa melayani **banyak domain** — inilah kekuatan virtual host. Contoh:
- `blog.example.com` → `/var/www/blog`
- `shop.example.com` → `/var/www/shop`
- `api.example.com` → `/var/www/api`

Tanpa virtual host, semua domain akan tampil isi yang sama.

### Alur Konfigurasi Apache

```
1. Install apache2
2. Buat document root + index.html
3. Buat config vhost di sites-available/
4. Enable dengan a2ensite (buat symlink ke sites-enabled/)
5. Reload/restart Apache
6. Tambah domain ke /etc/hosts (untuk local testing)
7. Verifikasi dengan curl
```

### `a2ensite` vs Edit Manual

| Cara | Kelebihan | Kekurangan |
|---|---|---|
| `a2ensite` | Otomatis buat symlink | Butuh nama file tepat |
| Manual `ln -s` | Kontrol penuh | Rawan typo |

**Rekomendasi:** Pakai `a2ensite` — lebih rapi dan ada validasi.

### `/etc/hosts` — DNS Lokal

`/etc/hosts` dipakai untuk resolve hostname **tanpa DNS server**. Cocok untuk:
- Testing lokal sebelum beli domain.
- Blokir domain (redirect ke `0.0.0.0`).
- Dev environment dengan domain custom.

**Format:**
```
<IP>    <hostname>  [alias1] [alias2] ...
```

### Reload vs Restart

| Perintah | Efek | Kapan pakai |
|---|---|---|
| `reload` | Baca ulang config tanpa matikan service | Setelah edit vhost |
| `restart` | Matikan lalu nyalakan | Setelah install/upgrade |

`reload` lebih halus — user yang sedang akses tidak terputus. Tapi kadang `restart` perlu untuk perubahan yang lebih besar.

### Struktur File yang Wajib Diketahui

```
/var/www/<domain>/           ← file web
/etc/apache2/sites-available/  ← config vhost (draft)
/etc/apache2/sites-enabled/    ← symlink config aktif
/var/log/apache2/              ← log (error.log, access.log)
```

### Cek Site Aktif

```bash
$ sudo a2query -s              # daftar site aktif
$ sudo apache2ctl -S           # ringkasan vhost & port
```

### Permission File Web

Default Apache berjalan sebagai user `www-data`. File web harus:
- **Owner:** `www-data` atau user dengan akses baca.
- **Permission:** minimal `644` untuk file, `755` untuk direktori.

```bash
$ sudo chown -R www-data:www-data /var/www/<domain>/
$ sudo chmod -R 755 /var/www/<domain>/
```

### Kapan Pakai Virtual Host?

| Skenario | Pakai? |
|---|---|
| Satu domain, satu aplikasi | Tidak perlu |
| Multiple domain di satu server | ✅ Ya |
| Testing lokal | ✅ Ya |
| Reverse proxy | ✅ Ya |
| Load balancing | ✅ Ya (dengan modul tambahan) |

### Kasus Nyata

- **Server multi-tenant:** Satu server, banyak klien dengan domain berbeda.
- **Development:** Developer testing banyak project dengan domain lokal (`myapp.local`).
- **Subdomain:** `blog.example.com`, `shop.example.com`, `api.example.com`.
- **Redirect/Reverse Proxy:** Apache di depan aplikasi Node.js/Python.

### Pelajaran Kunci dari Lab Ini

1. **Virtual host** memungkinkan satu server melayani banyak domain.
2. **`a2ensite`** membuat symlink dari `sites-available/` ke `sites-enabled/`.
3. **`/etc/hosts`** untuk testing local tanpa DNS.
4. **`reload`** lebih halus dari `restart` — pakai kalau bisa.
5. **Cek dengan `apache2ctl configtest`** sebelum reload.
6. **Permission `www-data:www-data`** untuk file web.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `sudo apt-get install apache2` | Install Apache |
| `sudo systemctl status apache2` | Cek status Apache |
| `sudo systemctl reload apache2` | Reload konfigurasi |
| `sudo systemctl restart apache2` | Restart service |
| `sudo a2ensite <site>.conf` | Aktifkan site |
| `sudo a2dissite <site>.conf` | Nonaktifkan site |
| `sudo a2query -s` | Daftar site aktif |
| `sudo apache2ctl configtest` | Cek syntax config |
| `sudo apache2ctl -S` | Ringkasan vhost |
| `curl http://<domain>` | Verifikasi web |

**File penting:**

| File | Isi |
|---|---|
| `/etc/apache2/sites-available/<site>.conf` | Config vhost (draft) |
| `/etc/apache2/sites-enabled/<site>.conf` | Symlink config aktif |
| `/var/www/<domain>/index.html` | File homepage |
| `/etc/hosts` | DNS lokal |
| `/var/log/apache2/error.log` | Log error |
| `/var/log/apache2/access.log` | Log akses |

**Contoh config virtual host:**

```apache
<VirtualHost *:80>
        ServerName www.<domain>
        ServerAlias <domain>
        DocumentRoot /var/www/<domain>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

---

## 📌 Kesimpulan

Lab ini memperkenalkan **Apache web server** dan **virtual host** — keterampilan inti sysadmin/web admin. Kemampuan mengonfigurasi virtual host memungkinkan satu server melayani banyak domain, yang sangat berguna di production. Yang paling penting dipahami:

- Struktur direktori Apache (`sites-available` vs `sites-enabled`).
- Cara kerja `a2ensite` (symlink).
- Peran `/etc/hosts` sebagai DNS lokal untuk testing.
- Verifikasi dengan `curl` dan `apache2ctl configtest`.

Kemampuan ini akan terus dipakai saat mengelola server web, deploy aplikasi, atau menyiapkan environment development lokal.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
