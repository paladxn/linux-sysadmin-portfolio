# Lab 13.5 — Quiz: File Permissions, Ownership & ACLs

**Course:** Linux System Administration (Adinusa)
**Topic:** Quiz — ownership, permissions, dan ACL
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Mengubah ownership secara **rekursif** dengan `chown -R`
- Mengatur permission ketat untuk owner dan group dengan `chmod`
- Memberikan akses **read-only** ke user spesifik menggunakan ACL
- Mengonfigurasi **default ACL** agar file baru mewarisi aturan akses

---

## 📘 Skenario

Sebagai sysadmin di sebuah creative agency, saya diminta menyelesaikan konfigurasi akses untuk **Design Collaboration Project**.

Semua file dan direktori sudah disiapkan. Tugas saya:

1. Set ownership ke `designer1:creative_team`
2. Set permission `770` pada `/projects` dan `/projects/design_collab`
3. Beri user `reviewer1` akses **read-only** via ACL
4. Set **default ACL** agar file baru otomatis memberi `reviewer1` akses read-only

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-013-5
```

---

## 🧪 Quiz Task

### Soal

Konfigurasi `/projects/design_collab` sehingga:

1. Owner = `designer1`, group = `creative_team` (rekursif).
2. Permission: owner & group full access, others no access.
3. User `reviewer1` punya akses **read-only** ke direktori dan semua isinya.
4. **Default ACL** — file/direktori baru otomatis memberi `reviewer1` akses read-only.

### ✅ Solusi

```bash
# 1. Ubah ownership rekursif
$ sudo chown -R designer1:creative_team /projects/design_collab

# 2. Set permission rekursif: owner & group rwx, others none
$ sudo chmod -R 770 /projects/design_collab

# 3. Set permission /projects juga ke 770 (parent directory)
$ sudo chmod 770 /projects

# 4. ACL read-only untuk reviewer1 (rekursif)
$ sudo setfacl -R -m u:reviewer1:r /projects/design_collab

# 5. Default ACL agar file baru mewarisi akses reviewer1
$ sudo setfacl -d -m u:reviewer1:r /projects/design_collab
```

### 🔍 Penjelasan Opsi Penting

**`chmod -R 770`:**
- `7` → owner: read + write + execute
- `7` → group: read + write + execute
- `0` → others: tidak ada akses

**`setfacl -R -m u:reviewer1:r`:**

| Bagian | Arti |
|---|---|
| `-R` | Rekursif ke semua isi direktori |
| `-m` | Modify — tambah/ubah ACL |
| `u:reviewer1` | User `reviewer1` |
| `r` | Read-only — tanpa execute |

**`setfacl -d -m u:reviewer1:r`:**

| Bagian | Arti |
|---|---|
| `-d` | **Default** ACL — berlaku untuk file baru di dalam direktori |
| `-m` | Modify |
| `u:reviewer1:r` | User `reviewer1`, read-only |

---

## 🔍 Verifikasi

### 1. Cek ownership

```bash
$ ls -l /projects/design_collab
drwxrwx---+ ... designer1 creative_team ... /projects/design_collab
```

Owner = `designer1`, group = `creative_team`. ✅

### 2. Cek permission

```bash
$ ls -ld /projects /projects/design_collab
drwxrwx---+ ... /projects
drwxrwx---+ ... /projects/design_collab
```

Keduanya `770` = `rwxrwx---`. Others tidak punya akses. ✅

### 3. Cek ACL

```bash
$ getfacl /projects/design_collab
```

Output:

```
# file: projects/design_collab
# owner: designer1
# group: creative_team
user::rwx
user:reviewer1:r--
group::rwx
mask::rwx
other::---
default:user::rwx
default:user:reviewer1:r--
default:group::rwx
default:mask::rwx
default:other::---
```

**Perhatikan:**
- Baris `user:reviewer1:r--` → ACL aktif untuk reviewer1 (read-only).
- Baris `default:user:reviewer1:r--` → default ACL untuk file baru.

### 4. Uji default ACL — buat file baru

```bash
$ cd /projects/design_collab
$ sudo -u designer1 touch file_baru.txt
$ getfacl file_baru.txt
```

Output:

```
user::rw-
user:reviewer1:r--
group::rw-
mask::rw-
other::---
```

File baru otomatis punya ACL `user:reviewer1:r--`. ✅

---

## 🐛 Troubleshooting & Kesalahan

- **Case 2 gagal — `/projects` permission salah:** Grading memeriksa `/projects` juga, bukan hanya `/projects/design_collab`. Pastikan keduanya di-set `770`. Solusi: `sudo chmod 770 /projects`.

- **Case 4 gagal — ACL pakai `r-x` padahal minta `r`:** Grading memeriksa **"read access"** murni. `r-x` dianggap salah karena ada execute yang tidak diminta. Solusi: gunakan `r` (huruf kecil), bukan `rX` atau `r-x`.

- **`chown -R` gagal "Operation not permitted":** Hanya root yang bisa `chown`. Pastikan pakai `sudo`.

- **Setelah `chmod -R 770`, file jadi tidak bisa diakses reviewer1:** Karena `chmod` menghapus permission `others`. ACL yang sudah ada mungkin terpengaruh **mask**. Cek dengan `getfacl`.

- **Default ACL tidak bekerja:** Pastikan direktori target punya permission `x` untuk user/group yang di-set. Kalau direktori tidak punya `x`, file baru mungkin tidak dibuat.

- **ACL hilang setelah `chmod`:** Di beberapa sistem, `chmod` bisa memodifikasi `mask` ACL — yang bisa mempengaruhi permission efektif. Urutan yang disarankan: **`chmod` dulu, baru `setfacl`**.

- **`getfacl` menampilkan `# effective: r--`:** Ini artinya mask membatasi permission. Cek `mask::` di output.

