# Lab 3.4 — Perintah `ls` & Kombinasi dengan Pipe

**Course:** Linux System Administration (Adinusa)
**Topic:** Opsi `ls`, hidden files, human-readable size, dan pipe ke `wc`
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Menggunakan `ls` untuk menampilkan isi direktori dengan berbagai opsi
- Menampilkan **hidden files** dengan opsi `-a`
- Menampilkan ukuran file dalam format **human-readable** dengan opsi `-h`
- Menggabungkan perintah dengan **pipe (`|`)** untuk menghitung jumlah file
- Meningkatkan kemampuan navigasi dan manajemen file di Linux

---

## 📘 Guided Example

### 1. Bantuan `ls`

```bash
$ ls --help
```

Menampilkan seluruh opsi yang tersedia untuk perintah `ls`.

### 2. Menampilkan semua file termasuk hidden (`-a`)

```bash
$ sudo ls -a /
```

Output:

```
.    boot  home  lib64       media  proc  sbin  sys  var
..   dev   lib   libx32      mnt    root  snap  tmp  usr
bin  etc   lib32 lost+found  opt    run   srv
```

**Penjelasan:** Opsi `-a` (*all*) menampilkan seluruh file, termasuk file **hidden** (yang diawali titik `.`). Simbol `.` adalah direktori saat ini, `..` adalah direktori parent.

### 3. Menampilkan detail dengan ukuran human-readable (`-lah`)

```bash
$ sudo ls -lah /
```

Output (sebagian):

```
total 72K
drwxr-xr-x  19 root root  4.0K Aug 19 08:47 .
drwxr-xr-x  19 root root  4.0K Aug 19 08:47 ..
lrwxrwxrwx   1 root root     7 Jan  8  2021 bin -> usr/bin
drwxr-xr-x   4 root root  4.0K Aug 25 07:53 boot
drwxr-xr-x  18 root root  4.0K Aug 19 08:47 dev
drwxr-xr-x 100 root root  4.0K Aug 25 07:58 etc
drwxr-xr-x   4 root root  4.0K Aug 19 08:47 home
lrwxrwxrwx   1 root root     7 Jan  8  2021 lib -> usr/lib
...
dr-xr-xr-x  13 root root     0 Aug 19 08:46 sys
drwxrwxrwt  13 root root  4.0K Aug 26 06:47 tmp
drwxr-xr-x  14 root root  4.0K Jan  8  2021 usr
drwxr-xr-x  13 root root  4.0K Jan  8  2021 var
```

**Penjelasan kombinasi opsi:**

| Opsi | Arti |
|---|---|
| `-l` | Long format (detail: permission, owner, size, tanggal) |
| `-a` | Tampilkan semua file, termasuk hidden |
| `-h` | Human-readable (4.0K, 1.5M, dll — bukan byte mentah) |

### 4. Kombinasi perintah dengan pipe (`|`)

```bash
$ sudo ls / | wc -l
23
```

**Penjelasan:**
- `ls /` menampilkan daftar isi direktori root.
- `| wc -l` menghitung jumlah baris dari output tersebut.
- Hasilnya: jumlah item di dalam `/`.

---

## 🧪 Practice Task

### Soal

1. Masuk ke direktori `lab34`.
2. Tampilkan daftar isi dari direktori `lab34`.
3. Hitung jumlah file berdasarkan nama yang ada. Misalnya:
   - Berapa file yang bernama `client`?
   - Berapa yang bernama `ops`?
   - Berapa yang bernama `manager`?
4. Tulis hasil perhitungan ke file `lab34-answer.txt` di `/home/student/lab34/` dengan format:

```
client = 20
ops = 10
manager = 5
```

(Nilai di atas hanya contoh — sesuaikan dengan hasil hitung yang sebenarnya.)

### ✅ Solusi

```bash
# 1. Masuk ke direktori lab34
$ cd ~/lab34

# 2. Tampilkan isi direktori
$ ls
# atau lihat semua termasuk hidden:
$ ls -la

# 3. Hitung jumlah file per pola nama
$ ls | grep -c client
$ ls | grep -c ops
$ ls | grep -c manager

# 4. Tulis hasil ke file answer
$ echo "client = $(ls | grep -c client)" > lab34-answer.txt
$ echo "ops = $(ls | grep -c ops)" >> lab34-answer.txt
$ echo "manager = $(ls | grep -c manager)" >> lab34-answer.txt

# 5. Verifikasi hasil
$ cat lab34-answer.txt
```

### 🔍 Hasil Akhir yang Diharapkan

Isi `lab34-answer.txt`:

```
client = 20
ops = 10
manager = 5
```

### 🔄 Alternatif Solusi

Kalau nama file memiliki pola yang lebih terstruktur (misal `client-1`, `client-2`, ...), bisa juga pakai wildcard:

```bash
$ ls client* | wc -l
$ ls ops* | wc -l
$ ls manager* | wc -l
```

Atau langsung menuliskan semuanya dalam satu perintah:

```bash
{
  echo "client = $(ls client* 2>/dev/null | wc -l)"
  echo "ops = $(ls ops* 2>/dev/null | wc -l)"
  echo "manager = $(ls manager* 2>/dev/null | wc -l)"
} > lab34-answer.txt
```

**Catatan:** `2>/dev/null` mencegah pesan error muncul jika tidak ada file yang cocok dengan pola tertentu.

---

## 💡 Catatan & Insight Pribadi

- Opsi `-h` pada `ls` **wajib** dikombinasikan dengan `-l`, karena hanya berlaku di long format.
- `ls -la` dan `ls -al` sama saja — urutan opsi bebas.
- Perintah `wc` (*word count*) sebenarnya punya tiga mode:
  - `wc -l` → hitung baris
  - `wc -w` → hitung kata
  - `wc -c` → hitung byte
  - Tanpa opsi → tampilkan ketiganya
- Kombinasi `ls | grep -c pola` jauh lebih fleksibel untuk menghitung file berdasarkan pola nama dibanding wildcard, karena bisa memakai regex.
- `sudo` diperlukan saat mengakses direktori root (`/`) karena beberapa file di dalamnya hanya bisa dibaca oleh root.
- Untuk direktori dengan banyak file, `ls | grep -c` bisa dijadikan dasar skrip monitoring sederhana (misalnya: cek apakah file log sudah melebihi batas tertentu).

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `ls --help` | Menampilkan bantuan perintah `ls` |
| `ls -a` | Tampilkan semua file termasuk hidden |
| `ls -l` | Format panjang (detail) |
| `ls -lah` | Detail + hidden + human-readable |
| `\|` (pipe) | Menghubungkan output ke perintah lain |
| `wc -l` | Hitung jumlah baris |
| `grep -c` | Hitung jumlah baris yang cocok dengan pola |
| `echo "..." > file` | Tulis output ke file (overwrite) |
| `echo "..." >> file` | Tulis output ke file (append) |
| `2>/dev/null` | Buang pesan error |

---

## 📌 Kesimpulan

Lab ini memperdalam pemahaman tentang `ls` sebagai alat utama navigasi file di Linux. Kombinasi opsi `-lah` menjadi kebiasaan baru yang sangat berguna untuk inspeksi direktori secara cepat. Yang lebih penting lagi, lab ini memperkenalkan pola **"perintah kecil → pipa → perintah kecil lain"** yang merupakan inti dari filosofi command line Unix. Pola ini akan terus dipakai di lab-lab berikutnya, terutama saat parsing log dan otomatisasi tugas administrasi.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.