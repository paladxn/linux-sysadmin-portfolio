# Lab 16.2 — Installing phpMyAdmin with Firewall Protection

**Course:** Linux System Administration (Adinusa)
**Topic:** phpMyAdmin + MySQL + PHP + Firewalld
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Menginstall **web server (Apache)**, **database (MySQL)**, dan **PHP**
- Menginstall dan mengonfigurasi **phpMyAdmin**
- Membuat **virtual host** untuk phpMyAdmin di port custom
- Mengonfigurasi **firewalld** untuk membatasi akses port

---

## 📘 Konsep Dasar

**phpMyAdmin** = aplikasi web untuk mengelola database MySQL/MariaDB lewat browser. Tidak perlu ketik perintah SQL di terminal.

**Stack yang dibutuhkan:**

| Komponen | Fungsi |
|---|---|
| **Apache** | Web server |
| **MySQL/MariaDB** | Database server |
| **PHP** | Bahasa pemrograman web |
| **phpMyAdmin** | Aplikasi web untuk MySQL |
| **Firewalld** | Firewall dinamis |

**Kenapa pakai port custom (8088)?**
- Port default phpMyAdmin di Apache biasanya 80, tapi itu sudah dipakai web utama.
- Port custom memudahkan pemisahan akses.
- Bisa dibatasi oleh firewall — hanya port tertentu yang dibuka.

**Kenapa pakai firewalld?**
- Firewalld = firewall modern dengan konsep **zone**.
- Bisa atur per-interface: mana interface yang di-protect, mana yang bebas.
- Interface `enp0s8` (host-only) dipakai untuk akses phpMyAdmin dari host.

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-016-2
```

> **Catatan:** Ganti `USERNAME` dengan username Adinusa masing-masing.

---

## 📘 Guided Example

### 1. Update system packages

```bash
$ sudo apt-get update -y
$ sudo apt-get upgrade -y
```

### 2. Install web server (Apache)

```bash
$ sudo apt install apache2 -y
$ sudo systemctl enable --now apache2
```

**Verifikasi:**
```bash
$ sudo systemctl status apache2
```

### 3. Install MySQL

```bash
$ sudo apt install mysql-server -y
$ sudo systemctl enable --now mysql
```

**Verifikasi:**
```bash
$ sudo systemctl status mysql
```

### 4. Install PHP dan ekstensi

```bash
$ sudo apt install php libapache2-mod-php php-mysql -y
$ sudo apt install php-{mbstring,zip,gd,curl} -y
```

**Penjelasan paket:**

| Paket | Fungsi |
|---|---|
| `php` | Bahasa pemrograman |
| `libapache2-mod-php` | Integrasi PHP dengan Apache |
| `php-mysql` | Driver PHP untuk MySQL |
| `php-mbstring` | Multi-byte string (untuk karakter Unicode) |
| `php-zip` | Kompresi ZIP |
| `php-gd` | Manipulasi gambar |
| `php-curl` | HTTP request |

### 5. Install phpMyAdmin

```bash
$ sudo apt install phpmyadmin -y
```

**Prompt yang muncul — pilih:**

| Prompt | Pilih |
|---|---|
| Web server to reconfigure | **apache2** (spasi untuk pilih, Enter untuk OK) |
| Configure database for phpmyadmin with dbconfig-common | **Yes** |
| MySQL application password for phpmyadmin | Isi password (ingat!) |
| Confirm password | Ulangi password |

### 6. Aktifkan ekstensi mbstring

```bash
$ sudo phpenmod mbstring
$ sudo systemctl restart apache2
```

### 7. Buat MySQL user

```bash
$ sudo mysql
```

Di dalam prompt `mysql>`:

```sql
-- Buat user untuk host '%' (remote) DAN 'localhost' (lokal)
CREATE USER 'USERNAME'@'%' IDENTIFIED WITH mysql_native_password BY 'adinusa';
CREATE USER 'USERNAME'@'localhost' IDENTIFIED WITH mysql_native_password BY 'adinusa';

-- Grant privilege
GRANT ALL PRIVILEGES ON *.* TO 'USERNAME'@'%' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'USERNAME'@'localhost' WITH GRANT OPTION;

-- Refresh
FLUSH PRIVILEGES;