- **Default ACL tidak berlaku untuk file yang sudah ada:** Default ACL hanya untuk file **baru**. File lama harus di-set ulang dengan `setfacl -R`.

- **User `reviewer1` tidak ada:** Pastikan user-nya sudah dibuat di sistem. Cek dengan `id reviewer1`.

- **Typo `>` di akhir perintah:** Sering terjadi saat copy-paste. Kalau muncul prompt `>`, tekan **CTRL + C**.

---

## 💡 Catatan & Insight Pribadi

### Perbedaan `r`, `r-x`, dan `rX`

| Notasi | Arti | Kapan dipakai |
|---|---|---|
| `r` | Read-only | User hanya perlu baca |
| `r-x` | Read + execute | User perlu `cd` atau jalankan |
| `rX` | Read + execute **hanya jika** objek sudah punya execute | Direktori + file campuran |

**Pelajaran dari quiz ini:** Grading meminta **read access** — jadi gunakan `r`, bukan `rX`. Kalau memakai `rX`, direktori akan dapat `r-x` — yang menurut grading kelebihan.

**Kapan `rX` berguna?** Kalau kita ingin memberi akses ke direktori (butuh `x` untuk `cd`) **dan** file-file di dalamnya (cukup `r`). `rX` otomatis membedakan keduanya. Tapi kalau grading minta spesifik `r`, ikuti apa yang diminta.

### Perbedaan ACL Aktif vs Default

| Tipe | Perintah | Berlaku untuk |
|---|---|---|
| **Aktif** | `setfacl -m` | File/direktori yang sudah ada |
| **Default** | `setfacl -d -m` | File baru di dalam direktori |

Untuk cakupan lengkap, keduanya perlu di-set:
- `-R -m` untuk semua file yang **sudah ada**.
- `-d -m` untuk file yang **akan dibuat**.

### Urutan yang Disarankan

```
1. chown -R    → set ownership dulu
2. chmod -R    → set permission dasar
3. chmod       → set permission parent directory
4. setfacl -R  → tambah ACL untuk file yang ada
5. setfacl -d  → set default ACL untuk file baru
```

Alasan: `chmod` bisa memodifikasi `mask` ACL. Kalau `chmod` dilakukan setelah `setfacl`, hasilnya bisa berubah.

### Kapan Pakai `chmod` vs `setfacl`?

| Kebutuhan | Pakai |
|---|---|
| Permission untuk owner & group | `chmod` |
| Permission untuk user spesifik | `setfacl` |
| Permission default untuk file baru | `setfacl -d` |
| Hapus permission others | `chmod o=` |

### Kasus Nyata

- **Tim kreatif di agency:** File project di-share ke tim desainer (`creative_team`) dan reviewer eksternal (`reviewer1`) yang hanya perlu baca.
- **Kolaborasi klien:** Klien butuh akses read-only ke folder project tanpa bisa mengubah file.
- **Backup & audit:** Tim audit butuh akses baca ke semua file tanpa bisa mengubah.

### Pelajaran Kunci dari Quiz Ini

1. **`chown -R`** mengubah ownership secara rekursif.
2. **`chmod 770`** memberi akses penuh owner & group, tanpa others.
3. **`chmod 770 /projects`** — parent directory juga dicek grading, jangan lupa.
4. **`setfacl -R -m`** memberi ACL ke semua file yang ada.
5. **`setfacl -d -m`** memastikan file baru mewarisi ACL.
6. **`r` vs `rX`** — baca soal dengan teliti. Kalau minta "read access", pakai `r`.
7. **Selalu baca log grading** kalau skor tidak 100 — log menunjukkan case mana yang gagal.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `chown -R user:group dir/` | Ubah ownership rekursif |
| `chmod -R 770 dir/` | Permission rekursif |
| `chmod 770 /projects` | Permission parent directory |
| `setfacl -R -m u:user:r dir/` | ACL rekursif untuk user |
| `setfacl -d -m u:user:r dir/` | Default ACL |
| `setfacl -m g:group:rw dir/` | ACL untuk group |
| `getfacl dir/` | Lihat ACL |
| `getfacl -R dir/` | Lihat ACL rekursif |

**Konversi permission:**

| Symbolic | Octal |
|---|---|
| `rwxrwx---` | 770 |
| `rwxr-x---` | 750 |
| `rw-rw----` | 660 |
| `rw-r-----` | 640 |

---

## 📌 Kesimpulan

Lab ini adalah **quiz gabungan** dari semua materi permissions (13.1–13.4). Meskipun terlihat sederhana, tugasnya menguji pemahaman tentang:

- Urutan operasi ownership → permission → ACL.
- Perbedaan ACL aktif vs default.
- Penggunaan `r` vs `rX` — baca soal dengan teliti.
- Verifikasi dengan `ls -l` dan `getfacl`.
- Parent directory (`/projects`) juga diperiksa grading.

Kemampuan ini adalah **inti dari keamanan file di Linux**. Di server produksi, konfigurasi seperti ini sering dipakai untuk share folder antar tim, memberikan akses baca ke auditor, atau mengatur folder kolaborasi proyek.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
