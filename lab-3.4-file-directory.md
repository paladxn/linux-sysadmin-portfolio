# Lab 3.4 — File & Directory Operations

**Course:** Linux System Administration (Adinusa)
**Topic:** Basic file and directory management
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Mengecek direktori kerja saat ini dengan `pwd`
- Membuat direktori dan sub-direktori dengan `mkdir`
- Berpindah antar direktori dengan `cd`
- Membuat file kosong dengan `touch`
- Membuat file berisi teks dengan redirection `>`
- Menghapus file dengan `rm`
- Menghapus direktori dengan `rmdir` dan `rm -r`

---

## 📘 Guided Example

### 1. Cek direktori kerja saat ini

```bash
$ pwd
```

**Penjelasan:** `pwd` (*print working directory*) menampilkan path absolut direktori tempat kita berada saat ini.

### 2. Membuat direktori `lab3`

```bash
$ mkdir lab3
```

### 3. Membuat sub-direktori

```bash
$ mkdir lab3/docs
$ mkdir lab3/images
```

### 4. Masuk ke dalam direktori

```bash
$ cd lab3
```

### 5. Membuat file kosong

```bash
$ touch file1.txt
$ touch docs/file2.txt
```

**Catatan:** `touch` digunakan untuk membuat file kosong. Jika file sudah ada, `touch` hanya memperbarui timestamp-nya.

### 6. Membuat file berisi teks

```bash
$ echo "Hello Linux" > hello.txt
```

**Penjelasan:** Simbol `>` melakukan **redirection**, yaitu menulis output perintah `echo` ke dalam file. Jika file belum ada, akan dibuat. Jika sudah ada, isinya akan **ditimpa**.

### 7. Menghapus file

```bash
$ rm file1.txt
```

### 8. Menghapus direktori

Hanya bisa jika direktori **kosong**:

```bash
$ rmdir docs
```

Jika direktori **berisi file**, gunakan:

```bash
$ rm -r docs
```

**Peringatan:** `rm -r` bersifat rekursif dan langsung menghapus isi direktori tanpa konfirmasi. Gunakan dengan hati-hati.

### 9. Kembali ke home directory

```bash
$ cd ~
```

---

## 🧪 Practice Task

### Soal

1. Buat direktori bernama `project`, di dalamnya buat dua sub-direktori: `src` dan `bin`.
2. Masuk ke dalam direktori `project`.
3. Di dalam `src`, buat dua file kosong bernama `main.c` dan `utils.c`.
4. Di dalam `bin`, buat file `readme.txt` berisi teks: `This is the bin folder.`
5. Hapus file `utils.c` dari folder `src`.

### ✅ Solusi

```bash
# 1. Membuat direktori project beserta sub-direktorinya
$ mkdir -p project/src project/bin

# 2. Masuk ke direktori project
$ cd project

# 3. Membuat dua file kosong di dalam src
$ touch src/main.c src/utils.c

# 4. Membuat file readme.txt di dalam bin dengan isi teks
$ echo "This is the bin folder." > bin/readme.txt

# 5. Menghapus file utils.c
$ rm src/utils.c
```

### 🔍 Verifikasi Hasil

```bash
# Cek struktur direktori
$ ls -R
.:
bin  src

./bin:
readme.txt

./src:
main.c

# Cek isi readme.txt
$ cat bin/readme.txt
This is the bin folder.
```

---

## 💡 Catatan & Insight Pribadi

- Opsi `-p` pada `mkdir` sangat berguna untuk membuat **parent directory sekaligus** dalam satu perintah, misalnya `mkdir -p project/src`.
- Perbedaan `rmdir` dan `rm -r`:
  - `rmdir` → hanya untuk direktori kosong, lebih aman.
  - `rm -r` → untuk direktori berisi file, **berisiko**, sebaiknya gunakan `rm -ri` agar ada konfirmasi interaktif.
- Redirection `>` menimpa isi file. Untuk **menambahkan** tanpa menimpa, gunakan `>>`.
- `touch` juga bisa membuat beberapa file sekaligus dengan satu perintah.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `pwd` | Menampilkan direktori kerja saat ini |
| `mkdir` | Membuat direktori |
| `mkdir -p` | Membuat direktori beserta parent-nya |
| `cd` | Berpindah direktori |
| `touch` | Membuat file kosong |
| `echo "..." > file` | Membuat file dengan isi teks |
| `rm` | Menghapus file |
| `rmdir` | Menghapus direktori kosong |
| `rm -r` | Menghapus direktori beserta isinya |
| `ls -R` | Menampilkan isi direktori secara rekursif |
| `cat` | Menampilkan isi file |

---

## 📌 Kesimpulan

Lab ini memberikan dasar yang kuat untuk navigasi dan manipulasi file/direktori di Linux. Perintah-perintah ini akan terus dipakai di lab-lab berikutnya, terutama saat mengelola konfigurasi sistem, log, dan skrip otomasi.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.