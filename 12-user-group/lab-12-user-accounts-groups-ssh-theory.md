# Lab 12.2 — User Accounts, Groups & SSH (Teori)

**Course:** Linux System Administration (Adinusa)
**Topic:** Konsep user, group, password, restricted shell, root, dan SSH
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Memahami konsep **multi-user environment** di Linux
- Menjelaskan struktur file **`/etc/passwd`**, **`/etc/shadow`**, dan **`/etc/group`**
- Membedakan **root user**, **normal user**, dan **service account**
- Mengelola **password aging** dengan `chage`
- Memahami **restricted shell** (`rbash`) dan keterbatasannya
- Menggunakan **SSH** untuk login remote, termasuk **key-based authentication**

---

## 📘 Bagian 1 — Konsep User Account

### Kenapa Linux Multi-User?

Linux dirancang sebagai sistem **multi-user** — banyak user (atau proses) bisa punya lingkungan kerja sendiri secara bersamaan. Tujuannya:

| Tujuan | Penjelasan |
|---|---|
| **Private workspace** | Setiap user punya file & setting sendiri, terisolasi dari user lain |
| **Dedicated service account** | Akun khusus untuk service (misal `www-data` untuk web server) |
| **Privilege separation** | Membedakan user biasa dan user administratif untuk keamanan |

### Tiga Jenis Akun

**1. Root user**

- Superuser dengan akses **tanpa batas**.
- Bisa membuat, mengubah, atau menghapus apapun.
- Sebaiknya dipakai **hanya saat benar-benar perlu** — risiko kesalahan besar.

**2. Normal user**

- Untuk pemakaian sehari-hari.
- Hak akses terbatas pada file & proses miliknya sendiri.
- Butuh `sudo` untuk tugas administratif.

**3. Service user (daemon)**

- Akun untuk proses yang berjalan di background.
- Tidak butuh hak root, jadi lebih aman.
- Contoh: `daemon`, `www-data`, `nobody`.
- Biasanya shell-nya diset ke `/sbin/nologin` atau `/bin/false`.

---

## 📘 Bagian 2 — Struktur `/etc/passwd`

File `/etc/passwd` menyimpan **semua akun user** di sistem. Satu baris = satu user, dengan **7 kolom** dipisahkan `:`.

**Contoh:**

```
beav:x:1000:1000:Theodore Cleaver:/home/beav:/bin/bash
warden:x:1001:1001:Ward Cleaver:/home/warden:/bin/bash
```

**Breakdown kolom:**

| # | Kolom | Contoh | Arti |
|---|---|---|---|
| 1 | Username | `beav` | Nama login unik |
| 2 | Password | `x` | Placeholder — hash ada di `/etc/shadow` |
| 3 | UID | `1000` | User ID (0 = root, <1000 = system, ≥1000 = normal) |
| 4 | GID | `1000` | Primary group ID |
| 5 | Comment (GECOS) | `Theodore Cleaver` | Nama lengkap / info kontak |
| 6 | Home Directory | `/home/beav` | Workspace user |
| 7 | Login Shell | `/bin/bash` | Program yang dijalankan saat login |

**Aturan UID (dari `/etc/login.defs`):**

- `0` → root
- `1–999` → system/service account
- `1000+` → normal user (default modern distro)

---

## 📘 Bagian 3 — Membuat User dengan `useradd`

### Perintah Dasar

```bash
$ sudo useradd dexter
```

### Apa yang Terjadi di Belakang Layar?

1. **UID assignment** — sistem ambil UID berikutnya ≥ `UID_MIN` (biasanya 1000).
2. **Group creation** — group baru dengan nama user dibuat, GID = UID (User Private Group).
3. **Home directory** — `/home/dexter` dibuat, owner user.
4. **Login shell** — default `/bin/bash`.
5. **Skeleton files** — isi `/etc/skel` disalin ke home (`.bashrc`, `.profile`).
6. **Password field** — diisi `!!` atau `!` di `/etc/shadow` → akun **belum aktif** sampai di-set password.

Aktifkan dengan:

```bash
$ sudo passwd dexter
```

### Overriding Defaults

```bash
$ sudo useradd -s /bin/csh -m -k /etc/skel -c "Bullwinkle J Moose" bmoose
```