-- Verifikasi
SELECT user, host, plugin FROM mysql.user WHERE user='USERNAME';

EXIT;
```

**Penjelasan:**
- `'%'` → user bisa login dari mana saja (remote).
- `'localhost'` → user bisa login dari VM yang sama (untuk phpMyAdmin).
- `mysql_native_password` → plugin kompatibel dengan phpMyAdmin.
- `WITH GRANT OPTION` → bisa memberikan privilege ke user lain.

> **Penting:** User `@'%'` dan `@'localhost'` adalah **dua user berbeda** di MySQL. Untuk phpMyAdmin yang berjalan di VM yang sama, harus ada user `@'localhost'` juga.

### 8. Buat virtual host untuk phpMyAdmin

```bash
$ sudo nano /etc/apache2/sites-available/phpmyadmin.conf
```

Isi file:

```apache
<VirtualHost *:8088>
    ServerAdmin admin@mydomain.local
    ServerName phpmyadmin.mydomain.local
    DocumentRoot /usr/share/phpmyadmin

    <Directory /usr/share/phpmyadmin>
        Options SymLinksIfOwnerMatch
        DirectoryIndex index.php

        # limit libapache2-mod-php to files and directories necessary by pma
        <IfModule mod_php7.c>
            php_admin_value upload_tmp_dir /var/lib/phpmyadmin/tmp
            php_admin_value open_basedir /usr/share/phpmyadmin/:/usr/share/doc/phpmyadmin/:/etc/phpmyadmin/:/var/lib/phpmyadmin/:/usr/share/php/:/usr/share/javascript/
        </IfModule>

        # PHP 8+
        <IfModule mod_php.c>
            php_admin_value upload_tmp_dir /var/lib/phpmyadmin/tmp
            php_admin_value open_basedir /usr/share/phpmyadmin/:/usr/share/doc/phpmyadmin/:/etc/phpmyadmin/:/var/lib/phpmyadmin/:/usr/share/php/:/usr/share/javascript/
        </IfModule>
    </Directory>

    # Disallow web access to directories that don't need it
    <Directory /usr/share/phpmyadmin/templates>
        Require all denied
    </Directory>
    <Directory /usr/share/phpmyadmin/libraries>
        Require all denied
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/phpmyadmin_error.log
    CustomLog ${APACHE_LOG_DIR}/phpmyadmin_access.log combined
</VirtualHost>
```

**Penjelasan direktif penting:**

| Direktif | Fungsi |
|---|---|
| `<VirtualHost *:8088>` | Vhost di port 8088 |
| `ServerName phpmyadmin.mydomain.local` | Nama domain |
| `DocumentRoot /usr/share/phpmyadmin` | Lokasi file phpMyAdmin |
| `Require all denied` | Blokir akses ke direktori internal |

### 9. Tambahkan port 8088 di `ports.conf`

```bash
$ sudo nano /etc/apache2/ports.conf
```

Ubah menjadi:

```apache
Listen 80
Listen 8088

<IfModule ssl_module>
    Listen 443
</IfModule>

<IfModule mod_gnutls.c>
    Listen 443
</IfModule>
```

**Penting:** Apache harus **listen** di port 8088 sebelum vhost bisa diakses.

### 10. Enable site & reload Apache

```bash
$ sudo a2ensite phpmyadmin
$ sudo systemctl reload apache2
```

### 11. Install firewalld

```bash
$ sudo apt install firewalld -y
```

### 12. Konfigurasi firewalld

**Tambahkan interface `enp0s8` ke firewalld:**

```bash
$ sudo firewall-cmd --add-interface=enp0s8 --permanent
```

**Buka port 8088 di zone public:**

```bash
$ sudo firewall-cmd --add-port=8088/tcp --zone=public --permanent
```

**Reload firewalld:**

```bash
$ sudo firewall-cmd --reload
```

**Verifikasi:**

```bash
$ sudo firewall-cmd --list-all --zone=public
```

Output harus menampilkan:
```
public (active)
  target: default
  interfaces: enp0s8
  ports: 8088/tcp
  ...
