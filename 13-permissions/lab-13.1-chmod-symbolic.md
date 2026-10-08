# Lab 13.1 — Chmod Symbolic Mode

**Course:** Linux System Administration (Adinusa)
**Topic:** Mengubah permission file dengan `chmod` (symbolic mode)
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Melihat detail file dan permission-nya dengan `ls -l`
- Mengubah permission file menggunakan `chmod` dengan **symbolic mode**
- Memberi dan mencabut permission **read**, **write**, dan **execute** untuk **user**, **group**, dan **others**

---

## 📘 Konsep Dasar

### Tiga Kelompok Permission

Setiap file di Linux punya permission untuk tiga kelompok:

| Kelompok | Simbol | Arti |
|---|---|---|
| **User** | `u` | Pemilik file (owner) |
| **Group** | `g` | Grup pemilik file |
| **Others** | `o` | Semua user lain |
| **All** | `a` | Ketiganya (u+g+o) |

### Tiga Jenis Permission

| Permission | Simbol | Angka | Efek pada File | Efek pada Direktori |
|---|---|---|---|---|
| **Read** | `r` | 4 | Bisa baca isi file | Bisa list isi direktori |
| **Write** | `w` | 2 | Bisa ubah isi file | Bisa buat/hapus file di dalamnya |
| **Execute** | `x` | 1 | Bisa jalankan file | Bisa masuk (`cd`) ke direktori |

### Cara Membaca `ls -l`

Contoh output:

```
-rw-rw-r-- 1 student student 0 Aug 26 08:47 example_file
│└┬┘└┬┘└┬┘
│ │  │  └── others: r-- (read only)
│ │  └───── group:  rw- (read + write)
│ └──────── user:   rw- (read + write)
└────────── tipe file: - (file biasa)
```

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-013-1
```

---

## 📘 Guided Example

### 1. Buat file `example_file`

```bash
$ touch ~/lab131/example_file
```

### 2. Lihat permission awal

```bash
$ ls -l ~/lab131/example_file
-rw-rw-r-- 1 student student 0 Aug 26 08:47 /home/student/lab131/example_file
```

Permission awal: `rw-` untuk user, `rw-` untuk group, `r--` untuk others.

### 3. Ubah permission: user=read, group=write, others=execute

```bash
$ chmod u=r,g=w,o=x ~/lab131/example_file
$ ls -l ~/lab131/example_file
-r---w---x 1 student student 0 Aug 26 08:47 /home/student/lab131/example_file
```

**Penjelasan simbol:**
- `u=r` → user hanya **read** (write & execute dicabut)
- `g=w` → group hanya **write**
- `o=x` → others hanya **execute**

### 4. Modifikasi lagi: tambah write untuk user, hapus write group, tambah read+write untuk others

```bash
$ chmod u+w,g-w,o+rw ~/lab131/example_file
$ ls -l ~/lab131/example_file
-rw----rwx 1 student student 0 Aug 26 08:47 /home/student/lab131/example_file
```

**Penjelasan simbol:**
- `u+w` → **tambah** write untuk user (dari `r--` jadi `rw-`)
- `g-w` → **hapus** write dari group (dari `-w-` jadi `---`)
- `o+rw` → **tambah** read dan write untuk others (dari `--x` jadi `rwx`)

### 5. Ubah lagi: hapus semua permission user & group, sisakan execute untuk others

```bash
$ chmod ug-rwx,o-rw ~/lab131/example_file
$ ls -l ~/lab131/example_file
---------x 1 student student 0 Aug 26 08:47 /home/student/lab131/example_file
```

**Penjelasan simbol:**
- `ug-rwx` → hapus read, write, execute dari user dan group
- `o-rw` → hapus read dan write dari others (execute tetap)

Hasil akhir: hanya others yang punya **execute**.

---

## 🧪 Practice Task

### Soal

1. Buat file `latihan.txt` di `~/lab131/`.
2. Lihat permission awalnya.
3. Ubah permission menjadi:
   - User: read + write + execute
   - Group: read only
   - Others: no permission
4. Tambahkan write untuk group, hapus execute dari user.
5. Hapus semua permission dari others (kalau ada).
6. Tampilkan hasil akhir dengan `ls -l`.

### ✅ Solusi

```bash
# 1. Buat file
$ touch ~/lab131/latihan.txt

