# Lab 12.5 — Quiz: Create User and Group

**Course:** Linux System Administration (Adinusa)
**Topic:** Quiz — user, password policy, shell, dan group
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membuat **user account lokal** dengan konfigurasi spesifik
- Set **password awal** dan memaksa **ganti password saat login pertama**
- Menetapkan **default shell** tertentu untuk user
- Membuat **group lokal** baru
- Menambahkan user ke group

---

## 📘 Konsep Dasar

| Perintah | Fungsi |
|---|---|
| `useradd` | Membuat user baru |
| `passwd` | Set password user |
| `usermod` | Modifikasi atribut user (shell, group, dll) |
| `chage -d 0` | Paksa ganti password saat login berikutnya |
| `groupadd` | Membuat group baru |

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-012-5
```

> **Catatan:** Nama user yang dibuat mengikuti username Adinusa masing-masing peserta. Di catatan ini, nama user ditulis sebagai `<username>` untuk alasan privasi.

---

## 🧪 Challenge (Quiz)

### Soal

1. Buat user lokal dengan nama **`<username>`** (sesuai username Adinusa).
2. Set password awal: **`bta!@#adn`**.
3. Set default shell user ke **`/bin/sh`**.
4. Modifikasi user sehingga password **harus diganti saat login pertama**.
5. Buat group lokal baru bernama **`mentor`**.
6. Tambahkan user **`<username>`** ke group **`mentor`**.

### ✅ Solusi

```bash
# 1. Buat user
$ sudo useradd <username>

# 2. Set password awal
$ sudo passwd <username>
New password: bta!@#adn
Retype new password: bta!@#adn
passwd: all authentication tokens updated successfully

# 3. Set default shell ke /bin/sh
$ sudo usermod -s /bin/sh <username>

# 4. Paksa ganti password saat login pertama
$ sudo chage -d 0 <username>

# 5. Buat group mentor
$ sudo groupadd mentor

# 6. Tambahkan user ke group mentor
$ sudo usermod -aG mentor <username>
```

### 🔍 Verifikasi

```bash
# Cek user sudah dibuat dengan shell yang benar
$ grep <username> /etc/passwd
<username>:x:1001:1001::/home/<username>:/bin/sh
                                          ↑ shell /bin/sh

# Cek password aging (harus menunjukkan "password must be changed")
$ sudo chage -l <username>
Last password change                                    : password must be changed
Password expires                                        : password must be changed
...

# Cek group mentor sudah dibuat
$ grep mentor /etc/group
mentor:x:1002:

# Cek user sudah masuk group mentor
$ id <username>
uid=1001(<username>) gid=1001(<username>) groups=1001(<username>),1002(mentor)
                                                                       ↑ group mentor
```

### 🎯 Grading

```bash
$ nusactl grade linlab-012-5
```

---

## 🐛 Troubleshooting & Kesalahan

- **Case 3 gagal — password tidak dipaksa ganti:** Penyebab paling umum adalah `chage -d 0` **belum dijalankan** atau **dijalankan sebelum password di-set**. Cek dengan `sudo chage -l <username>` — kalau baris *"Last password change"* menunjukkan tanggal normal (bukan *"password must be changed"*), berarti belum di-set. Solusi:
  ```bash
  sudo chage -d 0 <username>
  ```
  Setelah itu cek `/etc/shadow`, field ke-3 harus bernilai **`0`**.

- **Typo `>` di akhir perintah:** Sering terjadi saat copy-paste. Tanda `>` bikin bash menampilkan prompt lanjutan (`>`) dan menunggu input. Tekan **CTRL + C** untuk membatalkan, lalu ketik ulang perintah **tanpa** `>`.

- **`useradd` tanpa `-m` tidak membuat home directory:** Di beberapa distro, `useradd` tidak otomatis membuat home. Kalau lab membutuhkan home directory:
  ```bash
  $ sudo useradd -m <username>
  ```

- **Password ditolak karena kebijakan PAM:** Password `bta!@#adn` mungkin ditolak kalau kebijakan password di sistem ketat. Cek `/etc/security/pwquality.conf`. Di environment lab, biasanya password ini sudah disiapkan dan diterima.

- **`usermod -s /bin/sh` gagal "shell not found":** Pastikan `/bin/sh` ada:
  ```bash
  $ ls -l /bin/sh
  $ cat /etc/shells
  ```

- **`usermod -aG` vs `usermod -G`:** Untuk **menambahkan** user ke group tanpa menghapus group lain, wajib pakai `-aG`:
  ```bash
  $ sudo usermod -aG mentor <username>   # ✅ append
  $ sudo usermod -G mentor <username>    # ❌ overwrite
  ```

- **Group `mentor` sudah ada:** Gunakan `groupadd -f` untuk idempotent:
  ```bash
  $ sudo groupadd -f mentor
  ```

- **`id` tidak menampilkan group mentor:** Group baru hanya aktif setelah user **login ulang**. Untuk sesi saat ini, cek dengan `groups <username>` atau `getent group mentor`.

- **Urutan operasi penting:** Buat user → set password → ubah shell → paksa ganti password → buat group → tambahkan ke group. Kalau `chage -d 0` dijalankan sebelum password di-set, hasilnya bisa tidak sesuai.