```

---

## 🔍 Verifikasi

### 1. Cek Apache listen di port 8088

```bash
$ sudo ss -tlnp | grep :8088
```

Harus muncul `LISTEN`.

### 2. Cek site aktif

```bash
$ sudo a2query -s
```

Harus ada `phpmyadmin`.

### 3. Akses dari browser

Dari Windows (host), buka browser:

```
http://172.17.10.10:8088
```

*(Ganti IP sesuai interface `enp0s8` VM-mu)*

Login dengan:
- Username: `USERNAME`
- Password: `adinusa`

### 4. Cek firewalld aktif

```bash
$ sudo firewall-cmd --state
running

$ sudo firewall-cmd --list-all --zone=public
```

---

## 🧪 Practice Task

### Soal

1. Akses phpMyAdmin dari browser host.
2. Login dengan user `USERNAME`.
3. Buat database baru bernama `latihan`.
4. Buat tabel sederhana di dalamnya.
5. Cek database via MySQL CLI di VM.

### ✅ Solusi

**Di browser:**
1. Buka `http://172.17.10.10:8088`
2. Login: `USERNAME` / `adinusa`
3. Klik **New** di sidebar kiri → beri nama `latihan` → Create
4. Pilih database `latihan` → tab SQL → paste:
   ```sql
   CREATE TABLE users (
     id INT PRIMARY KEY AUTO_INCREMENT,
     nama VARCHAR(50),
     email VARCHAR(100)
   );
   ```
5. Klik **Go**

**Verifikasi via CLI:**

```bash
$ sudo mysql
mysql> SHOW DATABASES;
mysql> USE latihan;
mysql> SHOW TABLES;
mysql> DESCRIBE users;
mysql> EXIT;
```

### 🔍 Hasil yang Diharapkan

- Database `latihan` muncul di `SHOW DATABASES`.
- Tabel `users` muncul di `SHOW TABLES`.
- Struktur tabel tampil di `DESCRIBE users`.

---

## 🐛 Troubleshooting & Kesalahan

### Masalah Instalasi MySQL

- **`mysql-common` dari MariaDB bentrok dengan `mysql-server-8.0`:**
  Ini masalah klasik ketika **MariaDB sudah terinstall duluan** (dari Lab 6.2). Output error:
  ```
  /var/lib/dpkg/info/mysql-server-8.0.postinst: line 202: 
  /usr/share/mysql-common/configure-symlinks: No such file or directory
  ```
  **Penyebab:** `mysql-common` yang terinstall berasal dari repo MariaDB (`1:10.11.19+maria~ubu2404`), bukan dari Ubuntu. File `configure-symlinks` **hanya ada** di paket `mysql-common` versi Oracle/Ubuntu. MariaDB dan MySQL **tidak bisa terinstall bersamaan**.

  **Solusi lengkap:**
  ```bash
  # 1. Hentikan MariaDB
  sudo systemctl stop mariadb
  sudo systemctl disable mariadb

  # 2. Purge semua paket MariaDB & MySQL
  sudo apt purge -y 'mariadb*' 'mysql*'
  sudo apt autoremove -y

  # 3. Hapus sisa direktori
  sudo rm -rf /etc/mysql /var/lib/mysql /var/log/mysql /usr/share/mysql-common

  # 4. Hapus repo MariaDB
  sudo rm -f /etc/apt/sources.list.d/mariadb.list
  sudo rm -f /etc/apt/sources.list.d/mariadb.sources

  # 5. Update & install ulang
  sudo apt update
  sudo apt install mysql-server -y

  # 6. Verifikasi
  sudo systemctl status mysql
  ```

- **APT selalu pilih `mysql-common` versi MariaDB:**
  Cek dengan `apt-cache policy mysql-common`. Kalau candidate masih `1:10.11.x+maria`, berarti repo MariaDB masih aktif. Cek `/etc/apt/sources.list.d/` — kalau ada `mariadb.list` atau `mariadb.sources`, hapus.

- **`NO_PUBKEY F1656F24C74CD1D8` dari repo MariaDB:**
  GPG key MariaDB expired/hilang. Solusi paling cepat: hapus repo MariaDB kalau tidak dipakai. Atau tambah key ulang:
  ```bash
  sudo curl -LsS https://mariadb.org/mariadb_release_signing_key.pgp -o /etc/apt/keyrings/mariadb-keyring.pgp
  ```

