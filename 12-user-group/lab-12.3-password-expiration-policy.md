# Lab 12.3 — Manajemen Password Expiration Policy

**Course:** Linux System Administration (Adinusa)
**Topic:** Password aging, `chage`, dan `/etc/login.defs`
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Mengonfigurasi **password aging policy** untuk user tertentu dengan `chage`
- Memaksa user mengganti password saat login berikutnya
- Melihat dan memverifikasi kebijakan password aging
- Menetapkan **default policy system-wide** melalui `/etc/login.defs`
- Memahami bagaimana policy ini memengaruhi user baru

---

## 📘 Konsep Dasar

| Konsep | Penjelasan |
|---|---|
| **Password Aging** | Kebijakan yang mengatur kapan password harus diganti, berapa lama valid, dan kapan user diberi peringatan |
| **`chage`** | Utility untuk mengelola password aging per user |
| **`/etc/login.defs`** | File konfigurasi default untuk pembuatan user baru, termasuk default aging |
| **`/etc/shadow`** | Tempat penyimpanan informasi aging (field ke-3 sampai ke-8) |

**Parameter aging:**

| Parameter | Arti |
|---|---|
| `PASS_MAX_DAYS` | Maksimum hari password valid sebelum harus diganti |
| `PASS_MIN_DAYS` | Minimum hari sebelum user boleh ganti password lagi |
| `PASS_WARN_AGE` | Jumlah hari peringatan sebelum password expired |

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-012-3
```

---

## 📘 Guided Example

### 1. Buat user `policy_user`

```bash
$ sudo useradd policy_user
$ sudo passwd policy_user
New password: adinusa
Retype new password: adinusa
```

### 2. Set password aging policy

```bash
$ sudo chage -M 30 -m 7 -W 5 policy_user
```

**Penjelasan opsi:**

| Opsi | Arti |
|---|---|
| `-M 30` | Password expired setelah 30 hari |
| `-m 7` | User tidak bisa ganti password lagi sebelum 7 hari |
| `-W 5` | Peringatan 5 hari sebelum expired |

### 3. Verifikasi konfigurasi

```bash
$ sudo chage -l policy_user
```

Contoh output:

```
Last password change                                    : Oct 28, 2025
Password expires                                        : Nov 27, 2025
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 7
Maximum number of days between password change          : 30
Number of days of warning before password expires       : 5
```

### 4. Paksa user ganti password saat login berikutnya

```bash
$ sudo chage -d 0 policy_user
```

**Penjelasan:** `-d 0` mengeset tanggal perubahan password terakhir ke **epoch 0** (1 Jan 1970), sehingga sistem menganggap password sudah sangat lama dan **harus segera diganti**.

### 5. Test dengan login sebagai user

```bash
$ su - policy_user
```

Sistem akan menampilkan:

```
You are required to change your password immediately (administrator enforced).
```

Kamu bisa mengganti password atau keluar dengan `Ctrl + C`.

### 6. Lihat ringkasan aging semua user

```bash
$ sudo cat /etc/shadow | cut -d: -f1,3,5
```

Menampilkan **username**, **last change** (hari sejak epoch), dan **max days**.

### 7. Set default policy system-wide

```bash
$ sudo vim /etc/login.defs
```

Cari dan ubah baris berikut:

```
PASS_MAX_DAYS   30
PASS_MIN_DAYS   7
PASS_WARN_AGE   5
```

### 8. Buat user baru untuk verifikasi

```bash
$ sudo useradd broky
$ sudo chage -l broky
```

Expected output:

```
Last password change                                    : Oct 29, 2025
Password expires                                        : Nov 28, 2025
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 7
Maximum number of days between password change          : 30
Number of days of warning before password expires       : 5
```

**Kesimpulan:** User baru `broky` otomatis mewarisi policy dari `/etc/login.defs`.

---

## 🧪 Practice Task

### Soal

1. Buat user `testpolicy` dengan password `adinusa`.
2. Set policy: max 60 hari, min 5 hari, warning 10 hari.
3. Verifikasi dengan `chage -l`.
4. Paksa user ganti password saat login berikutnya.
5. Ubah default `/etc/login.defs` menjadi max 90, min 10, warning 7.
6. Buat user `testpolicy2` dan verifikasi bahwa policy-nya mengikuti default baru.

### ✅ Solusi

```bash
# 1. Buat user
$ sudo useradd testpolicy
$ echo "adinusa" | sudo passwd --stdin testpolicy

# 2. Set policy
$ sudo chage -M 60 -m 5 -W 10 testpolicy

# 3. Verifikasi
$ sudo chage -l testpolicy

# 4. Paksa ganti password
$ sudo chage -d 0 testpolicy

# 5. Ubah default
$ sudo sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS   90/' /etc/login.defs
$ sudo sed -i 's/^PASS_MIN_DAYS.*/PASS_MIN_DAYS   10/' /etc/login.defs
$ sudo sed -i 's/^PASS_WARN_AGE.*/PASS_WARN_AGE   7/' /etc/login.defs

