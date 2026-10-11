# Lab 16.1 — Configure HTTPS Using Self-Signed Certificate

**Course:** Linux System Administration (Adinusa)
**Topic:** HTTPS dengan self-signed certificate di Apache
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Mengaktifkan **modul SSL** di Apache
- Membuat **self-signed certificate** secara non-interaktif dengan `openssl`
- Mengonfigurasi **virtual host HTTPS** di Apache
- Memverifikasi akses HTTPS dengan `curl` dan browser

---

## 📘 Konsep Dasar

**HTTPS** = HTTP + TLS/SSL. Data antara client dan server dienkripsi. Untuk mengaktifkan HTTPS, server butuh **certificate**.

| Jenis Certificate | Cocok untuk |
|---|---|
| **Self-signed** | Testing, development, internal |
| **Let's Encrypt** | Produksi (gratis, otomatis) |
| **Commercial CA** | Produksi (berbayar, terpercaya) |

> **Self-signed** ditandatangani oleh diri sendiri, jadi browser akan menampilkan warning *"Not Secure"*. Untuk testing, ini cukup — cukup klik "Advanced → Proceed".

**Komponen SSL di Apache:**

| Komponen | Fungsi |
|---|---|
| `mod_ssl` | Modul Apache untuk SSL |
| `SSLCertificateFile` | File certificate publik (`.crt`) |
| `SSLCertificateKeyFile` | File private key (`.key`) |
| `SSLEngine on` | Aktifkan SSL di vhost |

**Port HTTPS:** `443` (HTTP: `80`).

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-016-1
```

> **Catatan:** Ganti `USERNAME` dengan username Adinusa masing-masing.

---

## 📘 Guided Example

### 1. Install Apache SSL support & enable module

```bash
$ sudo apt-get update
$ sudo apt-get install apache2 openssl ssl-cert -y
$ sudo a2enmod ssl
```

**Penjelasan:**
- `openssl` → tool untuk generate certificate.
- `ssl-cert` → paket Debian untuk SSL.
- `a2enmod ssl` → aktifkan modul `mod_ssl` di Apache.

### 2. Buat direktori untuk certificate

```bash
$ sudo mkdir /etc/apache2/ssl
```

### 3. Generate self-signed certificate (non-interactive)

```bash
$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/apache2/ssl/USERNAME-adinusa.id.key \
  -out /etc/apache2/ssl/USERNAME-adinusa.id.crt \
  -subj "/C=ID/ST=Jawa Barat/L=Bogor/O=Personal/OU=Personal/CN=USERNAME-adinusa.id/emailAddress=USERNAME@email.local"
```

**Penjelasan opsi:**

| Opsi | Fungsi |
|---|---|
| `req` | Mulai proses CSR |
| `-x509` | Output self-signed certificate (bukan CSR) |
| `-nodes` | Private key **tidak dienkripsi** dengan passphrase |
| `-days 365` | Certificate valid 365 hari |
| `-newkey rsa:2048` | Generate RSA private key 2048-bit |
| `-keyout` | Path simpan private key |
| `-out` | Path simpan certificate |
| `-subj` | Isi subject secara **non-interaktif** |

> **Kenapa `-nodes`?** Supaya private key tidak butuh passphrase setiap kali Apache restart. Ini praktis untuk server, tapi kalau key-nya dicuri, attacker bisa langsung pakai tanpa passphrase.

### 4. Cek certificate dan key

```bash
$ sudo ls -l /etc/apache2/ssl/
-rw-r--r-- 1 root root ... USERNAME-adinusa.id.crt
-rw------- 1 root root ... USERNAME-adinusa.id.key
```

**Perhatikan:** private key (`.key`) punya permission `600` — hanya root yang bisa baca. Certificate (`.crt`) boleh dibaca siapa saja.

### 5. Buat HTTPS virtual host config

```bash
$ sudo vim /etc/apache2/sites-available/USERNAME-adinusa.id-ssl.conf
```

Isi:

```apache
<IfModule mod_ssl.c>
  <VirtualHost *:443>
    ServerName USERNAME-adinusa.id
    ServerAlias www.USERNAME-adinusa.id
    DocumentRoot /var/www/USERNAME-adinusa.id

    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/USERNAME-adinusa.id.crt
    SSLCertificateKeyFile /etc/apache2/ssl/USERNAME-adinusa.id.key

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
</IfModule>
```

**Penjelasan direktif:**

| Direktif | Fungsi |
|---|---|
| `<IfModule mod_ssl.c>` | Hanya berlaku kalau modul SSL aktif |
| `<VirtualHost *:443>` | Vhost untuk port 443 (HTTPS) |
| `SSLEngine on` | Aktifkan SSL di vhost ini |
| `SSLCertificateFile` | File certificate publik |
| `SSLCertificateKeyFile` | File private key |

> **Pastikan document root-nya sama** dengan vhost HTTP (Lab 15.2). Jadi domain yang sama bisa diakses via HTTP dan HTTPS.

### 6. Enable site SSL & restart Apache

```bash
$ sudo a2ensite USERNAME-adinusa.id-ssl.conf
$ sudo systemctl reload apache2
$ sudo systemctl restart apache2
```

### 7. Update `/etc/hosts`

```bash
$ sudo vim /etc/hosts
```

Tambah:

```
127.0.0.1   USERNAME-adinusa.id   www.USERNAME-adinusa.id
```

### 8. Verifikasi HTTPS

**Opsi 1 — curl (ignore SSL warning):**

```bash
$ curl -vk https://USERNAME-adinusa.id
```

**Opsi 2 — browser:**

Buka `https://USERNAME-adinusa.id` — akan muncul warning certificate, klik **Advanced → Proceed**.