| Opsi | Arti |
|---|---|
| `-s /bin/csh` | Set shell jadi C shell |
| `-m` | Buat home directory (kalau belum ada) |
| `-k /etc/skel` | Direktori skeleton untuk file awal |
| `-c "..."` | Set comment (nama lengkap) |

> ⚠️ Di beberapa distro, `useradd` **tidak** otomatis membuat home directory. Selalu pakai `-m`.

---

## 📘 Bagian 4 — Modifikasi & Hapus User

### `userdel` — Hapus User

```bash
$ sudo userdel morgan        # hapus user, home tetap ada
$ sudo userdel -r morgan     # hapus user + home directory
```

**Catatan:**
- `userdel` menghapus entry di `/etc/passwd`, `/etc/shadow`, `/etc/group`.
- File **di luar home** yang dimiliki user **tetap ada** (harus dibersihkan manual).

### `usermod` — Modifikasi User

| Opsi | Fungsi |
|---|---|
| `-c "comment"` | Ubah GECOS/comment |
| `-d /path` | Set home directory baru |
| `-m -d /path` | Pindah home + isi |
| `-e YYYY-MM-DD` | Set tanggal expired akun |
| `-f N` | Set inactive setelah password expired |
| `-g group` | Set primary group baru |
| `-G group` | Set supplementary group (**menimpa**, bukan menambah) |
| `-a -G group` | **Append** ke supplementary group |
| `-l newlogin` | Ubah username |
| `-L` / `-U` | Lock / unlock akun |
| `-s shell` | Ubah default shell |
| `-u UID` | Ubah UID |
| `-p hash` | Set password terenkripsi (langsung) |

**Aturan penting:** Untuk menambah user ke group **tanpa menghapus** group lama, wajib pakai `-aG`. Kalau hanya `-G`, group lama akan hilang.

---

## 📘 Bagian 5 — Akun Terkunci & Akun Sistem

### Akun Sistem

- Dibuat otomatis saat instalasi Linux.
- **Tidak untuk login manusia** — shell-nya `/sbin/nologin` atau `/bin/false`.
- Contoh:
  ```
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  ```
- Kalau coba login → pesan *"This account is currently not available"*.

### Lock Akun User dengan `usermod -L`

```bash
$ sudo usermod -L dexter    # lock
$ sudo usermod -U dexter    # unlock
```

**Cara kerja:** `usermod -L` menambahkan `!` di depan hash password di `/etc/shadow`. Login jadi tidak mungkin sampai di-unlock.

### Lock Akun dengan `chage -E`

```bash
$ sudo chage -E 2014-09-11 morgan
```

Set tanggal expired di masa lalu → akun tidak bisa login selamanya sampai tanggal diubah.

### Praktik Umum

> Saat karyawan resign atau cuti panjang, **lock akunnya, jangan hapus**. Ini mempertahankan file dan ownership metadata, sambil mencegah login.

---

## 📘 Bagian 6 — `/etc/shadow` & Password

### Kenapa Ada `/etc/shadow`?

- `/etc/passwd` **harus** bisa dibaca semua user (untuk lookup user).
- Kalau hash password ditaruh di sana, attacker bisa copy dan brute-force.
- Solusi: hash password dipisah ke `/etc/shadow` yang hanya bisa dibaca **root**.

**Permission:**

| File | Permission | Isi |
|---|---|---|
| `/etc/passwd` | `644` (`-rw-r--r--`) | Info akun non-sensitif |
| `/etc/shadow` | `400` (`-r--------`) | Hash password & aging |

### Fields di `/etc/shadow` (9 kolom)

| # | Field | Isi |
|---|---|---|
| 1 | Username | Sama dengan `/etc/passwd` |
| 2 | Password | Hash + salt (detail di bawah) |
| 3 | lastchange | Hari sejak 1 Jan 1970 password terakhir diubah |
| 4 | mindays | Minimum hari sebelum boleh ganti password lagi |
| 5 | maxdays | Maksimum hari password valid |
| 6 | warn | Hari peringatan sebelum expired |
| 7 | grace (inactive) | Hari setelah expired sebelum akun dinonaktifkan |
| 8 | expire | Tanggal absolut akun dinonaktifkan |
| 9 | reserved | Reserved untuk penggunaan masa depan |

