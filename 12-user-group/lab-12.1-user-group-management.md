# Lab 12.1 — Manajemen User & Group

**Course:** Linux System Administration (Adinusa)
**Topic:** User, group, comment, password, supplementary group, dan sudo privilege
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membuat dan mengelola **user account**
- Menambahkan **comment** dan **password** ke user
- Membuat dan mengelola **group**
- Menambahkan user ke **supplementary group**
- Mengonfigurasi **sudo privilege** untuk sebuah group dan mengujinya

---

## 📘 Konsep Dasar

| Konsep | Penjelasan |
|---|---|
| **User** | Akun yang bisa login ke sistem, punya UID, home directory, dan shell |
| **Group** | Kumpulan user, punya GID, dipakai untuk permission bersama |
| **Primary Group** | Group utama user (dari `/etc/passwd`), biasanya sama dengan nama user |
| **Supplementary Group** | Group tambahan yang diikuti user, dicatat di `/etc/group` |
| **Comment (GECOS)** | Deskripsi user di kolom ke-5 `/etc/passwd` |
| **sudoers** | Konfigurasi siapa yang boleh pakai `sudo` dan sebagai apa |

**File-file penting:**

| File | Isi |
|---|---|
| `/etc/passwd` | Data user (nama, UID, GID, comment, home, shell) |
| `/etc/shadow` | Password user (terenkripsi) |
| `/etc/group` | Data group dan anggotanya |
| `/etc/sudoers` + `/etc/sudoers.d/` | Konfigurasi sudo |

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-012-1
```

---

## 📘 Guided Example

### 1. Switch ke root

```bash
$ sudo -i
```

### 2. Buat user `operator1` – `operator3`

```bash
$ useradd operator1
$ useradd operator2
$ useradd operator3
$ cat /etc/passwd | grep operator
```

### 3. Set password (semua: `adinusa`)

```bash
$ passwd operator1
New password: adinusa
Retype new password: adinusa
passwd: all authentication tokens updated successfully.

$ passwd operator2
$ passwd operator3
```

### 4. Tambahkan comment ke setiap user

```bash
$ usermod -c "Operator One" operator1
$ usermod -c "Operator Two" operator2
$ usermod -c "Operator Three" operator3
$ tail /etc/passwd | grep operator
```

**Penjelasan:** Opsi `-c` pada `usermod` menulis ke kolom **GECOS** (comment) di `/etc/passwd`.

### 5. Buat user `sysadmin1` – `sysadmin3`

```bash
$ useradd sysadmin1
$ useradd sysadmin2
$ useradd sysadmin3

$ passwd sysadmin1
$ passwd sysadmin2
$ passwd sysadmin3

$ usermod -c "System Admin1" sysadmin1
$ usermod -c "System Admin2" sysadmin2
$ usermod -c "System Admin3" sysadmin3
$ cat /etc/passwd | grep sysadmin
```

### 6. Hapus user `operator3`

```bash
$ userdel -r operator3
$ cat /etc/passwd | grep operator
```

**Penjelasan opsi `-r`:** Menghapus juga home directory dan mail spool user. Tanpa `-r`, file di home directory tetap tertinggal.

### 7. Buat group `ops` dengan GID 3000

```bash
$ groupadd -g 3000 ops
$ cat /etc/group | grep ops
```

### 8. Tambahkan `operator1` dan `operator2` ke group `ops`

```bash
$ usermod -aG ops operator1
$ usermod -aG ops operator2
$ id operator1
$ id operator2
```

**Penting:** Opsi `-aG` = **append** + **supplementary group**. Kalau hanya `-G` tanpa `-a`, group lama user akan **diganti**, bukan ditambahkan.

### 9. Tambahkan `sysadmin1` – `sysadmin3` ke group `admin`

```bash
$ usermod -aG admin sysadmin1
$ usermod -aG admin sysadmin2
$ usermod -aG admin sysadmin3
$ id sysadmin1
$ id sysadmin2
$ id sysadmin3
```

### 10. Buat konfigurasi sudoers untuk group `admin`

```bash
$ echo "%admin ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/admin
```

**Penjelasan sintaks sudoers:**

```
%admin  ALL=(ALL)  NOPASSWD:ALL
  │       │    │        │
  │       │    │        └── perintah yang boleh dijalankan
  │       │    └─────────── sebagai user/group apa
  │       └──────────────── host yang berlaku
  └──────────────────────── nama group (tanda % = group)