### Masalah Login phpMyAdmin

- **`Access denied for user 'USERNAME'@'localhost' (using password: YES)`:**
  Penyebab paling umum: **plugin `caching_sha2_password`** (default MySQL 8.0) tidak kompatibel dengan phpMyAdmin versi Ubuntu 24.04. Cek dengan:
  ```bash
  sudo mysql -e "SELECT user, host, plugin FROM mysql.user WHERE user='USERNAME';"
  ```
  Kalau plugin-nya `caching_sha2_password`, ubah ke `mysql_native_password`:
  ```sql
  ALTER USER 'USERNAME'@'localhost' IDENTIFIED WITH mysql_native_password BY 'adinusa';
  ALTER USER 'USERNAME'@'%' IDENTIFIED WITH mysql_native_password BY 'adinusa';
  FLUSH PRIVILEGES;
  ```

- **User `@'%'` tidak bisa login dari `localhost`:**
  MySQL memperlakukan `'USERNAME'@'%'` dan `'USERNAME'@'localhost'` sebagai **dua user berbeda**. User `@'%'` tidak otomatis bisa login dari `localhost`. Solusi: buat **keduanya**:
  ```sql
  CREATE USER 'USERNAME'@'localhost' IDENTIFIED WITH mysql_native_password BY 'adinusa';
  GRANT ALL PRIVILEGES ON *.* TO 'USERNAME'@'localhost' WITH GRANT OPTION;
  FLUSH PRIVILEGES;
  ```

- **Masih gagal login meskipun plugin sudah diubah:**
  Kemungkinan password tidak ter-set dengan benar. Solusi paling bersih — **drop & buat ulang user**:
  ```sql
  DROP USER IF EXISTS 'USERNAME'@'localhost';
  DROP USER IF EXISTS 'USERNAME'@'%';

  CREATE USER 'USERNAME'@'localhost' IDENTIFIED WITH mysql_native_password BY 'adinusa';
  CREATE USER 'USERNAME'@'%' IDENTIFIED WITH mysql_native_password BY 'adinusa';

  GRANT ALL PRIVILEGES ON *.* TO 'USERNAME'@'localhost' WITH GRANT OPTION;
  GRANT ALL PRIVILEGES ON *.* TO 'USERNAME'@'%' WITH GRANT OPTION;

  FLUSH PRIVILEGES;
  ```

- **Test login via CLI dulu sebelum phpMyAdmin:**
  ```bash
  mysql -u USERNAME -p
  # Masukkan: adinusa
  ```
  Kalau berhasil masuk `mysql>`, berarti user OK — masalahnya di phpMyAdmin. Kalau gagal, user masih bermasalah.

- **`Access denied for user 'root'@'localhost'`:**
  Di Ubuntu, root MySQL pakai `auth_socket` — harus lewat `sudo mysql`, bukan `mysql -u root -p`.

### Masalah Akses dari Windows

- **Ping dari Windows ke VM timeout:**
  Windows tidak punya IP di subnet yang sama dengan VM. Cek dengan `ipconfig` di Windows — cari adapter **VirtualBox Host-Only Network**. Harus punya IP di subnet `172.17.10.x` untuk bisa akses `enp0s9`.

  Kalau tidak ada, set manual:
  - Buka **Control Panel → Network and Internet → Network Connections** (`ncpa.cpl`)
  - Klik kanan adapter VirtualBox Host-Only → Properties → IPv4 → Properties
  - Set IP: `172.17.10.1`, Mask: `255.255.255.0`, Gateway: kosong

- **Firewall Windows block akses:**
  Test nonaktifkan sementara:
  - **Windows Security → Firewall & network protection → Private network → Turn off**

  Atau buat rule permanen (PowerShell sebagai Admin):
  ```powershell
  New-NetFirewallRule -DisplayName "VM HTTP" -Direction Inbound -LocalPort 80,8088 -Protocol TCP -Action Allow
  ```

- **Domain custom `USERNAME-adinusa.id` tidak resolve di Windows:**
  Windows tidak mengenal domain itu. Tambahkan ke file `hosts` Windows:
  1. Buka **Notepad sebagai Administrator**
  2. Buka `C:\Windows\System32\drivers\etc\hosts`
  3. Tambah baris: `172.17.10.10   USERNAME-adinusa.id   www.USERNAME-adinusa.id`
  4. Simpan