---

## 🧪 Practice Task

### Soal

1. Buat certificate baru dengan masa berlaku 730 hari untuk domain `test.local`.
2. Buat vhost HTTPS untuk `test.local` dengan document root `/var/www/test.local`.
3. Enable site, reload Apache.
4. Tambah `test.local` ke `/etc/hosts`.
5. Verifikasi dengan `curl -vk`.

### ✅ Solusi

```bash
# 1. Generate certificate (730 hari)
$ sudo openssl req -x509 -nodes -days 730 -newkey rsa:2048 \
  -keyout /etc/apache2/ssl/test.local.key \
  -out /etc/apache2/ssl/test.local.crt \
  -subj "/C=ID/ST=Jawa Barat/L=Bogor/O=Personal/CN=test.local"

# 2. Buat vhost
$ sudo vim /etc/apache2/sites-available/test.local-ssl.conf
```

Isi:

```apache
<IfModule mod_ssl.c>
  <VirtualHost *:443>
    ServerName test.local
    DocumentRoot /var/www/test.local

    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/test.local.crt
    SSLCertificateKeyFile /etc/apache2/ssl/test.local.key

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
</IfModule>
```

```bash
# 3. Enable + reload
$ sudo a2ensite test.local-ssl.conf
$ sudo systemctl reload apache2
$ sudo systemctl restart apache2

# 4. Update /etc/hosts
$ sudo vim /etc/hosts
# Tambah: 127.0.0.1 test.local

# 5. Verifikasi
$ curl -vk https://test.local
```

### 🔍 Hasil yang Diharapkan

- `curl -vk` menampilkan HTML dari document root.
- Warning SSL muncul karena self-signed, tapi koneksi tetap berhasil.
- `-v` menampilkan info SSL handshake, `-k` mengabaikan verifikasi certificate.

---

## 🐛 Troubleshooting & Kesalahan

- **`SSL_ERROR_RX_RECORD_TOO_LONG` atau "SSL handshake failed":** Biasanya karena `mod_ssl` belum aktif atau config vhost salah. Cek:
  ```bash
  sudo a2enmod ssl
  sudo apache2ctl configtest
  ```

- **Apache gagal start setelah enable SSL site:** Cek log error:
  ```bash
  sudo tail -20 /var/log/apache2/error.log
  ```
  Penyebab umum: path certificate salah, atau file key tidak bisa dibaca.

- **`curl: (60) SSL certificate problem`:** Normal untuk self-signed. Gunakan `-k` untuk ignore:
  ```bash
  curl -k https://USERNAME-adinusa.id
  ```