**Format hash password:**

```
$6$iCZyCnBJH9rmq7P.$RYNm10Jg3wrhAtUnahBZ/mTMg...
 │  │                │
 │  │                └── hash
 │  └─────────────────── salt
 └────────────────────── algoritma
```

| Prefix | Algoritma |
|---|---|
| `$1$` | MD5 (lama, lemah) |
| `$5$` | SHA-256 |
| `$6$` | SHA-512 (paling umum) |

**Nilai khusus di kolom password:**

- `*` → akun disabled
- `!` → password locked
- `!!` → password belum di-set (default saat `useradd`)

---

## 📘 Bagian 7 — Manajemen Password dengan `passwd` & `chage`

### `passwd` — Ganti Password

**User biasa** (ganti password sendiri):

```bash
$ passwd
(current) UNIX password: ****
New UNIX password: ****
Retype new UNIX password: ****
passwd: all authentication tokens updated successfully
```

**Root** (ganti password user lain, tidak perlu password lama):

```bash
$ sudo passwd kevin
New UNIX password: ****
Retype new UNIX password: ****
```

### Password Quality Check

Dilakukan oleh **PAM** (`pam_cracklib.so` atau `pam_pwquality.so`). Aturan umum:

- Panjang minimum.
- Tidak berbasis username.
- Hindari kata kamus.
- Campuran huruf besar, kecil, angka, simbol.

### `chage` — Password Aging

**Syntax:**

```bash
chage [-m mindays] [-M maxdays] [-d lastday] [-I inactive] [-E expiredate] [-W warndays] user
```

| Opsi | Arti |
|---|---|
| `-m N` | Minimum hari sebelum boleh ganti password |
| `-M N` | Maksimum hari password valid |
| `-d N` | Set tanggal perubahan password terakhir (hari sejak epoch) |
| `-I N` | Hari setelah expired sebelum akun dinonaktifkan |
| `-E YYYY-MM-DD` | Tanggal akun expired |
| `-W N` | Hari peringatan sebelum expired |
| `-l` | Tampilkan info aging user (view only) |

**Contoh:**

```bash
# Cek info aging
$ sudo chage -l stephane

# Paksa ganti password tiap 30 hari, tidak boleh sebelum 14 hari
$ sudo chage -m 14 -M 30 kevlin

# Expire akun pada tanggal tertentu
$ sudo chage -E 2012-04-01 isabelle

# Paksa user ganti password saat login berikutnya
$ sudo chage -d 0 clyde
```

**Hanya root** yang bisa mengubah setting aging. User biasa hanya bisa cek sendiri dengan `chage -l`.

---

## 📘 Bagian 8 — Restricted Shell (`rbash`)

### Apa itu `rbash`?

Versi terbatas dari Bash. Dipakai untuk membatasi apa yang bisa dilakukan user dalam sesi shell — cocok untuk **guest account**, **demo environment**, atau akun khusus aplikasi.

### Cara Invoke

```bash
$ bash -r
# atau
$ ln -s /bin/bash /bin/rbash
# lalu set /bin/rbash sebagai login shell di /etc/passwd
```

### Batasan `rbash`

- ❌ Tidak bisa `cd` ke direktori lain.
- ❌ Tidak bisa set/modifikasi env var (`SHELL`, `ENV`, `PATH`).
- ❌ Tidak bisa jalankan command dengan `/` di path (`/bin/ls`).
- ❌ Tidak bisa redirect input/output (`>`, `<`, `>>`).

### ⚠️ `rbash` Bukan Security Tool

> **Penting:** `rbash` **mudah di-bypass**. User berpengalaman bisa keluar dari batasan ini lewat editor (misal `vi`), pager (`less`), atau command substitution. Jangan dijadikan satu-satunya mekanisme keamanan.

### Alternatif yang Lebih Aman

| Alternatif | Kelebihan |
|---|---|
| **SELinux** | Mandatory access control, sangat granular |
| **AppArmor** | Mirip SELinux, lebih mudah dikonfigurasi |
| **Container (Docker, LXC)** | Isolasi kuat di level OS |
| **chroot jail** | Batasi view filesystem proses |