### Masalah Umum

- **`phpmyadmin` tidak bisa diakses di browser:** Cek urutan:
  1. Apache jalan? `sudo systemctl status apache2`
  2. Listen di port 8088? `sudo ss -tlnp | grep :8088`
  3. Site aktif? `sudo a2query -s`
  4. Firewall block? `sudo firewall-cmd --list-all`
  5. Dari host, ping VM dulu: `ping 172.17.10.10`

- **Error `Not Found` di browser:** DocumentRoot salah atau site belum di-enable. Cek `sudo apache2ctl -S`.

- **`The mbstring extension is missing`:** Jalankan `sudo phpenmod mbstring` lalu restart Apache.

- **`firewall-cmd: command not found`:** Firewalld belum terinstall. `sudo apt install firewalld -y`.

- **Firewalld tidak start:** Cek konflik dengan `ufw` — matikan ufw dulu:
  ```bash
  sudo systemctl disable --now ufw
  sudo systemctl enable --now firewalld
  ```

- **Port 8088 tidak bisa diakses meskipun sudah dibuka:** Cek interface `enp0s8` benar-benar ada di zone public:
  ```bash
  sudo firewall-cmd --list-interfaces --zone=public
  ```

- **Firewalld memblokir SSH:** Firewalld secara default mengizinkan SSH, tapi kalau tiba-tiba SSH terputus, dari console VM jalankan:
  ```bash
  sudo firewall-cmd --add-service=ssh --permanent
  sudo firewall-cmd --reload
  ```

- **`--permanent` vs tanpa `--permanent`:** Tanpa `--permanent` = hanya berlaku sampai reboot. Selalu pakai `--permanent` lalu `--reload`.

- **`phpmyadmin` prompt tidak muncul saat install:** Bisa karena instalasi sebelumnya setengah jalan. Reconfigure:
  ```bash
  sudo dpkg-reconfigure phpmyadmin
  ```

- **Port 8088 sudah dipakai proses lain:** Cek:
  ```bash
  sudo ss -tlnp | grep :8088
  ```
  Kalau ada, matikan proses itu atau ganti port.

---

## 💡 Catatan & Insight Pribadi

### Arsitektur Stack

```
Browser (Windows)
    ↓ HTTP :8088
Firewalld (zone public, interface enp0s8)
    ↓ allow :8088
Apache (VirtualHost :8088)
    ↓ PHP-FPM
phpMyAdmin (PHP app di /usr/share/phpmyadmin)
    ↓ mysqli
MySQL Server (user: USERNAME)
```

Setiap lapisan harus "terbuka" — kalau salah satu nutup, akses gagal.

### Kenapa Port Custom?

| Alasan | Penjelasan |
|---|---|
| **Keamanan** | Port non-standard, tidak mudah ditebak scanner |
| **Isolasi** | Web utama di :80, phpMyAdmin di :8088 |
| **Firewall** | Bisa buka/tutup port spesifik tanpa ganggu web utama |
| **Multi-app** | Satu server, banyak aplikasi di port berbeda |

### Firewalld — Konsep Zone

Firewalld memakai **zone** — kumpulan aturan. Zone umum:

| Zone | Kepercayaan | Default |
|---|---|---|
| `public` | Rendah (jaringan publik) | Aktif untuk interface baru |
| `trusted` | Tinggi (semua diizinkan) | — |
| `home` | Sedang (jaringan rumah) | — |
| `work` | Sedang (jaringan kantor) | — |
| `drop` | Block semua, no response | — |

**Interface assignment:**
- Interface yang di-assign ke zone, mengikuti aturan zone itu.
- `enp0s8` di-assign ke `public` → hanya port yang diizinkan yang bisa diakses.
- Interface lain (enp0s3 untuk NAT) tidak di-assign → default zone.

### Firewalld vs UFW

| Aspek | Firewalld | UFW |
|---|---|---|
| Distro khas | RHEL/CentOS/Fedora | Ubuntu/Debian |
| Konsep | Zone | Rule sederhana |
| Runtime vs permanent | Ya, terpisah | Tidak |
| Dynamic changes | Ya | Perlu reload |
| Kompleksitas | Sedang | Rendah |