- **Browser warning "Not Secure":** Normal untuk self-signed. Klik **Advanced → Proceed** untuk melanjutkan. Untuk menghilangkan warning, perlu certificate dari CA resmi (misalnya Let's Encrypt).

- **Port 443 tidak bisa diakses:** Cek Apache listening di port 443:
  ```bash
  sudo ss -tlnp | grep :443
  ```
  Kalau tidak muncul, cek `/etc/apache2/ports.conf` — pastikan ada `Listen 443`.

- **Certificate expired:** Cek tanggal expiry:
  ```bash
  sudo openssl x509 -in /etc/apache2/ssl/USERNAME-adinusa.id.crt -noout -dates
  ```

- **Private key tidak bisa dibaca Apache:** Apache berjalan sebagai `www-data`, jadi kadang perlu permission khusus. Cek:
  ```bash
  sudo ls -l /etc/apache2/ssl/USERNAME-adinusa.id.key
  ```
  Harus `600` dan owner `root`. Kalau owner bukan root, `chown root:root`.

- **`apache2ctl configtest` error "SSLCertificateFile: file does not exist":** Path salah atau typo. Cek dengan `ls -l` path yang ditulis di config.

- **Salah Subject (`-subj`) format:** Harus pakai `/` sebagai pemisah, bukan spasi. Contoh: `/C=ID/ST=Jawa Barat/L=Bogor/CN=domain.local`.

- **Certificate tidak match domain:** `CN` di `-subj` harus sama dengan `ServerName` di vhost. Kalau beda, browser akan warning "certificate not valid for this domain".

- **Lupa `-nodes`:** Private key akan dienkripsi dan Apache akan minta passphrase saat start. Di server, ini bikin Apache tidak bisa restart otomatis. Selalu pakai `-nodes` untuk server.

- **Redirect HTTP → HTTPS belum jalan:** Lab ini hanya setup HTTPS. Kalau mau redirect otomatis, perlu `Redirect permanent / https://domain/` atau `RewriteEngine` di vhost HTTP.

---

## 💡 Catatan & Insight Pribadi

### Self-Signed vs CA-Signed

| Aspek | Self-Signed | CA-Signed |
|---|---|---|
| Biaya | Gratis | Gratis (Let's Encrypt) / Berbayar |
| Setup | Manual, cepat | Otomatis (Certbot) |
| Browser warning | Muncul | Tidak |
| Cocok untuk | Testing, internal | Produksi publik |

> **Untuk produksi:** Pakai **Let's Encrypt** dengan Certbot — gratis dan otomatis. Self-signed hanya untuk testing.

### Anatomi `openssl req` Command

```
openssl req  -x509  -nodes  -days 365  -newkey rsa:2048
     │       │       │         │            │
     │       │       │         │            └── key 2048-bit
     │       │       │         └─────────────── valid 1 tahun
     │       │       └───────────────────────── key tanpa passphrase
     │       └───────────────────────────────── self-signed (bukan CSR)
     └───────────────────────────────────────── perintah dasar
```

### Apa Itu `-subj`?

Singkatan dari **subject**. Berisi informasi identitas certificate:

| Field | Arti | Contoh |
|---|---|---|
| `C` | Country | `ID` |
| `ST` | State | `Jawa Barat` |
| `L` | Locality | `Bogor` |
| `O` | Organization | `Personal` |
| `OU` | Organizational Unit | `Personal` |
| `CN` | Common Name | `USERNAME-adinusa.id` |
| `emailAddress` | Email | `USERNAME@email.local` |

> **`CN` adalah field paling penting** — harus sama dengan domain yang diakses, kalau tidak browser akan warning.

### Struktur File SSL

```
/etc/apache2/
├── ssl/
│   ├── USERNAME-adinusa.id.crt    ← certificate (publik)
│   └── USERNAME-adinusa.id.key    ← private key (rahasia, 600)
├── sites-available/
│   └── USERNAME-adinusa.id-ssl.conf
└── sites-enabled/
    └── USERNAME-adinusa.id-ssl.conf → symlink
```

### Vhost HTTP vs HTTPS

| Aspek | HTTP | HTTPS |
|---|---|---|
| Port | 80 | 443 |
| Config file | `USERNAME-adinusa.id.conf` | `USERNAME-adinusa.id-ssl.conf` |
| SSL directives | Tidak ada | `SSLEngine`, `SSLCertificateFile`, dll |
| Block | `<VirtualHost *:80>` | `<VirtualHost *:443>` |

Bisa punya **kedua vhost** untuk domain yang sama — HTTP dan HTTPS berjalan bersamaan.

### Cek Info Certificate

```bash
# Lihat subject, issuer, tanggal expiry
$ sudo openssl x509 -in /etc/apache2/ssl/USERNAME-adinusa.id.crt -noout -text

# Tanggal saja
$ sudo openssl x509 -in /etc/apache2/ssl/USERNAME-adinusa.id.crt -noout -dates

# Subject
$ sudo openssl x509 -in /etc/apache2/ssl/USERNAME-adinusa.id.crt -noout -subject

# Cek key cocok dengan cert
$ sudo openssl x509 -noout -modulus -in cert.crt | md5sum
$ sudo openssl rsa -noout -modulus -in cert.key | md5sum
# Hash harus sama
```

### Kapan Pakai Self-Signed?

| Skenario | Self-signed? |
|---|---|
| Testing local | ✅ Ya |
| Dev environment | ✅ Ya |
| Server internal (intranet) | ⚠️ Bisa, kalau semua user tahu warning |
| Website publik | ❌ Tidak — pakai Let's Encrypt |
| API publik | ❌ Tidak — butuh cert tepercaya |
| E-commerce | ❌ Tidak — client butuh trust |

### Redirect HTTP → HTTPS (Opsional)

Kalau mau semua akses HTTP otomatis redirect ke HTTPS, tambahkan di vhost HTTP (`USERNAME-adinusa.id.conf`):

```apache
<VirtualHost *:80>
    ServerName USERNAME-adinusa.id
    Redirect permanent / https://USERNAME-adinusa.id/
</VirtualHost>
```

Atau pakai `RewriteEngine`:

```apache
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
```

Perlu `a2enmod rewrite`.

### Best Practice SSL

1. **Jangan pakai `-nodes` untuk key produksi** — enkripsi dengan passphrase, atau simpan aman.
2. **Backup private key** — kalau hilang, harus generate ulang.
3. **Rotasi certificate** sebelum expired (`-days 90` untuk produksi).
4. **Monitor expiry** — bisa pakai cron + script.
5. **Gunakan CA resmi untuk produksi** — Let's Encrypt gratis dan otomatis.
6. **Test dengan `apache2ctl configtest`** sebelum reload.

### Kasus Nyata

- **Server internal:** Self-signed untuk dashboard admin yang hanya diakses karyawan.
- **Development:** Self-signed untuk testing aplikasi web dengan HTTPS.
- **Staging:** Sama seperti produksi, tapi certificate berbeda.
- **Produksi:** Let's Encrypt + Certbot untuk auto-renewal setiap 90 hari.

### Pelajaran Kunci dari Lab Ini

1. **`mod_ssl`** harus di-enable (`a2enmod ssl`).
2. **`openssl req -x509 -nodes`** untuk self-signed non-interaktif.
3. **`-subj`** memungkinkan generate tanpa prompt.
4. **Private key permission `600`**, owner root.
5. **Vhost HTTPS** pakai port `443` dan direktif `SSL*`.
6. **`curl -k`** untuk ignore warning self-signed.
7. **Self-signed = testing**, bukan produksi.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `sudo a2enmod ssl` | Aktifkan modul SSL |
| `sudo a2enmod rewrite` | Aktifkan modul rewrite (untuk redirect) |
| `sudo mkdir /etc/apache2/ssl` | Buat direktori certificate |
| `sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 ...` | Generate self-signed cert |
| `sudo openssl x509 -in cert.crt -noout -text` | Lihat info certificate |
| `sudo openssl x509 -in cert.crt -noout -dates` | Lihat tanggal expiry |
| `sudo a2ensite <site>-ssl.conf` | Aktifkan vhost SSL |
| `sudo systemctl reload apache2` | Reload Apache |
| `curl -k https://<domain>` | Verifikasi HTTPS |
| `curl -vk https://<domain>` | Verifikasi + info SSL handshake |
| `sudo apache2ctl configtest` | Cek syntax config |

**File penting:**

| File | Isi |
|---|---|
| `/etc/apache2/mods-enabled/ssl.load` | Symlink modul SSL aktif |
| `/etc/apache2/ssl/*.crt` | Certificate publik |
| `/etc/apache2/ssl/*.key` | Private key (600) |
| `/etc/apache2/sites-available/*-ssl.conf` | Config vhost HTTPS |
| `/etc/apache2/ports.conf` | Port listening (`Listen 443`) |
| `/var/log/apache2/error.log` | Log error Apache |

**Contoh config vhost HTTPS:**

```apache
<IfModule mod_ssl.c>
  <VirtualHost *:443>
    ServerName <domain>
    ServerAlias www.<domain>
    DocumentRoot /var/www/<domain>

    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/<domain>.crt
    SSLCertificateKeyFile /etc/apache2/ssl/<domain>.key

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
</IfModule>
```

---

## 📌 Kesimpulan

Lab ini memperkenalkan **HTTPS di Apache** dengan **self-signed certificate** — langkah pertama sebelum masuk ke certificate production-grade. Kemampuan yang didapat:

- Mengaktifkan **modul SSL** di Apache.
- Generate certificate dengan **`openssl req` non-interaktif** (`-subj`).
- Mengonfigurasi **vhost HTTPS** dengan direktif SSL.
- Memverifikasi dengan `curl -k`.

Yang paling penting dipahami: **self-signed hanya untuk testing**. Untuk produksi, wajib pakai **Let's Encrypt** atau CA resmi. Konsep SSL handshake, private key vs public key, dan **port 443** adalah fondasi untuk semua topik keamanan web ke depannya.

Kemampuan ini sangat relevan di dunia kerja — hampir semua server web modern wajib HTTPS. Baik untuk dashboard internal, API, atau e-commerce, konfigurasi SSL adalah keterampilan wajib sysadmin.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