### Pertimbangan Keamanan `rbash`

1. **Home directory permission** — Pastikan user **tidak punya write/execute** di home, karena dia bisa edit `.bash_profile` untuk bypass.
2. **PATH** — Jangan include `/bin` atau `/usr/bin` di PATH, karena user bisa jalankan `/bin/bash` untuk keluar dari `rbash`.
3. **False sense of security** — `rbash` bukan pengganti mekanisme keamanan sejati.

---

## 📘 Bagian 9 — Root Account

### Karakteristik Root

- Superuser dengan **unrestricted privileges**.
- Bisa modifikasi/hapus file apapun, kill proses apapun, konfigurasi apapun.
- **Risiko:** kesalahan kecil bisa merusak sistem, dan kalau akunnya diambil alih attacker, seluruh sistem jatuh.

### Praktik Keamanan Root

**1. Gunakan `sudo` daripada login langsung sebagai root.**

- `sudo` memberi privilege root ke user/group tertentu.
- Ada **audit trail** — semua perintah `sudo` dicatat.
- Lebih aman dari membagikan password root.

**2. Gunakan `su` saat perlu.**

- `su -` switch shell ke root setelah masukkan password root.
- Bisa dibatasi via PAM agar hanya user tertentu yang boleh `su`.

**3. Hindari login root langsung.**

- Ubuntu **menonaktifkan** akun root secara default.
- Administrator pakai `sudo` untuk task elevated.

**4. Konfigurasi SSH agar tidak izinkan root login.**

```
# /etc/ssh/sshd_config
PermitRootLogin no
```

**5. Audit aktivitas root.**

- Gunakan `auditd` atau system log untuk track semua perintah root.

---

## 📘 Bagian 10 — Groups

### Konsep

Group adalah **kumpulan user** yang berbagi file, direktori, dan privilege. User bisa jadi anggota **satu primary group** dan **beberapa secondary group** (maksimum 15).

### Format `/etc/group`

```
groupname:password:GID:user1,user2,...
```

| Kolom | Arti |
|---|---|
| groupname | Nama group |
| password | Biasanya `x` (hash ada di `/etc/gshadow`) |
| GID | Group ID (0–99 system, 100–999 special, ≥1000 UPG) |
| members | Daftar user (kosong kalau ini primary group user) |

### User Private Group (UPG)

- Setiap user baru dapat **group pribadi** dengan nama sama dengan username.
- Primary GID = UID.
- **Default umask** dengan UPG:
  - File → `664` (`rw-rw-r--`)
  - Direktori → `775` (`rwxrwxr-x`)
- Memudahkan kolaborasi dalam group yang sama.

### Primary vs Secondary Group

| Aspek | Primary | Secondary |
|---|---|---|
| Didefinisikan di | `/etc/passwd` (kolom 4) | `/etc/group` |
| Dipakai saat | User buat file/direktori | Additional access |
| Jumlah | 1 | Maks 15 |

### Cek Group Membership

```bash
$ groups [username]
$ id -Gn [username]
```

Contoh output:

```
student adm cdrom sudo dip plugdev lpadmin sambashare libvirt
```

---

## 📘 Bagian 11 — SSH (Secure Shell)

### Kenapa SSH?

SSH adalah protokol standar untuk login remote & transfer file secara aman. Menyediakan:

- **Enkripsi** — data tidak bisa disadap.
- **Autentikasi** — hanya user yang berhak bisa masuk.
- **Integritas** — data tidak bisa diubah di tengah jalan.

### Penggunaan Dasar

```bash
# Login dengan username default (sama dengan lokal)
$ ssh remote_computer.com

# Login dengan username spesifik
$ ssh some_user@remote_computer.com
$ ssh -l some_user remote_computer.com

# Jalankan perintah remote
$ ssh some_user@remote_computer.com apt-get update

# Copy file dengan SCP
$ scp file.txt remote_computer.com:/tmp
$ scp -r some_dir farflung.com:/tmp/some_dir

# Copy antar dua remote host
$ scp usr1@rem1.com:/tmp/f.txt usr2@rem2.com:/tmp/nf.txt
```

### SSH Key-Based Authentication