# 2. Lihat permission awal
$ ls -l ~/lab131/latihan.txt

# 3. Set permission: u=rwx, g=r, o=
$ chmod u=rwx,g=r,o= ~/lab131/latihan.txt
$ ls -l ~/lab131/latihan.txt
-rwxr----- 1 student student 0 ... latihan.txt

# 4. Tambah write untuk group, hapus execute dari user
$ chmod g+w,u-x ~/lab131/latihan.txt
$ ls -l ~/lab131/latihan.txt
-rw-rw---- 1 student student 0 ... latihan.txt

# 5. Others sudah tidak punya permission, tidak perlu diubah

# 6. Tampilkan hasil akhir
$ ls -l ~/lab131/latihan.txt
-rw-rw---- 1 student student 0 ... latihan.txt
```

### 🔍 Hasil yang Diharapkan

File `latihan.txt` dengan permission `-rw-rw----`:
- User: read + write
- Group: read + write
- Others: tidak ada permission

---

## 🐛 Troubleshooting & Kesalahan

- **Lupa tanda `=` vs `+` vs `-`:** 
  - `=` → **set** permission persis seperti yang ditulis (permission lain dicabut).
  - `+` → **tambah** permission (yang lain tetap).
  - `-` → **hapus** permission.
  Kesalahan umum: pakai `+` padahal maksudnya `=`, sehingga permission lama masih tertinggal.

- **Tidak bisa mengubah permission file milik orang lain:** Hanya **root** atau **pemilik file** yang bisa `chmod`. Kalau bukan pemilik, akan muncul *"Operation not permitted"*.

- **Permission pada direktori berbeda efeknya:** 
  - `r` pada direktori → bisa `ls`.
  - `w` pada direktori → bisa buat/hapus file di dalamnya (meskipun tidak bisa baca file itu).
  - `x` pada direktori → bisa `cd` masuk.
  Kombinasi `--x` pada direktori = bisa masuk tapi tidak bisa list isi.

- **Lupa bahwa `chmod` butuh path lengkap:** Kalau file ada di `~/lab131/`, pastikan tulis `~/lab131/example_file`, bukan hanya `example_file` (kecuali sedang berada di direktori itu).

- **Mengubah permission file sistem:** 🚨 Jangan pernah `chmod` file di `/etc`, `/bin`, `/usr` tanpa tahu akibatnya. Bisa membuat sistem tidak bisa boot.

- **Symbolic mode vs numeric mode:** Lab ini fokus pada symbolic (`u=r,g=w,o=x`). Numeric mode (`chmod 644`) akan dipelajari di lab terpisah. Keduanya setara, hanya beda cara penulisan.

- **`chmod` pada symlink:** `chmod` pada symlink akan mengubah permission **target**, bukan symlink-nya. Untuk mengubah symlink itu sendiri, gunakan `chmod -h` (di beberapa sistem).

- **Permission `x` pada file teks:** Memberi `x` pada file teks tidak membuatnya otomatis bisa dijalankan — file harus punya **shebang** (`#!/bin/bash`) dan interpreter yang sesuai.

---

## 💡 Catatan & Insight Pribadi

### Symbolic Mode vs Numeric Mode

| Aspek | Symbolic | Numeric |
|---|---|---|
| Contoh | `u=rwx,g=r,o=` | `chmod 740` |
| Kelebihan | Jelas, mudah dibaca | Singkat |
| Kekurangan | Lebih panjang | Harus hafal angka |
| Kapan pakai | Saat ingin ubah sebagian | Saat ingin set semua sekaligus |

**Konversi cepat:**

| Permission | Symbolic | Numeric |
|---|---|---|
| `rwx` | read+write+execute | 7 |
| `rw-` | read+write | 6 |
| `r-x` | read+execute | 5 |
| `r--` | read only | 4 |
| `-wx` | write+execute | 3 |
| `-w-` | write only | 2 |
| `--x` | execute only | 1 |
| `---` | no permission | 0 |

