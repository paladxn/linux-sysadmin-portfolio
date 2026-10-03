# Lab 8.2 — Symbolic Link (Soft Link)

**Course:** Linux System Administration (Adinusa)
**Topic:** Symbolic link, perbedaan dengan hard link, dan broken link
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membuat file dan memeriksa **inode number**-nya
- Membuat **symbolic link** dan memverifikasi propertinya
- Membedakan **hard link** dan **soft link** dari inode number-nya
- Memahami apa yang terjadi ketika file asli dihapus tapi symbolic link masih ada

---

## 📘 Konsep Dasar

| Konsep | Penjelasan |
|---|---|
| **Symbolic Link (Symlink)** | File khusus yang **menunjuk ke path/nama file lain**, bukan ke inode. |
| **Broken / Dangling Link** | Symlink yang targetnya sudah tidak ada. Tetap ada sebagai file, tapi tidak bisa dibuka. |
| **Inode Symlink** | Symlink punya inode **sendiri** yang berbeda dari targetnya. |
| **Ukuran Symlink** | Ukuran symlink = panjang path target (dalam byte), bukan ukuran isi file. |

**Perbedaan kunci dengan hard link:**

| Aspek | Hard Link | Symbolic Link |
|---|---|---|
| Menunjuk ke | Inode | Path/nama file |
| Inode number | Sama dengan target | Berbeda |
| Lintas filesystem | ❌ | ✅ |
| Link ke direktori | ❌ | ✅ |
| Kalau target dihapus | Data tetap ada | Link jadi rusak |

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-008-2
```

---

## 📘 Guided Example

### 1. Masuk ke direktori lab82

```bash
$ cd ~/lab82
$ pwd
```

Expected output:

```
/home/student/lab82
```

### 2. Membuat file `original.txt`

```bash
$ echo "This is the original file" > original.txt
```

### 3. Cek inode number file asli

```bash
$ ls -li original.txt
```

Contoh output:

```
264533 -rw-r--r-- 1 student student 27 Aug 25 13:00 original.txt
```

Catat inode number-nya (contoh: `264533`).

### 4. Membuat soft link

```bash
$ ln -s original.txt link.txt
```

**Catatan:** Opsi `-s` (symbolic) inilah yang membedakan `ln` biasa (hard link) dengan `ln -s` (soft link).

### 5. Verifikasi soft link

```bash
$ ls -li original.txt link.txt
```

Contoh output:

```
264533 -rw-r--r-- 1 student student 27 Aug 25 13:00 original.txt
264534 lrwxrwxrwx 1 student student 12 Aug 25 13:05 link.txt -> original.txt
```

**Perhatikan:**

- **Inode number berbeda**: `264533` (asli) vs `264534` (symlink).
- Permission dimulai dengan `l` (menandakan link).
- Ada tanda panah `->` yang menunjuk ke target: `link.txt -> original.txt`.
- Ukuran `link.txt` adalah **12 byte** — yaitu panjang string `original.txt`, bukan ukuran isi file.

### 6. Test konsistensi data

```bash
$ echo "Added through original file" >> original.txt
$ cat link.txt
```

Expected output:

```
This is the original file
Added through original file
```

Symlink membaca **isi target** — jadi setiap perubahan di `original.txt` langsung terlihat lewat `link.txt`.

### 7. Uji broken link — **HANYA SETELAH GRADING**

> ⚠️ **PENTING:** Jalankan langkah ini **setelah** `nusactl grade`. Kalau dijalankan sebelum grading, grading akan gagal karena file `original.txt` sudah tidak ada.

```bash
$ rm original.txt
$ cat link.txt
```

Output:

```
cat: link.txt: No such file or directory
```

Symlink sekarang menjadi **broken link** — file `link.txt` masih ada (bisa dilihat dengan `ls`), tapi targetnya sudah tidak ada sehingga tidak bisa dibuka.

Verifikasi bahwa symlink-nya masih ada tapi rusak:

```bash
$ ls -li link.txt
264534 lrwxrwxrwx 1 student student 12 Aug 25 13:05 link.txt -> original.txt
```

Symlink tetap ada, tapi arah panahnya menunjuk ke file yang tidak ada. Coba:

```bash
$ ls -l link.txt
$ stat link.txt
```

Akan terlihat bahwa symlink-nya "dangling" (menggantung).

---

## 🧪 Practice Task

### Soal

1. Buat file `data.txt` dengan isi `Belajar symbolic link`.
2. Cek inode number-nya.
3. Buat symlink `alias.txt` yang menunjuk ke `data.txt`.
4. Verifikasi bahwa inode `alias.txt` berbeda dan ada tanda `->`.
5. Tambahkan baris `Baris kedua` ke `data.txt`.
6. Baca `alias.txt` — pastikan perubahan terlihat.
7. Hapus `data.txt`, lalu coba baca `alias.txt`. Catat hasilnya.
8. Buat ulang `data.txt` dengan isi baru, dan cek apakah `alias.txt` otomatis berfungsi lagi.

### ✅ Solusi

```bash
# 1. Buat file asli
$ echo "Belajar symbolic link" > ~/lab82/data.txt