Lebih aman dan nyaman dari password login. Menggunakan **pasangan kunci publik-privat**:

- **Private key** → disimpan aman di komputer lokal (**jangan pernah dibagikan**).
- **Public key** → disimpan di server remote (`~/.ssh/authorized_keys`).

Saat connect, server pakai public key untuk mengirim **challenge** yang hanya bisa dijawab private key.

### Generate Key Pair

```bash
$ ssh-keygen
```

Prompt:

```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/student/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/student/.ssh/id_rsa
Your public key has been saved in /home/student/.ssh/id_rsa.pub
```

**Lokasi file:**

| File | Permission |
|---|---|
| `~/.ssh/id_rsa` (private) | `600` |
| `~/.ssh/id_rsa.pub` (public) | `644` |

**Passphrase** bersifat opsional, tapi menambah lapisan keamanan kalau private key dicuri.

### Copy Public Key ke Server

```bash
$ ssh-copy-id -i ~/.ssh/id_rsa.pub user@remotehost
```

Perintah ini menambahkan public key ke `~/.ssh/authorized_keys` di server. Setelah itu, login tanpa password:

```bash
$ ssh user@remotehost
```

### Otomasi Multi-Server

```bash
for machines in node1 node2 node3
do
    (ssh $machines some_command &)
done
```

---

## 🐛 Troubleshooting & Kesalahan

- **`useradd` tidak membuat home directory:** Beberapa distro tidak otomatis buat home tanpa opsi `-m`. Selalu cek dengan `ls /home/` setelah `useradd`.

- **User tidak bisa login setelah `useradd`:** Karena password belum di-set — field di `/etc/shadow` diisi `!!` (locked). Jalankan `passwd username`.

- **Group baru tidak aktif setelah `usermod -aG`:** Keanggotaan group baru hanya berlaku setelah **logout & login ulang**. Solusi cepat: `su - username` atau `newgrp nama-group`.

- **`usermod -G` tanpa `-a` menghapus group lama:** Ini kesalahan klasik. **Selalu `-aG`** untuk append.

- **Edit `/etc/passwd` langsung menyebabkan korup:** Jangan pernah edit manual. Gunakan `usermod`, `useradd`, `passwd`, `groupmod`.

- **`/etc/shadow` permission salah:** Kalau permission bukan `400`, sistem akan warning dan bisa menolak login. Cek dengan `ls -l /etc/shadow`.

- **Password aging terlalu ketat:** User cenderung menulis password di kertas atau memilih variasi lemah. Gunakan kebijakan yang wajar (misal 90 hari, bukan 30 hari).

- **`rbash` di-bypass oleh user:** Ingat bahwa `rbash` **bukan security tool**. Gunakan SELinux, AppArmor, atau container untuk keamanan sejati.

- **SSH key tidak berfungsi setelah `ssh-copy-id`:** Cek permission di server:
  - `~/.ssh` → `700`
  - `~/.ssh/authorized_keys` → `600`
  - Home directory tidak boleh **group-writable**.

- **Root login via SSH ditolak:** Karena `PermitRootLogin no` di `/etc/ssh/sshd_config`. Ini memang best practice. Login sebagai user biasa dulu, baru `sudo`.

- **`chage -d 0` tidak memaksa ganti password:** Kadang user perlu login ulang. Cek dengan `chage -l username` — kolom *Last password change* harus menunjukkan *"password must be changed"*.

---

## 💡 Catatan & Insight Pribadi

### File yang Terpengaruh Saat Buat/Hapus User

| File | Perubahan |
|---|---|
| `/etc/passwd` | Entry user baru |
| `/etc/shadow` | Password (kosong / `!!`) |
| `/etc/group` | Group pribadi user |
| `/etc/gshadow` | Password group |
| `/home/username/` | Home directory |
| `/var/mail/username` | Mail spool |

### UID/GID Convention

| Range | Peruntukan |
|---|---|
| `0` | root |
| `1–999` | System / service account |
| `1000+` | Normal user |
| `65534` | nobody |

### Root vs Sudo — Kapan Pakai Apa?