Untuk server yang butuh aturan kompleks, firewalld lebih fleksibel. Untuk pemakaian simpel, ufw cukup.

### MySQL 8.0 vs MariaDB

| Aspek | MySQL 8.0 | MariaDB 10.11 |
|---|---|---|
| Developer | Oracle | Komunitas |
| Default plugin | `caching_sha2_password` | `mysql_native_password` |
| Kompatibilitas phpMyAdmin | Butuh versi terbaru | Lebih kompatibel |
| Ketersediaan di repo | Ubuntu main | Repo terpisah (MariaDB) |
| Bisa terinstall bersamaan? | ❌ Tidak | ❌ Tidak |

**Pelajaran penting:** Kalau mau MySQL 8.0, **jangan pernah** punya repo MariaDB aktif. Kalau tidak bisa hindari, pakai **MariaDB saja** — lebih kompatibel dengan phpMyAdmin di Ubuntu.

### Plugin Autentikasi MySQL 8.0

| Plugin | Keterangan |
|---|---|
| `caching_sha2_password` | Default MySQL 8.0 — **tidak kompatibel** dengan phpMyAdmin lama |
| `mysql_native_password` | Legacy, kompatibel dengan semua client |
| `sha256_password` | Alternatif, perlu SSL/TLS |

**Kapan pakai mana:**
- **Produksi modern:** `caching_sha2_password` (aman, tapi butuh client terbaru).
- **Kompatibilitas:** `mysql_native_password` (untuk client lama).
- **Lab / testing:** `mysql_native_password` — lebih sedikit masalah.

### Kapan Pakai phpMyAdmin?

| Skenario | Pakai phpMyAdmin? |
|---|---|
| Development | ✅ Ya — praktis |
| Manage database sesekali | ✅ Ya |
| Server produksi publik | ❌ Tidak — lebih baik CLI + tunnel SSH |
| Belajar MySQL | ✅ Ya |
| Akses database dari mobile | ✅ Ya |

> **Peringatan:** phpMyAdmin di server produksi publik = target utama attacker. Selalu:
> - Batasi akses via firewall
> - Pakai HTTPS
> - Ganti nama URL default
> - Nonaktifkan kalau tidak dipakai

### Alur Konfigurasi Firewalld

```
1. Install firewalld
2. Assign interface ke zone
3. Buka port/service di zone
4. Reload
5. Verifikasi
```

Semua perubahan permanen pakai `--permanent`, baru `--reload`.

### Kasus Nyata

- **Server admin:** phpMyAdmin diakses dari intranet, dibatasi firewall ke subnet internal saja.
- **Development:** phpMyAdmin untuk memudahkan developer manage DB tanpa SSH.
- **Backup:** Export database via phpMyAdmin — klik **Export** → dapat file `.sql`.
- **Migrasi:** Import `.sql` di server baru via **Import** di phpMyAdmin.

### Best Practice

1. **Jangan pakai password default** — `adinusa` hanya untuk lab.
2. **Batasi akses firewall** — hanya interface tertentu.
3. **Pakai HTTPS** untuk phpMyAdmin di production.
4. **Backup rutin** — jangan andalkan phpMyAdmin saja.
5. **Update rutin** — phpMyAdmin sering ada patch keamanan.
6. **Batasi privilege user MySQL** — jangan semua user `ALL PRIVILEGES`.
7. **Pakai `mysql_native_password`** kalau phpMyAdmin versi lama.
8. **Hindari konflik MySQL vs MariaDB** — pilih salah satu.

### Pelajaran Kunci dari Lab Ini