# 6. Buat user baru
$ sudo useradd testpolicy2
$ sudo chage -l testpolicy2
```

### 🔍 Hasil yang Diharapkan

`chage -l testpolicy2` akan menunjukkan:

```
Minimum number of days between password change          : 10
Maximum number of days between password change          : 90
Number of days of warning before password expires       : 7
```

---

## 🐛 Troubleshooting & Kesalahan

- **`chage -d 0` tidak memaksa ganti password:** Pastikan user login melalui PAM yang mendukung (biasanya `su -` atau login normal). Jika menggunakan `su` tanpa `-`, environment mungkin tidak memicu prompt. Gunakan `su - username`.

- **`/etc/login.defs` tidak berpengaruh pada user lama:** Perubahan di `login.defs` **hanya berlaku untuk user baru**. User yang sudah ada tetap memakai policy lamanya sampai diubah manual dengan `chage`.

- **User tidak bisa ganti password meskipun `-m 7`:** Jika user mencoba ganti password sebelum 7 hari, sistem akan menolak dengan pesan *"You must wait longer to change your password"*. Ini memang perilaku yang diinginkan.

- **`chage -l` tidak menampilkan output yang diharapkan:** Pastikan kamu menjalankan sebagai root atau dengan `sudo`. User biasa hanya bisa melihat policy miliknya sendiri.

- **`/etc/login.defs` syntax error:** Jika ada typo, `useradd` bisa gagal atau menggunakan nilai default. Selalu verifikasi dengan `useradd -D` untuk melihat default yang aktif.

- **Password policy terlalu ketat:** User cenderung menulis password di kertas atau memilih variasi lemah. Gunakan kebijakan yang wajar (misal 90 hari) dan edukasi user.

- **File `/etc/shadow` tidak bisa dibaca:** Permission harus `400` dan hanya root yang bisa baca. Jangan pernah mengubah permission-nya.

- **`chage -E` vs `chage -d`:** `-E` untuk **account expiration** (tanggal akun dinonaktifkan), sedangkan `-d` untuk **last password change**. Keduanya berbeda.

- **User baru tidak bisa login setelah `useradd`:** Karena password belum di-set. Field di `/etc/shadow` diisi `!!`. Jalankan `passwd username`.

- **`/etc/login.defs` tidak ada:** Di beberapa distro minimal, file ini mungkin tidak ada. Buat manual atau install paket `login`.

---

## 💡 Catatan & Insight Pribadi

### Kenapa Password Aging Penting?

- **Membatasi waktu** jika password bocor — attacker hanya punya jendela terbatas.
- **Kepatuhan** terhadap standar keamanan (PCI-DSS, ISO 27001, dll).
- **Tapi:** Kebijakan terlalu ketat bisa **backfire** — user memilih password lemah atau menuliskannya.

### Best Practice

| Parameter | Rekomendasi |
|---|---|
| `PASS_MAX_DAYS` | 90 hari (jangan 30, terlalu sering) |
| `PASS_MIN_DAYS` | 1–7 hari |
| `PASS_WARN_AGE` | 7–14 hari |
| Kompleksitas | Gunakan `pam_pwquality` |
| Riwayat password | Cegah pemakaian ulang dengan `remember=N` |

### Perbedaan `chage -d` dan `chage -E`

| Opsi | Fungsi |
|---|---|
| `-d 0` | Paksa ganti password saat login berikutnya |
| `-E YYYY-MM-DD` | Set tanggal akun expired |
| `-I N` | Set berapa hari setelah password expired sebelum akun dinonaktifkan |

### Kapan Menggunakan `chage -d 0`?

- Saat **onboarding** user baru — mereka harus segera ganti password default.
- Setelah **reset password** oleh admin.
- Saat **kebijakan keamanan** mengharuskan user mengganti password.

### Melihat Policy User Lain

```bash
$ sudo chage -l username
```

### Melihat Default System

```bash
$ useradd -D
```

Menampilkan default yang akan dipakai untuk user baru.

### Kasus Nyata

- **Server produksi:** Biasanya `PASS_MAX_DAYS` 90–180 hari, `PASS_MIN_DAYS` 1–7 hari, `PASS_WARN_AGE` 7 hari.
- **Akun service:** Biasanya tidak punya password aging (karena tidak login interaktif).
- **Akun admin:** Mungkin punya kebijakan lebih ketat.
- **Compliance:** Beberapa standar (misal PCI-DSS) mewajibkan ganti password setiap 90 hari.

### Pelajaran Kunci

1. **`chage`** mengubah policy per user.
2. **`/etc/login.defs`** mengubah default untuk user baru.
3. **`chage -d 0`** memaksa ganti password.
4. **`chage -l`** untuk verifikasi.
5. **Jangan terlalu ketat** — security vs usability.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `chage -M N -m N -W N user` | Set max, min, warning days |
| `chage -l user` | Lihat info aging |
| `chage -d 0 user` | Paksa ganti password saat login |
| `chage -E YYYY-MM-DD user` | Set account expiration |
| `chage -I N user` | Set inactive days setelah expired |
| `cat /etc/shadow \| cut -d: -f1,3,5` | Lihat ringkasan aging |
| `vim /etc/login.defs` | Edit default policy |
| `useradd -D` | Lihat default useradd |

---

## 📌 Kesimpulan

Lab ini memberikan pemahaman praktis tentang **password aging policy** — salah satu aspek keamanan sistem yang sering diabaikan pemula. Dengan `chage`, kita bisa mengontrol kapan password harus diganti, kapan user diberi peringatan, dan kapan akun dinonaktifkan. Sementara itu, `/etc/login.defs` memungkinkan kita menetapkan default yang konsisten untuk semua user baru.

Kemampuan mengelola password aging adalah keterampilan wajib sysadmin, terutama di lingkungan yang memiliki standar keamanan ketat. Namun, perlu diingat bahwa kebijakan yang terlalu ketat bisa menurunkan keamanan itu sendiri — karena user akan mencari cara untuk menghindarinya. Keseimbangan antara keamanan dan kemudahan adalah kunci.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
