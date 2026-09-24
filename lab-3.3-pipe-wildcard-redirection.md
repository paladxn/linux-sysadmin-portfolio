# Lab 3.3 — Pipe, Wildcard & Redirection

**Course:** Linux System Administration (Adinusa)
**Topic:** Pipe operator, wildcard, dan redirection
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Menggunakan **pipe (`|`)** untuk menghubungkan output satu perintah ke perintah lain
- Menggunakan **wildcard (`*`)** untuk mencari file dengan pola tertentu
- Mengarahkan output perintah ke file dengan **`>`** (overwrite) dan **`>>`** (append)
- Melakukan operasi dasar: membuat file kosong, menampilkan isi file, mencari teks, dan menggabungkan isi beberapa file

---

## 📘 Guided Example

### 1. Pipe (`|`)

Pipe mengirim output satu perintah sebagai input ke perintah berikutnya.

```bash
$ ls -l | less
```

- `ls -l` menampilkan file dalam format panjang.
- `less` memungkinkan scroll output halaman per halaman.

Contoh lain:

```bash
$ cat /etc/passwd | grep root
```

Menampilkan hanya baris yang mengandung kata `root`.

### 2. Asterisk (`*`)

Asterisk adalah wildcard yang cocok dengan **nol atau lebih karakter**.

```bash
$ ls *.txt       # Menampilkan semua file berekstensi .txt
$ ls file*       # Menampilkan semua file yang diawali "file"
```

### 3. Redirect output ke file (`>`)

Operator `>` menulis output perintah ke file (**menimpa** isi lama).

```bash
$ echo "Hello Linux" > output.txt
```

- Membuat file `output.txt` berisi teks `Hello Linux`.
- Jika `output.txt` sudah ada, isinya akan **diganti**.

### 4. Append output ke file (`>>`)

Operator `>>` menambahkan output ke **akhir** file tanpa menghapus isi sebelumnya.

```bash
$ echo "New line added" >> output.txt
```

Menambahkan `New line added` di baris terakhir `output.txt`.

---

## 🧪 Practice Task

### Soal

Pastikan berada di direktori `lab33`:

```bash
$ cd ~/lab33
```

Kerjakan tugas berikut:

1. Buat satu file kosong di dalam direktori `lab33`: `file1.txt`.
2. Tampilkan semua file `.txt` di direktori saat ini menggunakan wildcard `*`.
3. Tambahkan teks `Adinusa First Line` ke `file1.txt` menggunakan `>`.
4. Tambahkan baris lain `Adinusa Second Line` ke `file1.txt` menggunakan `>>`.
5. Tampilkan isi `file1.txt` di layar.
6. Gunakan `cat` dan pipe (`|`) untuk menampilkan hanya baris yang mengandung kata `Second` dari `file1.txt`.
7. Gabungkan dua file (`notes.txt` dan `data.txt`) menjadi file baru bernama `combined.txt` menggunakan `>`, lalu verifikasi isinya dengan `cat`.

### ✅ Solusi

```bash
# 1. Buat file kosong
$ touch file1.txt

# 2. Tampilkan semua file .txt
$ ls *.txt

# 3. Tulis teks ke file1.txt (overwrite)
$ echo "Adinusa First Line" > file1.txt

# 4. Tambahkan baris kedua (append)
$ echo "Adinusa Second Line" >> file1.txt

# 5. Tampilkan isi file
$ cat file1.txt

# 6. Filter baris yang mengandung "Second"
$ cat file1.txt | grep Second

# 7. Gabungkan dua file menjadi satu
$ cat notes.txt data.txt > combined.txt
$ cat combined.txt
```

### 🔍 Hasil Akhir yang Diharapkan

Isi `file1.txt`:

```
Adinusa First Line
Adinusa Second Line
```

Output dari `cat file1.txt | grep Second`:

```
Adinusa Second Line
```

---

## 💡 Catatan & Insight Pribadi

- **Pipe (`|`)** adalah salah satu konsep paling powerful di Linux. Dengan pipe, kita bisa menggabungkan perintah kecil menjadi satu pipeline yang kompleks, sesuai filosofi Unix: *"do one thing and do it well"*.
- Perbedaan `>` dan `>>` sangat krusial:
  - `>` → **overwrite** (menimpa isi lama).
  - `>>` → **append** (menambahkan di akhir).
  - Salah pakai `>` di file penting bisa berakibat fatal — file lama langsung hilang tanpa konfirmasi.
- Wildcard `*` bisa dikombinasikan dengan pola lain, misal `ls *.log`, `ls data-*.csv`.
- `grep` sangat berguna untuk filter teks; bisa dipadukan dengan `-i` (case-insensitive) atau `-v` (invert/match selain pola).
- Untuk melihat isi file panjang, `less` lebih nyaman daripada `cat` karena bisa di-scroll.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `\|` (pipe) | Menghubungkan output ke perintah berikutnya |
| `*` | Wildcard: cocok dengan nol atau lebih karakter |
| `>` | Redirect output ke file (overwrite) |
| `>>` | Redirect output ke file (append) |
| `touch` | Membuat file kosong |
| `ls *.txt` | List file dengan pola tertentu |
| `cat file1 file2 > file3` | Gabungkan beberapa file |
| `grep` | Filter baris berdasarkan pola |
| `less` | Menampilkan output halaman per halaman |

---

## 📌 Kesimpulan

Lab ini memperkenalkan tiga konsep inti pengolahan data di command line Linux: **pipe**, **wildcard**, dan **redirection**. Kombinasi ketiganya adalah fondasi dari hampir semua tugas otomasi dan administrasi sistem — mulai dari parsing log, filter output, sampai membangun pipeline data. Menguasainya akan sangat membantu di lab-lab berikutnya, terutama saat menulis skrip Bash.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.