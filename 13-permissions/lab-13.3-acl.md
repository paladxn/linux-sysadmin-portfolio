# Lab 13.3 — Managing File Access with ACL

**Course:** Linux System Administration (Adinusa)
**Topic:** Access Control List (ACL) untuk permission granular
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Memahami keterbatasan permission tradisional (user/group/others)
- Menginstal dan menggunakan paket **ACL**
- Memberikan permission **per user** tanpa mengubah owner atau group
- Membaca, memodifikasi, dan menghapus ACL dengan `setfacl` dan `getfacl`

---

## 📘 Konsep Dasar

### Kenapa Butuh ACL?

Permission tradisional Linux hanya punya **tiga kelompok**:

| Kelompok | Cakupan |
|---|---|
| User (owner) | Satu orang |
| Group | Satu group |
| Others | Semua sisanya |

**Masalahnya:** Bagaimana kalau kita ingin memberi akses ke **user tertentu** tanpa:
- Mengubah owner file?
- Memasukkan user itu ke group pemilik?
- Membuka akses ke "others"?

Contoh kasus: File milik `user1` ingin bisa dibaca oleh `user2` saja, tapi **tidak** oleh user lain. Permission tradisional tidak bisa menyelesaikan ini dengan bersih.

**Solusinya:** **ACL (Access Control List)** — permission tambahan di luar user/group/others.

### Cara Kerja ACL

ACL memungkinkan kita menambahkan **entry** ke file:

```
user::rw-          ← owner (setara permission tradisional)
user:user2:r--     ← ACL untuk user2 (hanya baca)
user:broky:r-x     ← ACL untuk broky (baca + eksekusi)
group::rw-         ← group (tradisional)
mask::rw-          ← batas maksimum untuk ACL & group
other::---         ← others (tradisional)
```

Tanda `+` di output `ls -l` menandakan file punya ACL:

```
-rw-rw----+ 1 user1 user1 29 ... confidential.txt
          ↑
       ada ACL
```

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-013-3
```

---

## 📘 Guided Example

### 1. Install paket ACL

```bash
$ sudo apt install acl -y
```

**Catatan:** Di beberapa distro, paket ini sudah terinstal default. Tapi Ubuntu minimal sering belum.

### 2. Buat dua user untuk testing

```bash
$ sudo useradd user1 -m -s /bin/bash
$ sudo useradd user2 -m -s /bin/bash
```

### 3. Buat file `confidential.txt` sebagai `user1`

```bash
$ sudo -u user1 touch /tmp/confidential.txt
$ sudo -u user1 bash -c 'echo "This is a confidential file belonging to user1" > /tmp/confidential.txt'
```

**Cek permission awal:**

```bash
$ ls -l /tmp/confidential.txt
-rw-rw-r-- 1 user1 user1 29 Oct 28 07:41 /tmp/confidential.txt
```

**Penjelasan opsi `sudo -u user1`:** Menjalankan perintah sebagai user `user1` (bukan sebagai root). Ini penting agar file dimiliki user1, bukan root.

### 4. Test akses awal dengan `user2`

```bash
$ sudo -u user2 cat /tmp/confidential.txt
This is a confidential file belonging to user1
```

**Kenapa bisa dibaca?** Karena permission `others` masih `r--` — siapapun bisa membaca.

### 5. Ubah permission agar benar-benar private

```bash
$ sudo chmod o= /tmp/confidential.txt
$ ls -l /tmp/confidential.txt
-rw-rw---- 1 user1 user1 29 Oct 28 07:41 /tmp/confidential.txt
```

**Penjelasan `chmod o=`:** Menghapus **semua** permission untuk `others`. Sekarang hanya owner dan group yang bisa akses.

### 6. Test ulang dengan `user2`

```bash
$ sudo -u user2 cat /tmp/confidential.txt
cat: /tmp/confidential.txt: Permission denied
```

`user2` sekarang **tidak bisa** membaca file — karena bukan owner, bukan anggota group, dan others tidak punya permission.

### 7. Beri akses baca ke `user2` via ACL

```bash
$ sudo setfacl -m u:user2:r /tmp/confidential.txt
```

**Penjelasan sintaks:**

| Bagian | Arti |
|---|---|
| `setfacl` | Perintah untuk set ACL |
| `-m` | **Modify** — tambah atau ubah entry |
| `u:user2:r` | **u**ser `user2` diberi **r**ead |
| `/tmp/confidential.txt` | File target |

**Efek:** `user2` bisa membaca file tanpa mengubah owner atau group.

### 8. Cek hasil ACL

```bash
$ getfacl /tmp/confidential.txt
```

Output:

```
# file: tmp/confidential.txt
# owner: user1
# group: user1
user::rw-
user:user2:r--
group::rw-
mask::rw-
other::---
```

**Breakdown:**

| Baris | Arti |
|---|---|
| `user::rw-` | Owner (user1) punya read + write |
| `user:user2:r--` | **ACL**: user2 hanya read |
| `group::rw-` | Group user1 punya read + write |
| `mask::rw-` | Batas maksimum untuk ACL & group |
| `other::---` | Others tidak punya akses |

### 9. Verifikasi akses `user2`

```bash
$ sudo -u user2 cat /tmp/confidential.txt
This is a confidential file belonging to user1
```

`user2` sekarang bisa membaca. ✅

### 10. Beri akses read + execute ke user `broky`

```bash
$ sudo setfacl -m u:broky:r-x /tmp/confidential.txt
$ getfacl /tmp/confidential.txt
```

Output:

```
user::rw-
user:broky:r-x
user:user2:r--
group::rw-
mask::rwx
other::---
```

**Perhatikan:** `mask::rwx` naik dari `rw-` ke `rwx` — karena `broky` punya `x`, mask harus mengizinkan `x`.

### 11. Hapus ACL untuk `broky`

```bash
$ sudo setfacl -x u:broky /tmp/confidential.txt
$ getfacl /tmp/confidential.txt
```

Output:

```
user::rw-
user:user2:r--
group::rw-
mask::rw-
other::---
```

ACL untuk `broky` sudah dihapus, tapi `user2` tetap.

---

## 🧪 Practice Task

### Soal

1. Buat file `/tmp/shared.txt` milik user `user1`.
2. Set permission agar hanya owner dan group yang bisa akses (hapus others).
3. Beri ACL read-only ke `user2`.
4. Beri ACL read+write ke user `editor` (buat dulu).
5. Verifikasi dengan `getfacl`.
6. Hapus ACL untuk `editor`, biarkan `user2` tetap.
7. Uji akses sebagai `user2` dan pastikan bisa baca.

### ✅ Solusi

```bash
# 1. Buat file sebagai user1
$ sudo -u user1 bash -c 'echo "Shared content" > /tmp/shared.txt'

# 2. Hapus permission others
$ sudo chmod o= /tmp/shared.txt

# 3. ACL read untuk user2
$ sudo setfacl -m u:user2:r /tmp/shared.txt

# 4. Buat user editor & beri read+write
$ sudo useradd editor -m
$ sudo setfacl -m u:editor:rw /tmp/shared.txt

# 5. Verifikasi
$ getfacl /tmp/shared.txt

# 6. Hapus ACL editor
$ sudo setfacl -x u:editor /tmp/shared.txt