- **Password `bta!@#adn` mengandung karakter khusus:** Karakter `!` bisa memicu **history expansion** di bash. Saat mengetik di prompt `passwd`, ketik apa adanya. Kalau lewat `echo`, gunakan **kutip tunggal**:
  ```bash
  $ echo 'bta!@#adn' | sudo passwd --stdin <username>
  ```

---

## 💡 Catatan & Insight Pribadi

### Kenapa Pakai `/bin/sh` dan Bukan `/bin/bash`?

`/bin/sh` adalah **POSIX shell** — standar paling dasar. Di Ubuntu, `/bin/sh` biasanya symlink ke `dash`, yang lebih ringan dan cepat dari bash.

| Shell | Karakteristik |
|---|---|
| `/bin/sh` | POSIX-compliant, ringan, cepat |
| `/bin/bash` | Fitur lengkap, lebih berat |
| `/bin/dash` | POSIX-compliant, default `/bin/sh` di Debian/Ubuntu |
| `/bin/zsh` | Fitur modern, populer untuk interactive |

### Urutan Operasi yang Benar

```
1. useradd       → buat user
2. passwd        → set password
3. usermod -s    → set shell
4. chage -d 0    → paksa ganti password
5. groupadd      → buat group
6. usermod -aG   → tambah user ke group
```

Alasan `chage -d 0` dilakukan **setelah** set password: kalau dilakukan sebelum, field *"last password change"* di `/etc/shadow` di-set ke 0, tapi password masih kosong (`!!`) — sistem bisa bingung. Setelah password di-set, baru paksa ganti.

### Perbedaan `chage -d 0` vs `passwd -e`

| Perintah | Efek |
|---|---|
| `chage -d 0 user` | Set tanggal perubahan password ke epoch 0 → paksa ganti |
| `passwd -e user` | Expire password user → paksa ganti |

Keduanya sama-sama efektif. `passwd -e` lebih singkat, tapi `chage -d 0` lebih eksplisit.

### Best Practice

- **Selalu pakai `-aG`** untuk menambahkan user ke group, jangan `-G`.
- **Test dengan `su - user`** setelah konfigurasi untuk memastikan semuanya bekerja.
- **Cek dengan `id`** untuk memverifikasi keanggotaan group.
- **Verifikasi sebelum grading** — jangan langsung grading tanpa cek hasil.

### Verifikasi Sebelum Grading

Biasakan menjalankan ini sebelum `nusactl grade`:

```bash
sudo chage -l <username>       # cek aging
sudo passwd -S <username>      # cek status password (harus P)
id <username>                  # cek group
grep <username> /etc/passwd    # cek shell
```

Kalau semua sudah benar, baru grading. Ini mencegah skor rendah karena satu case kecil yang terlewat.

### Kasus Nyata

- **Onboarding user baru:** User dibuat dengan password sementara, dipaksa ganti saat login pertama.
- **Akun guest:** Dibuat dengan shell `/bin/sh` dan group khusus untuk akses terbatas.
- **Akun service:** Dibuat dengan shell `/sbin/nologin` karena tidak untuk login interaktif.
- **Akun kolaborasi:** User ditambahkan ke group khusus (seperti `mentor`) untuk akses file bersama.

### Pelajaran Kunci dari Quiz Ini

1. **`useradd`** + **`passwd`** + **`usermod`** + **`chage`** + **`groupadd`** = kombinasi lengkap untuk manajemen user.
2. **Urutan operasi** penting — jangan sampai `chage -d 0` dijalankan sebelum password di-set.
3. **`-aG` vs `-G`** — kesalahan klasik yang bisa menghapus group lama user.
4. **Verifikasi dengan `chage -l`** — cara paling cepat memastikan password aging sudah benar.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `useradd <user>` | Buat user baru |
| `useradd -m <user>` | Buat user + home directory |
| `passwd <user>` | Set password user |
| `usermod -s /bin/sh <user>` | Set default shell |
| `usermod -aG <group> <user>` | Tambah user ke supplementary group |
| `chage -d 0 <user>` | Paksa ganti password saat login |
| `chage -l <user>` | Lihat info aging |
| `passwd -S <user>` | Cek status password |
| `groupadd <group>` | Buat group baru |
| `groupadd -f <group>` | Buat group (idempotent) |
| `id <user>` | Lihat UID, GID, dan group user |
| `groups <user>` | Daftar group user |
| `grep <user> /etc/passwd` | Cek user di `/etc/passwd` |
| `grep <user> /etc/shadow` | Cek hash & aging di `/etc/shadow` |
| `grep <group> /etc/group` | Cek group di `/etc/group` |
| `su - <user>` | Login sebagai user |

---

## 📌 Kesimpulan

Lab ini adalah **quiz gabungan** dari semua materi Lab 12 — user, password, shell, group, dan password aging. Meskipun sederhana, quiz ini menguji pemahaman tentang:

- Urutan operasi yang benar saat membuat user dengan konfigurasi lengkap.
- Perbedaan `usermod -aG` dan `usermod -G` yang krusial.
- Bagaimana `chage -d 0` memaksa user mengganti password.
- Cara memverifikasi hasil dengan `id`, `grep`, dan `chage -l`.

Kemampuan ini adalah **inti dari manajemen user** di Linux dan akan terus dipakai di dunia kerja, terutama saat onboarding user baru atau mengelola akses di server produksi.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