| Skenario | Pakai |
|---|---|
| Tugas administratif sesekali | `sudo` |
| Maintenance sistem intensif | `su -` |
| Otomasi (cron, script) | `sudo NOPASSWD` (dengan hati-hati) |
| Debug dari remote | Login user biasa → `sudo` |

### SSH Key vs Password

| Aspek | Password | Key-Based |
|---|---|---|
| Keamanan | Rentan brute force | Sangat kuat |
| Kenyamanan | Perlu ketik tiap login | Sekali setup |
| Otomasi | Sulit | Mudah (ssh agent) |
| Cocok untuk | Akses manual sesekali | Server production, CI/CD |

### Kasus Nyata

- **Server production:** Root login via SSH **dimatikan** (`PermitRootLogin no`). Administrator pakai user biasa + `sudo`.
- **Onboarding user baru:** Buat user, set password sementara, paksa ganti password saat login pertama (`chage -d 0`), tambahkan ke group yang sesuai.
- **Offboarding karyawan:** **Lock** akun (`usermod -L`), jangan hapus — untuk preservasi file & audit.
- **Service account:** Dibuat tanpa shell (`/sbin/nologin`), hanya untuk menjalankan daemon.
- **Audit akses:** Cek `last`, `lastlog`, `who`, dan log `/var/log/auth.log` untuk track login.

### Aturan Emas

> **Jangan pernah edit `/etc/passwd`, `/etc/shadow`, atau `/etc/group` secara langsung.** Selalu gunakan tools resmi (`useradd`, `usermod`, `userdel`, `passwd`, `chage`, `groupadd`, `groupmod`). Direct editing bisa menyebabkan korupsi yang sulit diperbaiki dan bisa mengunci kamu dari sistem.

---

## 🧠 Perintah & Konsep yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `useradd` | Buat user |
| `userdel -r` | Hapus user + home |
| `usermod -aG` | Append user ke supplementary group |
| `usermod -L` / `-U` | Lock / unlock user |
| `passwd <user>` | Set password |
| `groupadd -g <gid>` | Buat group dengan GID |
| `id <user>` | Info user (UID, GID, groups) |
| `groups <user>` | Daftar group user |
| `chage -l <user>` | Lihat info aging |
| `chage -d 0 <user>` | Force ganti password |
| `chage -E <date> <user>` | Expire akun |
| `bash -r` | Restricted shell |
| `ssh user@host` | Login remote |
| `ssh-keygen` | Generate SSH key pair |
| `ssh-copy-id` | Copy public key ke server |
| `scp file host:/path` | Copy file via SSH |

**File penting:**

| File | Permission | Isi |
|---|---|---|
| `/etc/passwd` | `644` | Data user |
| `/etc/shadow` | `400` | Hash password & aging |
| `/etc/group` | `644` | Data group |
| `/etc/gshadow` | `400` | Password group |
| `/etc/login.defs` | `644` | Default aturan user |
| `/etc/skel/` | — | Template file home user baru |
| `/etc/ssh/sshd_config` | `600` | Konfigurasi SSH server |
| `~/.ssh/id_rsa` | `600` | Private key |
| `~/.ssh/id_rsa.pub` | `644` | Public key |
| `~/.ssh/authorized_keys` | `600` | Public key yang diizinkan |

---

## 📌 Kesimpulan

Lab ini memberikan pemahaman konseptual yang mendalam tentang **manajemen user, group, dan SSH** di Linux. Yang paling penting dipahami:

1. **Tiga jenis akun** (root, normal, service) dan kapan masing-masing dipakai.
2. **Struktur file** `/etc/passwd`, `/etc/shadow`, `/etc/group` dan peran masing-masing.
3. **Password aging** dengan `chage` — kapan menggunakannya dan bagaimana.
4. **Root account** — hindari login langsung, gunakan `sudo` untuk audit trail.
5. **SSH key-based authentication** — lebih aman dan nyaman dari password.
6. **Restricted shell** bukan security tool — jangan andalkan untuk keamanan sejati.

Kemampuan mengelola user dan akses adalah inti dari sysadmin. Kesalahan kecil di sini bisa berdampak besar — dari user tidak bisa login sampai sistem tidak bisa diakses sama sekali. Karena itu, kebiasaan menggunakan tool resmi dan tidak mengedit file konfigurasi secara langsung sangat penting.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
