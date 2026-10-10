# Lab 13.6 — Quiz: File Permissions & Ownership

**Course:** Linux System Administration (Adinusa)
**Topic:** Quiz — ownership, SGID, immutable, chmod, ACL
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Mengubah ownership (user & group) secara **rekursif** dengan `chown -R`
- Menambahkan **SGID bit** pada file dengan `chmod g+s`
- Menghapus file yang memiliki atribut **immutable** (`chattr -i`)
- Mengatur permission file dengan `chmod` (numeric)
- Memberikan akses tulis spesifik ke user dengan **ACL** (`setfacl`)

---

## 📘 Skenario

Saya diminta menyelesaikan konfigurasi ownership, permission, dan ACL pada direktori `~/lab136`. Beberapa file memiliki atribut khusus yang perlu ditangani.

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-013-6
```

> **Catatan:** Ganti `<username>` dengan username Adinusa masing-masing.

---

## 🧪 Quiz Task

### Soal

1. Ubah ownership `~/lab136/change_me` menjadi `<username>:student` secara **rekursif**.
2. Tambahkan **hanya SGID bit** pada file `~/lab136/answer/perm` (tanpa mengubah permission lain).
3. Hapus file `~/lab136/answer/garbage` (file ini mungkin memiliki atribut immutable).
4. Set permission `~/lab136/answer/permissions.txt` menjadi:
   - Owner: `rwx`
   - Group: `r-x`
   - Others: `---`
5. Tambahkan ACL pada `~/lab136/answer/acl_mnop.txt` sehingga user `<username>` memiliki akses **write**.

### ✅ Solusi

```bash
# 1. Ubah ownership rekursif
$ sudo chown -R `<username>`:student ~/lab136/change_me

# 2. Tambah SGID bit pada file perm (tanpa mengubah permission lain)
$ sudo chmod g+s ~/lab136/answer/perm

# 3. Hapus file garbage (hilangkan immutable dulu jika ada)
$ sudo chattr -i ~/lab136/answer/garbage 2>/dev/null
$ sudo rm ~/lab136/answer/garbage

# 4. Set permission permissions.txt ke 750 (rwxr-x---)
$ sudo chmod 750 ~/lab136/answer/permissions.txt