```

- `%admin` → semua anggota group `admin`.
- `ALL=(ALL)` → di semua host, sebagai semua user.
- `NOPASSWD:ALL` → bisa jalankan semua perintah **tanpa** diminta password.

### 11. Test sudo access dengan `sysadmin1`

```bash
$ su - sysadmin1
sysadmin1:~$ sudo cat /etc/sudoers.d/admin
%admin ALL=(ALL) NOPASSWD:ALL
sysadmin1:~$ exit
```

Perhatikan: tidak ada prompt password karena `NOPASSWD` sudah diset.

### 12. Kembali ke user `student`

```bash
$ exit
$ exit
student:~$
```

---

## 🧪 Practice Task

### Soal

1. Buat user `dev1` dan `dev2` (password: `adinusa`).
2. Tambahkan comment `Developer One` dan `Developer Two`.
3. Buat group `devteam` dengan GID `4000`.
4. Tambahkan `dev1` dan `dev2` ke group `devteam`.
5. Buat file `/etc/sudoers.d/devteam` sehingga anggota group `devteam` bisa menjalankan **hanya** perintah `systemctl` tanpa password.
6. Test dengan `dev1` menggunakan `sudo systemctl status ssh`.

### ✅ Solusi

```bash
# 1. Buat user
$ useradd dev1
$ useradd dev2
$ echo "adinusa" | passwd --stdin dev1
$ echo "adinusa" | passwd --stdin dev2

# 2. Comment
$ usermod -c "Developer One" dev1
$ usermod -c "Developer Two" dev2

# 3. Group dengan GID 4000
$ groupadd -g 4000 devteam

# 4. Tambah user ke group
$ usermod -aG devteam dev1
$ usermod -aG devteam dev2

# 5. Sudoers: hanya systemctl, tanpa password
$ echo "%devteam ALL=(ALL) NOPASSWD:/usr/bin/systemctl" > /etc/sudoers.d/devteam