1. **Stack LAMP** = Linux + Apache + MySQL + PHP.
2. **phpMyAdmin** = GUI untuk MySQL.
3. **Port custom (8088)** untuk isolasi.
4. **Firewalld** = firewall dengan konsep zone.
5. **Interface assignment** = tentukan interface mana di-protect.
6. **`--permanent` + `--reload`** = cara commit perubahan firewalld.
7. **phpMyAdmin di production = risiko**, batasi aksesnya.
8. **MySQL 8.0 + phpMyAdmin lama = konflik plugin** — pakai `mysql_native_password`.
9. **User `@'%'` ≠ user `@'localhost'`** — buat keduanya untuk phpMyAdmin.
10. **MariaDB dan MySQL tidak bisa coexist** — pilih salah satu.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `sudo apt install apache2 mysql-server php libapache2-mod-php php-mysql` | Install stack LAMP |
| `sudo apt install php-{mbstring,zip,gd,curl}` | Install ekstensi PHP |
| `sudo apt install phpmyadmin` | Install phpMyAdmin |
| `sudo phpenmod mbstring` | Aktifkan ekstensi mbstring |
| `sudo systemctl restart apache2` | Restart Apache |
| `sudo systemctl enable --now apache2` | Enable + start Apache |
| `sudo mysql` | Login MySQL sebagai root (auth_socket) |
| `CREATE USER 'USERNAME'@'localhost' IDENTIFIED WITH mysql_native_password BY 'pass'` | Buat user lokal |
| `ALTER USER ... IDENTIFIED WITH mysql_native_password BY 'pass'` | Ubah plugin user |
| `DROP USER IF EXISTS 'USERNAME'@'localhost'` | Hapus user |
| `sudo a2ensite <site>` | Aktifkan vhost |
| `sudo apt install firewalld` | Install firewalld |
| `sudo firewall-cmd --add-interface=enp0s8 --permanent` | Assign interface ke zone |
| `sudo firewall-cmd --add-port=8088/tcp --zone=public --permanent` | Buka port |
| `sudo firewall-cmd --reload` | Reload konfigurasi |
| `sudo firewall-cmd --list-all --zone=public` | Lihat konfigurasi zone |

**File penting:**

| File | Isi |
|---|---|
| `/etc/apache2/sites-available/phpmyadmin.conf` | Config vhost phpMyAdmin |
| `/etc/apache2/ports.conf` | Port listening (`Listen 8088`) |
| `/usr/share/phpmyadmin/` | File aplikasi phpMyAdmin |
| `/etc/phpmyadmin/config.inc.php` | Config phpMyAdmin |
| `/var/log/apache2/phpmyadmin_error.log` | Log error phpMyAdmin |
| `/etc/apt/sources.list.d/` | Repo tambahan (MariaDB, dll) |

**Contoh query MySQL:**

```sql
-- Buat user (kedua host)
CREATE USER 'USERNAME'@'localhost' IDENTIFIED WITH mysql_native_password BY 'adinusa';
CREATE USER 'USERNAME'@'%' IDENTIFIED WITH mysql_native_password BY 'adinusa';

-- Grant privilege
GRANT ALL PRIVILEGES ON *.* TO 'USERNAME'@'localhost' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'USERNAME'@'%' WITH GRANT OPTION;

-- Refresh
FLUSH PRIVILEGES;

-- Cek user & plugin
SELECT user, host, plugin FROM mysql.user WHERE user='USERNAME';

-- Ubah plugin
ALTER USER 'USERNAME'@'localhost' IDENTIFIED WITH mysql_native_password BY 'adinusa';

-- Hapus user
DROP USER IF EXISTS 'USERNAME'@'localhost';
```

---

## 📌 Kesimpulan

Lab ini memperkenalkan **phpMyAdmin** — GUI web untuk mengelola MySQL — dengan proteksi **firewalld**. Kemampuan yang didapat:

- Install stack **LAMP** (Linux + Apache + MySQL + PHP).
- Konfigurasi **phpMyAdmin** di port custom.
- Mengelola database via browser.
- Konfigurasi **firewalld** dengan konsep zone dan interface assignment.

Yang paling penting dipahami: **keamanan berlapis**. phpMyAdmin tidak cukup hanya diinstall — harus dibatasi aksesnya via firewall, dan idealnya pakai HTTPS di production. Firewalld memberikan kontrol granular mana interface yang boleh akses, port mana yang dibuka.

Dari troubleshooting nyata, saya belajar bahwa **konflik dependency** (MySQL vs MariaDB) dan **plugin autentikasi** (`caching_sha2_password` vs `mysql_native_password`) adalah dua masalah paling umum yang akan dihadapi sysadmin di dunia kerja. Menguasai cara mengatasinya sangat berharga.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