### Kapan Pakai `=` dan Kapan Pakai `+`/`-`

- **`=`** → saat ingin **reset** permission ke kondisi tertentu. Contoh: `chmod u=rwx` akan menghapus permission lain yang mungkin ada.
- **`+`/`-`** → saat ingin **menambah/mengurangi** dari kondisi saat ini. Contoh: `chmod u+x` hanya menambah execute, tidak mengubah yang lain.

### Permission pada Direktori

| Permission | Efek |
|---|---|
| `r` | Bisa list isi direktori (`ls`) |
| `w` | Bisa buat/hapus/rename file di dalamnya |
| `x` | Bisa masuk (`cd`) dan akses file di dalamnya |
| `rwx` | Akses penuh |
| `--x` | Bisa masuk tapi tidak bisa list isi |
| `r--` | Bisa list tapi tidak bisa masuk |

### Best Practice Permission

| File | Rekomendasi | Alasan |
|---|---|---|
| File biasa | `644` (`rw-r--r--`) | Owner bisa edit, orang lain baca |
| Script | `755` (`rwxr-xr-x`) | Owner bisa edit & jalankan, orang lain jalankan |
| File sensitif | `600` (`rw-------`) | Hanya owner yang bisa akses |
| Direktori | `755` (`rwxr-xr-x`) | Owner akses penuh, orang lain bisa masuk & list |
| Direktori private | `700` (`rwx------`) | Hanya owner |

> 🚨 **Jangan pernah** `chmod 777` pada file atau direktori di sistem produksi. Itu memberikan akses penuh ke **semua orang** — pintu terbuka untuk attacker.

### Kasus Nyata

- **Script deployment:** File script harus `755` agar bisa dijalankan.
- **Kunci SSH:** Private key `~/.ssh/id_rsa` **harus** `600` — kalau tidak, SSH akan menolak.
- **Config rahasia:** File seperti `.env` atau `credentials.json` sebaiknya `600`.
- **Shared folder:** Direktori untuk kolaborasi bisa `775` (group bisa write).
- **Web server:** File HTML biasanya `644`, direktori `755`.

### Pelajaran Kunci dari Lab Ini

1. **`chmod` symbolic** memberi kontrol granular — bisa ubah per kelompok.
2. **`=` vs `+` vs `-`** — bedanya krusial, sering bikin salah.
3. **Permission direktori** berbeda efeknya dari file.
4. **`ls -l`** adalah cara baca permission — kuasai membacanya.
5. **Jangan asal `chmod 777`** — permission longgar = risiko keamanan.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `ls -l file` | Lihat permission file |
| `chmod u=r,g=w,o=x file` | Set permission dengan `=` |
| `chmod u+w file` | Tambah write untuk user |
| `chmod g-w file` | Hapus write dari group |
| `chmod o+rw file` | Tambah read+write untuk others |
| `chmod ug-rwx file` | Hapus semua permission user & group |
| `chmod a+x file` | Tambah execute untuk semua (`u+g+o`) |
| `chmod o= file` | Hapus semua permission others |

**Simbol yang perlu dihafal:**

| Simbol | Arti |
|---|---|
| `u` | User (owner) |
| `g` | Group |
| `o` | Others |
| `a` | All (u+g+o) |
| `r` | Read |
| `w` | Write |
| `x` | Execute |
| `=` | Set (reset) |
| `+` | Tambah |
| `-` | Hapus |

---

## 📌 Kesimpulan

Lab ini adalah fondasi untuk memahami **keamanan file di Linux**. Permission bukan sekadar formalitas — ini adalah **lapisan pertahanan pertama** dalam sistem multi-user. Dengan `chmod` symbolic, kita bisa memberikan akses yang tepat: cukup untuk bekerja, tapi tidak berlebihan.

Kemampuan membaca `ls -l` dan mengubah permission adalah keterampilan yang **wajib dimiliki setiap sysadmin**. Kesalahan kecil seperti `chmod 777` bisa membuka celah keamanan besar. Sebaliknya, permission yang terlalu ketat bisa membuat user tidak bisa bekerja.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