# 2. Cek inode
$ ls -li ~/lab82/data.txt

# 3. Buat symlink
$ ln -s ~/lab82/data.txt ~/lab82/alias.txt

# 4. Verifikasi
$ ls -li ~/lab82/data.txt ~/lab82/alias.txt
# Inode berbeda, alias.txt punya tanda ->

# 5. Tambah data lewat file asli
$ echo "Baris kedua" >> ~/lab82/data.txt

# 6. Baca lewat symlink
$ cat ~/lab82/alias.txt
# Output:
# Belajar symbolic link
# Baris kedua

# 7. Hapus file asli, coba baca symlink
$ rm ~/lab82/data.txt
$ cat ~/lab82/alias.txt
# Output: cat: alias.txt: No such file or directory

# 8. Buat ulang file asli, cek symlink
$ echo "Data baru" > ~/lab82/data.txt
$ cat ~/lab82/alias.txt
# Output: Data baru
# Symlink "hidup" kembali karena targetnya sudah ada lagi
```

### 🔍 Insight dari Step 8

Ini poin penting: **symlink menyimpan path, bukan inode**. Selama ada file dengan path yang sama, symlink otomatis berfungsi lagi. Ini **berbeda** dengan hard link yang benar-benar terikat ke inode.

---

## 🐛 Troubleshooting & Kesalahan

- **Salah pakai `ln` tanpa `-s`:** Awalnya saya ketik `ln original.txt link.txt` tanpa `-s`, dan yang terbuat adalah **hard link**, bukan symbolic link. Akibatnya inode-nya sama, dan tidak ada tanda `->` di output `ls`. Solusinya: **selalu pakai `-s`** untuk symbolic link.

- **Symlink relatif vs absolut:** Saat membuat symlink dengan `ln -s original.txt link.txt`, path yang disimpan adalah **path relatif** (`original.txt`). Kalau symlink dipindah ke direktori lain, dia akan rusak karena target relatifnya tidak ditemukan. Untuk symlink yang tahan pindah, gunakan **path absolut**:
  ```bash
  $ ln -s /home/student/lab82/original.txt /home/student/lab82/link.txt
  ```

- **Grading gagal karena file asli sudah dihapus:** Karena penasaran, saya sempat menghapus `original.txt` sebelum grading. Akibatnya grading gagal karena memeriksa keberadaan `original.txt`. Pelajaran: **selalu ikuti urutan lab** — jangan lompat ke langkah "delete original file" sebelum grading.

- **Bingung bedakan hard link dan symlink di `ls -l`:** Output `ls -l` untuk symlink selalu dimulai dengan `l` (bukan `-`), dan ada tanda `->`. Kalau tidak ada tanda `->`, berarti itu bukan symlink.

- **Symlink tidak bisa dibuat ke path yang belum ada?** Sebenarnya **bisa**. `ln -s target link` tidak memvalidasi apakah `target` ada. Symlink bisa dibuat dulu, dan baru "hidup" saat target-nya muncul. Ini yang terjadi di practice task step 8.

- **`rm` pada symlink:** Menghapus symlink dengan `rm link.txt` hanya menghapus symlink-nya, **bukan** targetnya. Aman.

- **`cat` pada symlink yang broken:** Menghasilkan pesan *"No such file or directory"*. Untuk melihat symlink-nya sendiri (bukan targetnya), gunakan `ls -l` atau `readlink link.txt`.

---

## 💡 Catatan & Insight Pribadi

- **Symlink = "shortcut":** Analogi paling mudah: symlink itu seperti **shortcut di Windows** atau **alias di macOS**. Dia bukan file-nya sendiri, tapi penunjuk ke file lain.

- **Ukuran symlink:** `ls -l` menampilkan ukuran symlink sebagai panjang path target. Contoh: `original.txt` = 12 karakter, jadi ukuran symlink = 12 byte.

- **Cara membaca symlink tanpa membuka targetnya:**
  ```bash
  $ readlink link.txt
  original.txt
  ```

- **Cara mengetahui apakah sebuah file adalah symlink:**
  ```bash
  $ test -L link.txt && echo "symlink"
  $ file link.txt
  ```

- **Symlink ke symlink:** Bisa dibuat berlapis-lapis (symlink → symlink → file). Tapi ini membuat rantai menjadi panjang dan sulit dilacak. Hindari kecuali sangat diperlukan.

- **Broken link tetap "ada":** `rm link.txt` tetap bisa dijalankan, dan file-nya masih muncul di `ls`. Yang rusak adalah **tujuan**-nya, bukan symlink itu sendiri. Ini yang membedakan dengan hard link — hard link tidak pernah "broken", karena mereka adalah inode itu sendiri.

- **Kapan symlink lebih baik dari hard link?**
  - Menunjuk ke direktori (hard link tidak bisa).
  - Menunjuk lintas filesystem (hard link tidak bisa).
  - Ingin tahu targetnya secara eksplisit (mudah dilacak dengan `readlink`).
  - Ingin fleksibilitas: target bisa diganti tanpa mengubah link.

- **Kapan hard link lebih baik?**
  - Butuh efisiensi storage untuk file besar (misal backup snapshot).
  - Tidak ingin link rusak kalau file asli dihapus.
  - Tidak peduli target path berubah.

- **Kasus nyata:** Symlink banyak dipakai di konfigurasi server:
  - `/etc/nginx/sites-enabled/` berisi symlink ke `sites-available/`.
  - `/usr/lib/...` sering berisi symlink ke versi library spesifik.
  - `python3` sering symlink ke `python3.12` atau versi tertentu.
  - Mengganti versi aplikasi cukup dengan mengganti symlink-nya.

- **`find -L` vs `find`:** `find` tanpa `-L` tidak mengikuti symlink, sedangkan `find -L` mengikuti. Berguna saat mencari file dalam sistem yang penuh symlink.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `ln -s source target` | Buat symbolic link |
| `ln source target` | Buat hard link |
| `ls -li` | Lihat inode + info file |
| `readlink link` | Lihat path target symlink |
| `file link` | Identifikasi tipe file (symlink atau bukan) |
| `test -L link` | Cek apakah file adalah symlink |
| `stat file` | Info lengkap termasuk inode |
| `rm link` | Hapus symlink (bukan targetnya) |

---

## 📌 Kesimpulan

Lab ini melengkapi pemahaman dari Lab 8.1 tentang hard link. Kalau hard link mengikat nama ke **inode**, symbolic link mengikat nama ke **path**. Perbedaan filosofis ini punya konsekuensi praktis yang besar: symlink bisa lintas filesystem, bisa menunjuk direktori, tapi bisa "rusak" kalau targetnya hilang. Sebaliknya, hard link lebih "kokoh" tapi terbatas pada satu filesystem dan tidak bisa menunjuk direktori.

Memahami keduanya penting untuk sysadmin, terutama saat menangani:
- Konfigurasi server (Nginx, Apache, PHP).
- Manajemen versi library.
- Backup dan snapshot.
- Migrasi data antar direktori tanpa mengubah path yang direferensikan aplikasi.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.