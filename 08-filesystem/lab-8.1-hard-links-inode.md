# Lab 8.1 — Hard Links & Inode

**Course:** Linux System Administration (Adinusa)
**Topic:** Inode, hard link, dan persistensi data
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membuat file dan memeriksa **inode number**-nya
- Membuat **hard link** dan memverifikasi bahwa beberapa nama file berbagi inode yang sama
- Menunjukkan **konsistensi data** antar hard link
- Memahami **persistensi data** ketika file asli dihapus tapi hard link masih ada

---

## 📘 Konsep Dasar

Sebelum praktik, ada beberapa konsep penting:

| Konsep | Penjelasan |
|---|---|
| **Inode** | Struktur data di filesystem yang menyimpan metadata file (permission, owner, size, timestamp, pointer ke data block). Setiap file punya **satu inode unik** per filesystem. |
| **Hard Link** | Nama file tambahan yang menunjuk ke **inode yang sama**. Tidak ada perbedaan "asli" vs "link" — semuanya setara. |
| **Link Count** | Jumlah nama file yang menunjuk ke inode tertentu. Bisa dilihat di kolom kedua `ls -l`. |
| **Data Block** | Tempat data file sebenarnya disimpan. Inode hanya menunjuk ke block ini. |

**Poin penting:**
> Hard link **bukan salinan** file. Kalau kamu edit salah satu nama, yang lain langsung ikut berubah karena keduanya menunjuk inode yang sama. Data baru benar-benar terhapus hanya ketika **semua** hard link-nya dihapus (link count = 0).

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-008-1
```

---

## 📘 Guided Example

### 1. Masuk ke direktori lab81

```bash
$ cd ~/lab81
$ pwd
```

Expected output:

```
/home/student/lab81
```

### 2. Membuat file `file1.txt`

```bash
$ echo "This is the original file" > file1.txt
```

### 3. Cek inode number

```bash
$ ls -li file1.txt
```

Contoh output:

```
264520 -rw-r--r-- 1 student student 27 Aug 25 13:00 file1.txt
```

**Catat inode number-nya** (contoh: `264520`).

**Penjelasan kolom:**
| Kolom | Isi |
|---|---|
| `264520` | **Inode number** |
| `-rw-r--r--` | Permission |
| `1` | **Link count** (jumlah nama yang menunjuk inode ini) |
| `student student` | Owner & group |
| `27` | Ukuran file (byte) |
| `Aug 25 13:00` | Timestamp |
| `file1.txt` | Nama file |

### 4. Membuat hard link

```bash
$ ln file1.txt file2.txt
$ ln file1.txt file3.txt
```

### 5. Verifikasi hard link

```bash
$ ls -li
```

Contoh output:

```
264520 -rw-r--r-- 3 student student 27 Aug 25 13:00 file1.txt
264520 -rw-r--r-- 3 student student 27 Aug 25 13:00 file2.txt
264520 -rw-r--r-- 3 student student 27 Aug 25 13:00 file3.txt
```

**Perhatikan:**
- Ketiga file punya **inode number yang sama** (`264520`).
- **Link count** naik dari `1` menjadi `3`.

### 6. Data consistency — edit satu, yang lain ikut berubah

```bash
$ echo "Additional data via file1" >> file1.txt
$ cat file2.txt
```

Expected output:

```
This is the original file
Additional data via file1
```

Perubahan di `file1.txt` **langsung muncul** di `file2.txt` karena keduanya menunjuk inode yang sama.

### 7. Persistensi data — hapus file asli

```bash
$ rm file1.txt
$ cat file2.txt
```

Output tetap:

```
This is the original file
Additional data via file1
```

**Kenapa?** Karena inode masih direferensikan oleh `file2.txt` (dan `file3.txt`). Yang dihapus hanya **nama** `file1.txt`, bukan inode atau datanya.

Cek link count setelah `file1.txt` dihapus:

```bash
$ ls -li
```

Link count turun dari `3` menjadi `2` (untuk `file2.txt` dan `file3.txt`).

---

## 🧪 Practice Task

### Soal

1. Buat file `original.txt` dengan isi `Linux is fun`.
2. Cek inode number-nya dengan `ls -li`.
3. Buat dua hard link: `link1.txt` dan `link2.txt`.
4. Verifikasi ketiganya punya inode number yang sama dan link count = 3.
5. Tambahkan baris `Hard links share data` ke `original.txt`.
6. Baca `link1.txt` — pastikan perubahan terlihat.
7. Hapus `original.txt` dan `link1.txt`.
8. Verifikasi bahwa `link2.txt` masih bisa dibaca dan isinya utuh.
9. Cek link count di `link2.txt` — seharusnya sekarang bernilai **1**.

### ✅ Solusi

```bash
# 1. Buat file asli
$ echo "Linux is fun" > ~/lab81/original.txt

# 2. Cek inode
$ ls -li ~/lab81/original.txt

# 3. Buat hard link
$ ln ~/lab81/original.txt ~/lab81/link1.txt
$ ln ~/lab81/original.txt ~/lab81/link2.txt

# 4. Verifikasi inode & link count
$ ls -li ~/lab81/
# Ketiganya harus punya inode number sama, link count = 3

# 5. Tambah data
$ echo "Hard links share data" >> ~/lab81/original.txt

# 6. Baca lewat link
$ cat ~/lab81/link1.txt
# Output:
# Linux is fun
# Hard links share data

# 7. Hapus dua nama
$ rm ~/lab81/original.txt ~/lab81/link1.txt

# 8. Baca lewat link yang tersisa
$ cat ~/lab81/link2.txt
# Output tetap utuh