# 7. Uji akses user2
$ sudo -u user2 cat /tmp/shared.txt
Shared content
```

### 🔍 Hasil yang Diharapkan

`getfacl /tmp/shared.txt` setelah langkah 6:

```
user::rw-
user:user2:r--
group::r--
mask::r--
other::---
```

---

## 🐛 Troubleshooting & Kesalahan

- **`setfacl: command not found`:** Paket `acl` belum terinstal. Install dulu:
  ```bash
  $ sudo apt install acl -y
  ```

- **`Operation not supported`:** Filesystem target tidak mendukung ACL. Cek dengan:
  ```bash
  $ mount | grep acl
  ```
  Kalau tidak ada `acl` di opsi mount, ACL tidak aktif. Umumnya ext4 modern sudah support ACL by default.

- **`+` muncul di `ls -l`:** Ini tanda file punya ACL. Kalau tidak ingin ada `+`, hapus semua ACL:
  ```bash
  $ sudo setfacl -b /tmp/confidential.txt
  ```

- **ACL user tidak bisa akses meskipun sudah di-set:** Cek **mask**. Kalau mask tidak mengizinkan permission yang diminta, ACL akan **dibatasi oleh mask**. Contoh: mask `rw-`, tapi ACL bilang `r-x` — hasil efektifnya `r--`.
  ```bash
  $ sudo setfacl -m m::rwx /tmp/confidential.txt
  ```

- **`sudo -u user1` gagal "unknown user":** User belum dibuat. Pastikan `useradd` sudah dijalankan dan user benar-benar ada:
  ```bash
  $ id user1
  ```

- **`getfacl` menampilkan `getfacl: Removing leading '/' from absolute path names`:** Ini hanya warning — bukan error. `getfacl` menampilkan path tanpa `/` di depan untuk konsistensi. Bisa diabaikan.

- **Mengubah permission dengan `chmod` menghapus ACL:** Di beberapa sistem lama, `chmod` bisa memodifikasi mask ACL. Di ext4 modern, `chmod` hanya mengubah traditional permission, ACL tetap. Tapi selalu cek dengan `getfacl` setelah `chmod`.

- **`setfacl -x` gagal karena user tidak ada di ACL:** `-x` hanya menghapus kalau entry-nya ada. Kalau user tidak ada di ACL, perintah tetap sukses tapi tidak ada yang berubah.

- **ACL untuk direktori:** ACL di direktori berlaku untuk file di dalamnya **hanya jika** ada **default ACL**. Kalau tidak, ACL hanya berlaku untuk direktori itu sendiri. Untuk mengatur default ACL:
  ```bash
  $ sudo setfacl -d -m u:user2:r /tmp/shared_dir/
  ```

- **Permission tidak langsung berlaku:** Kadang butuh logout & login ulang untuk user yang terpengaruh, terutama kalau group-nya berubah. Untuk ACL user, biasanya langsung berlaku.

---

## 💡 Catatan & Insight Pribadi

### Kapan Pakai ACL vs Permission Tradisional?

| Kebutuhan | Solusi |
|---|---|
| Owner punya akses penuh | `chmod 644` / `755` |
| Group berbagi file | `chmod 664` / `775` |
| **Satu user spesifik** butuh akses | **ACL** |
| **Beberapa user berbeda** dengan permission berbeda | **ACL** |
| Akses default untuk file baru di direktori | **Default ACL** |
| Keamanan tingkat lanjut (MAC) | SELinux / AppArmor |

**Aturan praktis:** Kalau butuh lebih dari 3 kelompok akses, pakai ACL.

### Struktur ACL

```
user::rw-          → owner (setara permission tradisional user)
user:user2:r--     → ACL user tambahan
user:broky:r-x     → ACL user lain
group::rw-         → group owner (setara permission tradisional group)
group:dev:r--      → ACL group tambahan
mask::rw-          → batas maksimum
other::---         → others (setara permission tradisional others)
```

**Mask** membatasi permission efektif dari semua ACL user & group (kecuali owner & others). Kalau mask `rw-`, maka ACL `r-x` akan efektif jadi `r--`.

### Perintah `setfacl` yang Sering Dipakai

| Perintah | Fungsi |
|---|---|
| `setfacl -m u:user:rw file` | Tambah/ubah ACL user |
| `setfacl -m g:group:rx file` | Tambah/ubah ACL group |
| `setfacl -m m::rw file` | Set mask |
| `setfacl -x u:user file` | Hapus ACL user tertentu |
| `setfacl -b file` | Hapus **semua** ACL |
| `setfacl -d -m u:user:rw dir/` | Set **default** ACL di direktori |
| `setfacl -R -m u:user:rw dir/` | Set ACL rekursif |

### Perintah `getfacl` yang Sering Dipakai

| Perintah | Fungsi |
|---|---|
| `getfacl file` | Tampilkan ACL file |
| `getfacl -R dir/` | Tampilkan ACL rekursif |
| `getfacl -t file` | Tampilkan tanpa komentar (tabular) |

### Simbol `+` di `ls -l`

```
-rw-rw----+ 1 user1 user1 29 ... confidential.txt
          ↑
       ada ACL
