# Lab 4.1 — Resource Limits dengan `ulimit`

**Course:** Linux System Administration (Adinusa)
**Topic:** Melihat & mengubah resource limits
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Menggunakan perintah `ulimit` untuk melihat dan mengubah **resource limits** seperti jumlah open files dan proses
- Memahami bagaimana limits tersebut memengaruhi shell
- Menetapkan **temporary limits** untuk sesi shell saat ini
- Membuat limits menjadi **permanent** melalui `/etc/security/limits.conf`

---

## 📘 Guided Example

### 1. Mulai shell baru

```bash
$ bash
```

**Penjelasan:** Dengan membuka shell baru, perubahan limit yang kita lakukan **hanya berlaku di shell ini**, tidak memengaruhi shell utama. Ini praktik yang aman untuk eksperimen.

### 2. Cek semua limit saat ini

```bash
$ ulimit -a
```

Output:

```
real-time non-blocking time  (microseconds, -R) unlimited
core file size              (blocks, -c) 0
data seg size               (kbytes, -d) unlimited
scheduling priority                 (-e) 0
file size                   (blocks, -f) unlimited
pending signals                     (-i) 3700
max locked memory           (kbytes, -l) 123048
max memory size             (kbytes, -m) unlimited
open files                          (-n) 1024
pipe size                (512 bytes, -p) 8
POSIX message queues         (bytes, -q) 819200
real-time priority                  (-r) 0
stack size                  (kbytes, -s) 8192
cpu time                   (seconds, -t) unlimited
max user processes                  (-u) 3700
virtual memory              (kbytes, -v) unlimited
file locks                          (-x) unlimited
```

Menampilkan **semua limit** yang berlaku: ukuran file, open files, jumlah proses, memory, dan lain-lain.

### 3. Cek jumlah maksimum open files

```bash
$ ulimit -n
1024
```

Menampilkan jumlah maksimum **file descriptor** (open files) yang boleh dibuka oleh satu proses.

### 4. Set limit open files baru (temporary)

```bash
$ ulimit -n 1000
$ ulimit -n
1000
```

Menetapkan jumlah maksimum open files menjadi **1000** untuk sesi shell saat ini saja. Begitu shell ditutup, nilai kembali ke default.

### 5. Cek maksimum proses per user

```bash
$ ulimit -u
3700
```

Menampilkan jumlah maksimum proses yang bisa dijalankan oleh satu user.

### 6. Set limit proses baru (temporary)

```bash
$ ulimit -u 200
$ ulimit -u
200
```

Membatasi jumlah proses yang bisa dibuat user saat ini menjadi **200** (hanya berlaku di shell ini).

### 7. Membuat limits permanent

**Edit file konfigurasi (sebagai root):**

```bash
$ sudo nano /etc/security/limits.conf
```

**Tambahkan di akhir file:**

```
student   hard   nproc   200
student   soft   nproc   200
```

**Penjelasan:**

| Tipe Limit | Arti |
|---|---|
| `soft` | Nilai yang **diterapkan default** saat user login |
| `hard` | Nilai **maksimum** yang boleh dinaikkan user (batas atas dari soft) |

Simpan dengan `CTRL + O` → `Enter` → `CTRL + X`.

---

## 🧪 Practice Task

### Soal

Coba lakukan sendiri:

1. Buka shell baru dengan `bash`.
2. Tampilkan semua limit dengan `ulimit -a`.
3. Catat nilai default dari:
   - `open files` (`-n`)
   - `max user processes` (`-u`)
4. Ubah limit open files menjadi **500**.
5. Ubah limit proses menjadi **150**.
6. Verifikasi kedua perubahan tersebut.
7. Tutup shell dengan `exit`, lalu buka shell baru dan cek apakah limit kembali ke default.

### ✅ Solusi

```bash
# 1. Buka shell baru
$ bash

# 2. Tampilkan semua limit
$ ulimit -a

# 3. Catat nilai default
$ ulimit -n    # open files
$ ulimit -u    # max processes

# 4. Ubah limit open files menjadi 500
$ ulimit -n 500

# 5. Ubah limit proses menjadi 150
$ ulimit -u 150

# 6. Verifikasi perubahan
$ ulimit -n
500
$ ulimit -u
150

# 7. Keluar dari shell
$ exit
```

### 🔍 Hasil Akhir yang Diharapkan

Setelah `exit` dan membuka shell baru, nilai `ulimit -n` dan `ulimit -u` **kembali ke default** (1024 dan 3700 pada contoh di lab). Ini membuktikan bahwa perubahan `ulimit` bersifat **session-only**.

---

## 💡 Catatan & Insight Pribadi

- `ulimit` hanya mengatur limit untuk **shell saat ini** dan proses turunannya. Setelah shell ditutup, semua perubahan temporary akan hilang.
- Untuk melihat nilai `ulimit` tertentu secara singkat, cukup gunakan flag-nya:
  - `ulimit -n` → open files
  - `ulimit -u` → max user processes
  - `ulimit -c` → core file size
  - `ulimit -f` → file size
- **Tidak semua limit bisa dinaikkan** sebagai user biasa. Untuk menaikkan di atas nilai `hard limit`, kita butuh akses `root` atau `sudo`. Yang bisa dilakukan user biasa adalah **menurunkan** limit (di bawah hard limit).
- Perbedaan `soft` dan `hard`:
  - `soft` adalah nilai yang berlaku aktif.
  - `hard` adalah batas atasnya — user boleh menaikkan `soft` sampai setinggi `hard`.
- Untuk mengubah limit secara **permanen**, file yang diedit adalah `/etc/security/limits.conf`. Bisa juga pakai file di `/etc/security/limits.d/` untuk konfigurasi yang lebih modular.
- Di sistem modern (systemd), limit juga bisa diatur via `systemd` unit (`LimitNOFILE=`, dll), tapi `limits.conf` tetap relevan untuk sesi login biasa.
- **Kasus nyata:** Aplikasi web sering butuh limit open files yang tinggi (misal 65535) karena harus membuka banyak koneksi simultan. Kalau default `1024` terlalu kecil, aplikasi bisa error *"too many open files"*.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `bash` | Membuka shell baru |
| `ulimit -a` | Tampilkan semua limit |
| `ulimit -n` | Lihat/set limit open files |
| `ulimit -u` | Lihat/set limit proses per user |
| `ulimit -c` | Lihat/set limit core file size |
| `ulimit -f` | Lihat/set limit ukuran file |
| `exit` | Menutup shell |
| `sudo nano /etc/security/limits.conf` | Edit file limit permanen |

---

## 📌 Kesimpulan

Lab ini memperkenalkan konsep **resource limits** di Linux — hal yang sering diabaikan pemula tapi sangat penting di dunia sysadmin. `ulimit` memungkinkan kita membatasi penggunaan resource (file, proses, memory) baik untuk alasan keamanan maupun stabilitas sistem. Memahami perbedaan **temporary vs permanent** dan **soft vs hard limit** adalah bekal penting saat mengelola server produksi, terutama untuk tuning aplikasi seperti web server atau database yang membutuhkan file descriptor dalam jumlah besar.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.