# 5. Tambah ACL write untuk user
$ sudo setfacl -m u:<username>:w ~/lab136/answer/acl_mnop.txt
```

### 🔍 Penjelasan Opsi Penting

**`chown -R <username>:student`**
- `-R` → rekursif ke seluruh isi direktori.
- Format `user:group` → ubah owner dan group sekaligus.

**`chmod g+s`**
- `g+s` → tambahkan **SGID** pada group permission.
- File executable dengan SGID akan berjalan dengan group pemilik file.
- Pada direktori, SGID membuat file baru mewarisi group direktori.
- **Tidak mengubah** permission lain.

**`chattr -i`**
- Menghapus atribut **immutable**.
- File immutable tidak bisa dihapus/diubah bahkan oleh root.
- Wajib dijalankan sebelum `rm` jika file memiliki atribut `i`.

**`chmod 750`**
- `7` → owner: read + write + execute
- `5` → group: read + execute
- `0` → others: tidak ada akses

**`setfacl -m u:<username>:w`**
- `-m` → modify ACL
- `u:<username>` → user target
- `w` → write permission (tanpa read/execute)

---

## 🔍 Verifikasi

### 1. Cek ownership `change_me`

```bash
$ ls -lR ~/lab136/change_me | head
```

Owner harus `<username>`, group `student`.

### 2. Cek SGID pada `perm`

```bash
$ ls -l ~/lab136/answer/perm
-rwxr-sr-x ... perm
```

Perhatikan huruf **`s`** di posisi group execute.

### 3. Cek file `garbage` sudah terhapus

```bash
$ ls ~/lab136/answer/garbage
ls: cannot access '.../garbage': No such file or directory
```

### 4. Cek permission `permissions.txt`

```bash
$ ls -l ~/lab136/answer/permissions.txt
-rwxr-x--- ... permissions.txt
```

### 5. Cek ACL pada `acl_mnop.txt`

```bash
$ getfacl ~/lab136/answer/acl_mnop.txt
```

Output harus menampilkan:

```
user:<username>:w-
```

---

## 🐛 Troubleshooting & Kesalahan

- **`chown -R` gagal "No such file or directory":** Penyebab paling umum adalah menulis `<username>` **dengan** tanda kurung siku di terminal. Tanda `<` di bash adalah input redirection, jadi shell mengira kamu membaca file. Solusi: ketik username **tanpa** `<>`.

- **`chattr -i` gagal "Operation not supported":** Filesystem tidak mendukung atribut (misal FAT32). Lab biasanya pakai ext4, jadi seharusnya bisa.

- **File `garbage` tidak bisa dihapus:** Pastikan sudah `chattr -i` terlebih dahulu. Jika masih gagal, cek permission direktori parent.

- **SGID tidak muncul di `ls -l`:** Pastikan `chmod g+s` dijalankan sebagai root atau owner file. Cek dengan `ls -l` — harus ada huruf `s` di posisi group execute.

- **ACL tidak berlaku:** Cek `mask` di output `getfacl`. Jika mask tidak mengizinkan `w`, ACL write akan diblokir. Set mask dengan `setfacl -m m::rw`.

- **`chown -R` gagal "Operation not permitted":** Hanya root yang bisa `chown`. Gunakan `sudo`.

- **User `<username>` tidak ada:** Pastikan user sudah dibuat di sistem. Cek dengan `id <username>`.

- **Permission `permissions.txt` salah:** Pastikan menggunakan **numeric mode** `750`, bukan symbolic. `750` = `rwxr-x---`.

---

## 💡 Catatan & Insight Pribadi

### SGID pada File vs Direktori

| Tipe | Efek SGID |
|---|---|
| **File executable** | Dijalankan dengan group pemilik file (bukan group user yang menjalankan) |
| **Direktori** | File baru di dalamnya mewarisi group direktori (bukan group user pembuat) |

SGID sering dipakai untuk folder kolaborasi tim — semua file baru otomatis masuk ke group yang sama.

### Immutable — Proteksi Ekstra

File dengan atribut `i` (immutable) **tidak bisa diubah, di-rename, atau dihapus** bahkan oleh root. Untuk menghapusnya, harus `chattr -i` dulu. Ini berguna untuk file konfigurasi kritis, tapi bisa merepotkan kalau lupa.

### ACL — Write-Only Access

ACL `u:user:w` memberikan **hanya write** tanpa read atau execute. Ini jarang dipakai, tapi berguna untuk skenario seperti:
- User hanya boleh menulis ke file log, tidak boleh membacanya.
- User hanya boleh menambahkan data ke file antrian.

Biasanya ACL write diberikan bersama read (`rw`). Tapi lab ini spesifik meminta `w` saja.

### Urutan Operasi yang Disarankan

```
1. chown -R    → ownership dulu
2. chmod       → permission dasar
3. chmod g+s   → SGID jika perlu
4. chattr -i   → lepas immutable sebelum hapus
5. rm          → hapus file
6. setfacl     → ACL terakhir
```

### Pelajaran Kunci dari Quiz Ini

1. **`chown -R user:group`** mengubah owner & group rekursif.
2. **`chmod g+s`** menambahkan SGID tanpa mengubah permission lain.
3. **`chattr -i`** wajib sebelum menghapus file immutable.
4. **`chmod 750`** = `rwxr-x---`.
5. **`setfacl -m u:user:w`** memberi write-only access.
6. **Selalu verifikasi** dengan `ls -l` dan `getfacl`.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `chown -R user:group dir/` | Ubah owner & group rekursif |
| `chmod g+s file` | Tambah SGID |
| `chattr -i file` | Hapus atribut immutable |
| `chmod 750 file` | Permission `rwxr-x---` |
| `setfacl -m u:user:w file` | ACL write-only untuk user |
| `getfacl file` | Lihat ACL |
| `ls -l` | Lihat permission & SGID |

**Konversi permission:**

| Symbolic | Octal |
|---|---|
| `rwxr-x---` | 750 |
| `rwxr-sr-x` | 2755 (dengan SGID) |
| `rw-rw----` | 660 |
| `rw-------` | 600 |

---

## 📌 Kesimpulan

Lab ini adalah **quiz gabungan** dari topik ownership, permission, special permission (SGID), file attribute (immutable), dan ACL. Setiap langkah menguji pemahaman tentang:

- Perbedaan `chown`, `chmod`, `chattr`, dan `setfacl`.
- Efek SGID pada file.
- Cara melepas immutable sebelum menghapus.
- Memberi akses spesifik lewat ACL (write-only).

Kemampuan ini adalah **inti dari administrasi file di Linux** dan akan terus dipakai di dunia kerja, terutama saat mengelola file share, kolaborasi tim, dan keamanan file.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