```

Kalau `+` tidak muncul, berarti file **tidak punya ACL**.

### Backup & Restore ACL

Bisa backup ACL ke file, lalu restore:

```bash
# Backup
$ getfacl -R /path > acl_backup.txt

# Restore
$ setfacl --restore=acl_backup.txt
```

Berguna sebelum migrasi atau cloning direktori.

### Kapan ACL Berguna di Dunia Nyata?

- **Shared server:** Beberapa developer butuh akses ke folder bersama, tapi dengan permission berbeda-beda.
- **Web server:** File `/var/www/html` diakses oleh user `www-data` dan user deploy.
- **Backup:** User `backup` butuh baca file sensitif tanpa harus jadi owner.
- **CI/CD:** Runner butuh akses ke file tertentu tanpa mengubah ownership.
- **Log server:** User `analyst` butuh baca log tanpa akses tulis.

### Keterbatasan ACL

- **Tidak semua filesystem support:** ext4 dan xfs modern support, tapi beberapa filesystem lawas (FAT32, exFAT) tidak.
- **Bisa membingungkan:** Terlalu banyak ACL membuat permission sulit dilacak.
- **Bukan security tool sejati:** Untuk keamanan tingkat lanjut, gunakan SELinux atau AppArmor.
- **Kompleksitas maintenance:** Kalau user dihapus, ACL-nya tetap ada (harus dibersihkan manual).

### Pelajaran Kunci dari Lab Ini

1. **ACL menyelesaikan masalah permission granular** yang tidak bisa diselesaikan permission tradisional.
2. **`mask` membatasi permission efektif** — jangan lupa cek kalau ACL tidak bekerja.
3. **`+` di `ls -l` menandakan file punya ACL.**
4. **`chmod o=`** adalah cara cepat untuk membuat file "private".
5. **Default ACL** berguna untuk direktori shared — file baru otomatis mewarisi ACL.
6. **ACL bukan pengganti permission tradisional** — kombinasi keduanya.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `sudo apt install acl -y` | Install paket ACL |
| `sudo setfacl -m u:user:r file` | Tambah ACL user (read) |
| `sudo setfacl -m u:user:rw file` | Tambah ACL user (read+write) |
| `sudo setfacl -m u:user:r-x file` | Tambah ACL user (read+execute) |
| `sudo setfacl -m g:group:r file` | Tambah ACL group |
| `sudo setfacl -x u:user file` | Hapus ACL user tertentu |
| `sudo setfacl -b file` | Hapus semua ACL |
| `sudo setfacl -d -m u:user:r dir/` | Set default ACL di direktori |
| `sudo setfacl -R -m u:user:r dir/` | Set ACL rekursif |
| `getfacl file` | Lihat ACL |
| `getfacl -R dir/` | Lihat ACL rekursif |
| `chmod o= file` | Hapus semua permission others |
| `sudo -u user cmd` | Jalankan perintah sebagai user lain |

---

## 📌 Kesimpulan

Lab ini membuka pintu ke **manajemen permission tingkat lanjut** dengan ACL. Kalau permission tradisional (Lab 13.1 dan 13.2) hanya bisa mengatur tiga kelompok, ACL memungkinkan kita memberikan akses **per user** dan **per group** tanpa mengubah struktur ownership.

Konsep ini sangat berguna di server produksi, terutama untuk:
- File share yang perlu akses granular
- File konfigurasi yang diakses beberapa service
- Direktori kolaborasi dengan permission berbeda per orang

Kemampuan menguasai ACL adalah **nilai tambah besar** untuk sysadmin, karena tidak semua orang paham konsep ini. Yang paling penting dipahami: **`mask`** adalah kunci utama — kalau ACL tidak bekerja seperti yang diharapkan, cek dulu mask-nya.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