# 6. Test
$ su - dev1
dev1:~$ sudo systemctl status ssh
dev1:~$ exit
```

### 🔍 Hasil yang Diharapkan

- `id dev1` menampilkan group `devteam`.
- `sudo systemctl status ssh` berjalan tanpa password.
- `sudo cat /etc/shadow` **gagal** karena tidak ada di sudoers.

---

## 🐛 Troubleshooting & Kesalahan

- **`usermod -G` tanpa `-a` menghapus group lama:** Ini kesalahan klasik. Kalau user `operator1` sudah punya group `ops` dan kamu jalankan `usermod -G admin operator1` (tanpa `-a`), maka group `ops` akan **hilang**. Selalu pakai `usermod -aG` untuk menambahkan group.

- **Group baru tidak langsung berlaku:** Setelah `usermod -aG`, user **harus logout & login ulang** agar keanggotaan group yang baru aktif. Kalau tidak, `id` mungkin sudah menampilkan group baru, tapi akses file/privilege belum berlaku. Solusi cepat untuk testing:
  ```bash
  $ su - username
  ```
  atau
  ```bash
  $ newgrp nama-group
  ```

- **`/etc/sudoers.d/admin` salah tulis = sudo rusak:** Kalau isi file salah, semua anggota group `admin` bisa kehilangan akses sudo (atau lebih parah, semua sudo rusak). **Selalu verifikasi** dengan:
  ```bash
  $ visudo -c
  ```
  Ini akan memvalidasi semua file sudoers. Kalau ada error, perbaiki sebelum logout.

- **Hapus user tanpa `-r`:** `userdel operator3` tanpa `-r` akan menghapus user dari `/etc/passwd` tapi **home directory tetap ada**. Ini bisa jadi masalah privasi. Biasakan pakai `userdel -r` kalau user memang tidak dibutuhkan lagi.

- **User dengan UID < 1000:** Sistem biasanya menggunakan UID 0 untuk root, 1–999 untuk system users, 1000+ untuk user biasa. `useradd` default memberikan UID di atas 1000. Kalau butuh UID spesifik, gunakan `useradd -u 1500 nama-user`.

- **Password kosong / `passwd` gagal:** Saat `passwd` diminta password, kadang user mengetik dan tidak ada tanda visual (untuk keamanan). Kalau ragu, cek dengan:
  ```bash
  $ passwd -S username
  ```
  Status `P` berarti password sudah diset, `L` berarti locked, `NP` berarti no password.

- **`su - sysadmin1` gagal "Permission denied":** Pastikan user punya shell yang valid (`/bin/bash`, bukan `/sbin/nologin`). Cek dengan:
  ```bash
  $ grep sysadmin1 /etc/passwd
  ```

- **`sudo` tidak jalan meski sudah di group:** Kemungkinan group belum aktif di sesi saat ini. Logout & login ulang, atau cek dengan `groups`:
  ```bash
  $ groups
  ```

- **File di `/etc/sudoers.d/` harus permission 0440:** Setelah membuat file dengan `echo ... >`, cek permission-nya:
  ```bash
  $ chmod 0440 /etc/sudoers.d/admin
  ```
  sudo akan **menolak** file dengan permission yang terlalu longgar.

- **Nama file di `/etc/sudoers.d/` tidak boleh mengandung titik:** sudo akan mengabaikan file yang namanya mengandung `.` (misal `admin.conf`). Gunakan nama tanpa ekstensi.

---

## 💡 Catatan & Insight Pribadi

- **`useradd` vs `adduser`:** `useradd` adalah perintah low-level, tidak membuat home directory secara default (kecuali `-m`). `adduser` adalah wrapper Debian/Ubuntu yang lebih ramah (membuat home directory, prompt password, dll). Untuk scripting dan konsistensi lintas distro, `useradd` lebih sering dipakai.

- **UID dan GID:**
  - `0` = root.
  - `1–999` = system user/group.
  - `1000+` = user biasa.
  - `65534` = nobody (user tanpa privilege).

- **Cek informasi user lengkap:**
  ```bash
  $ id username
  $ finger username   # kalau finger terinstal
  $ getent passwd username
  $ getent group nama-group
  ```

- **File yang terpengaruh saat buat user:**
  - `/etc/passwd` → data user
  - `/etc/shadow` → password
  - `/etc/group` → primary group baru (biasanya sama dengan username)
  - `/etc/gshadow` → password group
  - `/home/username/` → home directory (kalau `-m`)
  - `/var/mail/username` → mail spool

- **`usermod` — opsi penting:**

  | Opsi | Fungsi |
  |---|---|
  | `-c "comment"` | Set comment |
  | `-d /path` | Set home directory |
  | `-m` | Pindahkan isi home directory |
  | `-s /bin/bash` | Set shell |
  | `-aG group` | Tambah ke supplementary group (append) |
  | `-L` | Lock user |
  | `-U` | Unlock user |

- **`sudoers` format lengkap:**
  ```
  user  host=(runas)  [NOPASSWD:]command
  ```
  Contoh:
  ```
  operator1  ALL=(root)  NOPASSWD:/usr/bin/systemctl restart nginx
  %admin     ALL=(ALL)   ALL
  ```

- **Kenapa `/etc/sudoers.d/` lebih baik dari `/etc/sudoers`?**
  - Modular — satu file per group/user.
  - Tidak mengganggu file utama saat update paket.
  - Lebih mudah di-backup dan di-restore.

- **`NOPASSWD` — kapan boleh dan kapan bahaya?**
  - **Boleh**: untuk otomasi (script, cron), untuk perintah spesifik yang aman (status, reload).
  - **Bahaya**: kalau diberikan ke `ALL` untuk group besar. Kalau user bisa jalankan `sudo bash`, dia bisa jadi root sepenuhnya. **Jangan pakai `NOPASSWD:ALL` di production** kecuali benar-benar tahu risikonya.

- **Best practice sudo:**
  - Pakai group, bukan user langsung.
  - Batasi perintah yang diizinkan.
  - Hindari `NOPASSWD:ALL` kecuali untuk akun service.
  - Selalu test dengan `visudo -c`.

- **Test sudo dengan aman:**
  ```bash
  $ sudo -l -U username
  ```
  Menampilkan daftar perintah yang boleh dijalankan user tersebut **tanpa** perlu login.

- **Kasus nyata:** Di server produksi, pola umum:
  - Group `sysadmin` → sudo penuh.
  - Group `dba` → sudo hanya untuk perintah MySQL/PostgreSQL.
  - Group `webadmin` → sudo hanya untuk `nginx`/`apache2`.
  - Group `netadmin` → sudo hanya untuk `systemctl restart networking`.

- **Audit login user:**
  ```bash
  $ last              # riwayat login
  $ lastlog           # login terakhir setiap user
  $ who               # user yang sedang login
  ```

- **Force user ganti password saat login berikutnya:**
  ```bash
  $ chage -d 0 username
  ```
  Berguna saat onboarding user baru atau setelah reset password.

- **Lock user tanpa hapus:**
  ```bash
  $ usermod -L username       # lock
  $ usermod -U username       # unlock
  $ passwd -l username        # lock via passwd
  ```

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `sudo -i` | Masuk sebagai root |
| `useradd <user>` | Buat user |
| `userdel -r <user>` | Hapus user + home directory |
| `usermod -c "comment" <user>` | Set comment |
| `usermod -aG <group> <user>` | Tambah user ke supplementary group |
| `passwd <user>` | Set password |
| `groupadd -g <gid> <group>` | Buat group dengan GID spesifik |
| `id <user>` | Lihat UID, GID, dan group user |
| `groups <user>` | Lihat daftar group user |
| `cat /etc/passwd \| grep <user>` | Cek user di `/etc/passwd` |
| `cat /etc/group` | Lihat semua group |
| `echo "%group ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/<name>` | Set sudo privilege |
| `su - <user>` | Switch user dengan environment baru |
| `sudo -l -U <user>` | Lihat sudo privilege user |
| `visudo -c` | Validasi konfigurasi sudoers |

---

## 📌 Kesimpulan

Lab ini memberikan keterampilan praktis paling dasar bagi sysadmin: **manajemen user dan group**. Kemampuan membuat user, mengatur password, menambahkan ke group, dan memberikan sudo privilege adalah tugas sehari-hari di server. Yang paling penting dari lab ini adalah pemahaman tentang **supplementary group** (dan opsi `-aG` yang wajib), serta **sudoers.d** untuk memberikan akses administratif secara modular dan aman.

Pelajaran kunci yang akan terus dipakai:
- **Selalu pakai `usermod -aG`, bukan `-G`** — supaya tidak menghapus group lama.
- **User harus logout & login ulang** setelah diubah group-nya.
- **`NOPASSWD:ALL` itu bahaya** — di production, batasi perintah yang diizinkan.
- **Modular sudoers** di `/etc/sudoers.d/` jauh lebih rapi dari mengedit `/etc/sudoers` langsung.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