# 9. Cek link count
$ ls -li ~/lab81/link2.txt
# Link count sekarang = 1
```

### 🔍 Hasil Akhir yang Diharapkan

Setelah semua langkah:

```bash
$ cat ~/lab81/link2.txt
Linux is fun
Hard links share data

$ ls -li ~/lab81/
# hanya link2.txt yang tersisa, dengan link count = 1
```

---

## 🐛 Troubleshooting & Kesalahan

- **Salah paham "file asli" vs "hard link":** Awalnya saya pikir `file1.txt` adalah "file utama" dan `file2.txt`/`file3.txt` hanya "salinan" atau "alias" yang bergantung padanya. Ternyata **salah** — ketiganya **setara**. Setelah `file1.txt` dihapus, `file2.txt` dan `file3.txt` tetap berfungsi penuh, seolah-olah tidak pernah ada `file1.txt`. Konsep kuncinya: **inode** yang menyimpan data, nama file hanya label yang menunjuk ke inode itu.

- **Hard link tidak bisa lintas filesystem:** Saya sempat coba `ln /home/student/lab81/file1.txt /tmp/file2.txt`, dan gagal dengan error *"Invalid cross-device link"*. Ternyata **hard link hanya bisa dibuat dalam filesystem yang sama**, karena inode hanya unik per filesystem. Untuk lintas filesystem, harus pakai **symbolic link** (`ln -s`).

- **Hard link ke direktori:** Mencoba `ln ~/lab81 /tmp/lab81link` gagal dengan *"hard link not allowed for directory"*. Ini memang **dilarang** di Linux untuk mencegah loop di struktur direktori.

- **Bingung membaca output `ls -li`:** Awalnya saya tidak sadar kolom pertama di `ls -li` adalah **inode number**, bukan permission. Setelah tahu, baru jelas bahwa yang menentukan "file itu sama atau bukan" adalah inode number-nya, bukan namanya.

- **Link count vs jumlah nama:** Link count di `ls -l` menunjukkan **jumlah nama file** yang menunjuk inode tersebut. Saat link count = 0, inode dan datanya baru benar-benar dihapus oleh filesystem.

- **`ls -li` tidak menampilkan hidden file:** Kalau file-nya diawali titik (misal `.hidden.txt`), perlu tambahkan `-a`: `ls -lai`.

---

## 💡 Catatan & Insight Pribadi

- **Hard link vs symbolic link:**

  | Aspek | Hard Link (`ln`) | Symbolic Link (`ln -s`) |
  |---|---|---|
  | Menunjuk ke | Inode yang sama | Path/nama file |
  | Inode number | **Sama** dengan aslinya | **Berbeda** |
  | Lintas filesystem | ❌ Tidak bisa | ✅ Bisa |
  | Link ke direktori | ❌ Tidak bisa | ✅ Bisa |
  | Kalau file asli dihapus | Data tetap ada (inode masih direferensikan) | Link jadi rusak (*dangling*) |
  | `ls -l` | Tampak seperti file biasa | Ditandai dengan `l` dan `->` |

- **Kenapa hard link tidak bisa lintas filesystem?** Karena inode **unik per filesystem**. Nomor inode `264520` di `/home` bisa saja dipakai oleh file yang sama sekali berbeda di `/tmp`. Kalau hard link boleh lintas filesystem, sistem tidak akan tahu inode mana yang dimaksud.

- **Kapan hard link berguna?**
  - **Backup incremental**: Beberapa nama file menunjuk data yang sama, tapi hanya menambah metadata — sangat efisien untuk snapshot.
  - **`rsync --link-dest`**: Membuat snapshot yang hanya menyalin file berubah, sisanya hard link ke snapshot sebelumnya.
  - **Menghindari duplikasi file besar**: Daripada copy 10 GB file, buat hard link saja.

- **Inode number tidak selalu sama antar filesystem:** Saat kamu `ls -li` di `/home` dan `/tmp`, inode number bisa saja sama tapi menunjuk file yang berbeda, karena inode hanya unik **per filesystem**.

- **Cek filesystem dari sebuah file:**
  ```bash
  $ df -h /home/student/lab81
  $ stat -c "%d %i" file1.txt   # device + inode
  ```

- **Statistik link count di `stat`:**
  ```bash
  $ stat file1.txt
  ```
  Output akan menampilkan `Links: 3` dan `Inode: 264520`.

- **Kasus nyata:** Di server backup, teknik hard link sering dipakai untuk membuat snapshot harian yang hemat ruang. Tools seperti `rsnapshot` dan `backintime` memanfaatkan hard link untuk menyimpan versi berbeda dari file yang sama tanpa menduplikasi data.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `echo "..." > file` | Buat file dengan isi tertentu |
| `ls -li file` | Lihat detail file + inode number |
| `ln source target` | Buat hard link |
| `ln -s source target` | Buat symbolic link |
| `cat file` | Tampilkan isi file |
| `stat file` | Info lengkap file (inode, link count, dll) |
| `rm file` | Hapus nama file (inode tetap ada selama link count > 0) |
| `df -h /path` | Lihat filesystem dari sebuah path |

---

## 📌 Kesimpulan

Lab ini membuka pemahaman baru tentang **bagaimana Linux menyimpan file** di balik layar. Selama ini saya pikir file itu "satu nama = satu data", ternyata yang sebenarnya terjadi adalah **satu inode bisa direferensikan oleh banyak nama**. Konsep ini menjadi fondasi untuk memahami fitur-fitur tingkat lanjut seperti snapshot backup, versioning filesystem (ZFS, Btrfs), dan optimasi penyimpanan. Memahami hard link juga membuat kita lebih berhati-hati saat menghapus file — data tidak benar-benar hilang sampai semua hard link-nya dihapus.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
