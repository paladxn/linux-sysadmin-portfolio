# Lab Challenge 4 — Comparing Files

**Course:** Linux System Administration (Adinusa)
**Topic:** `diff` — membandingkan file & direktori, deteksi perbedaan
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membandingkan **isi dua file** dengan `diff`
- Membandingkan **dua directory tree** secara rekursif
- Mengidentifikasi file yang **hanya ada di satu direktori**
- Menyimpan hasil perbandingan ke **file output**

---

## 📘 Konsep Dasar

**`diff`** = perintah untuk membandingkan dua file atau direktori baris per baris dan menampilkan perbedaannya.

| Perintah | Fungsi |
|---|---|
| `diff file1 file2` | Bandingkan 2 file |
| `diff -i file1 file2` | **Ignore case** |
| `diff -r dir1 dir2` | **Recursive** (bandingkan isi direktori) |
| `diff -q dir1 dir2` | **Quiet/brief** — hanya tampilkan file yang berbeda |
| `diff -rq dir1 dir2` | Recursive + brief |
| `diff -u file1 file2` | Unified format (seperti patch) |
| `diff -y file1 file2` | Side-by-side |

**Format output `diff`:**

| Simbol | Arti |
|---|---|
| `a` | Add — baris ditambahkan |
| `d` | Delete — baris dihapus |
| `c` | Change — baris diubah |

Contoh:
```
5c5
< baris di file1
---
> baris di file2
```
Artinya: baris 5 di file1 **diubah** menjadi baris 5 di file2.

**Untuk direktori, `diff -r` akan:**

| Output | Arti |
|---|---|
| `Files X and Y differ` | File X dan Y isinya berbeda |
| `Only in dir1: file.txt` | File hanya ada di dir1 |
| `Only in dir2: file.txt` | File hanya ada di dir2 |

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-challenge-4
```

---

## 🧪 Challenge Tasks

### Soal

1. **Bandingkan** `/home/student/challenge04/fileA.txt` dan `fileB.txt`, **ignore case sensitivity**, simpan hasil ke `/home/student/challenge04/results01.txt`.

2. **Bandingkan semua file** di direktori `/home/student/challenge04/compare01` dan `compare02` (termasuk subdirektori). Tepat **satu file berbeda**. Tulis **absolute path** file yang berbeda itu ke `/home/student/challenge04/results02.txt`.

3. Ada dua direktori: `/home/student/challenge04/comdir01` dan `comdir02`. Cari file yang **hanya ada di `comdir02`**. Tulis **hanya nama file** (bukan path lengkap) ke `/home/student/challenge04/results03.txt`.

### ✅ Solusi

```bash
# 1. Bandingkan 2 file, ignore case
$ diff -i /home/student/challenge04/fileA.txt /home/student/challenge04/fileB.txt > /home/student/challenge04/results01.txt

# 2. Bandingkan direktori rekursif, cari 1 file yang berbeda
$ diff -rq /home/student/challenge04/compare01 /home/student/challenge04/compare02 | grep "^Files" | awk '{print $2}' > /home/student/challenge04/results02.txt

# 3. Cari file yang HANYA ada di comdir02
$ diff -rq /home/student/challenge04/comdir01 /home/student/challenge04/comdir02 | grep "Only in /home/student/challenge04/comdir02" | awk '{print $NF}' > /home/student/challenge04/results03.txt
```

### 🔍 Penjelasan Opsi Penting

**Task 1 — `diff -i`**
- `-i` = ignore case (case-insensitive).
- Tanpa `-i`, `Hello` ≠ `hello`. Dengan `-i`, keduanya dianggap sama.

**Task 2 — `diff -rq` + grep + awk**
- `-r` = recursive — turun ke subdirektori.
- `-q` = quiet — hanya tampilkan file yang **berbeda**, bukan isi perbedaannya.
- Output: `Files /path/to/file1 and /path/to/file2 differ`
- `grep "^Files"` = filter hanya baris "Files ... differ".
- `awk '{print $2}'` = ambil kolom ke-2 = path file pertama.

**Task 3 — `diff -rq` + filter "Only in"**
- Output `diff -rq` untuk file yang hanya ada di satu sisi:
  ```
  Only in /home/student/challenge04/comdir02: file_xyz.txt
  ```
- `grep "Only in /home/student/challenge04/comdir02"` = filter hanya baris yang menunjuk ke comdir02.
- `awk '{print $NF}'` = ambil kolom terakhir = nama file **saja** (tanpa path).

### 🧩 Alternatif Solusi Task 3 — Pakai `comm`

Kalau file di kedua direktori **flat** (tidak ada subdirektori):

```bash
# 1. List file, sort
$ ls /home/student/challenge04/comdir01 > /tmp/list01.txt
$ ls /home/student/challenge04/comdir02 > /tmp/list02.txt

# 2. comm — tampilkan file yang hanya ada di list02
$ comm -13 <(sort /tmp/list01.txt) <(sort /tmp/list02.txt) > /home/student/challenge04/results03.txt
```

**Penjelasan `comm`:**
- `comm -13` = tampilkan **kolom ke-3** saja (file yang hanya ada di file kedua).
- `-12` = hanya kolom 3, `-23` = hanya kolom 1, dst.
- Wajib pakai file yang sudah **sorted**.

---

## 🔍 Verifikasi

### 1. Cek file output dibuat

```bash
$ ls -la /home/student/challenge04/
```

Harus ada 3 file: `results01.txt`, `results02.txt`, `results03.txt`.

### 2. Cek hasil Task 1

```bash
$ cat /home/student/challenge04/results01.txt
```

Kalau case-insensitive, perbedaan **hanya di case** tidak akan muncul. Kalau ada perbedaan selain case, akan muncul format `XcY`.

### 3. Cek hasil Task 2

```bash
$ cat /home/student/challenge04/results02.txt
```

Harus berisi **satu absolute path** file yang berbeda (misal `/home/student/challenge04/compare01/subdir/file.txt`).

### 4. Cek hasil Task 3

```bash
$ cat /home/student/challenge04/results03.txt
```

Harus berisi **nama file saja** (bukan path).

### 5. Bandingkan dengan source (opsional)

```bash
# Lihat semua file di comdir02
$ ls -la /home/student/challenge04/comdir02/

# Lihat semua file di comdir01
$ ls -la /home/student/challenge04/comdir01/
```

File yang ada di `comdir02` tapi **tidak** di `comdir01` = isi `results03.txt`.

### 6. Grading

```bash
$ nusactl grade linlab-challenge-4
```

---

## 🐛 Troubleshooting & Kesalahan

- **`diff: No such file or directory`:** Cek path file/direktori:
  ```bash
  ls -la /home/student/challenge04/
  ```

- **Output `diff` kosong:** Berarti file **identik**. Untuk Task 1, kalau `-i` di-set tapi output kosong, berarti file sama persis (case-insensitive). Ini bukan error — tapi kalau lab minta ada perbedaan, cek apakah perlu **tanpa** `-i`.

- **Task 2 output > 1 baris:** Berarti ada lebih dari satu file berbeda. Cek manual:
  ```bash
  diff -rq compare01 compare02
  ```
  Kalau ada lebih dari satu, cek lagi — lab biasanya sengaja set tepat satu file berbeda.

- **Task 3 output ada path, bukan nama file:** Berarti `awk` ambil kolom salah. Pastikan pakai `$NF` (kolom terakhir):
  ```bash
  diff -rq comdir01 comdir02 | grep "Only in.*comdir02" | awk '{print $NF}'
  ```
  Output `diff` untuk "Only in" adalah `Only in <path>: <filename>` — `$NF` = `<filename>`.

- **`grep "Only in"` ambil dari comdir01 juga:** Filter lebih spesifik:
  ```bash
  grep "Only in /home/student/challenge04/comdir02"
  ```
  bukan hanya `grep "Only in"`.

- **`comm` error "file not sorted":** `comm` butuh input **sorted**. Pakai `sort` dulu:
  ```bash
  comm -13 <(sort list1) <(sort list2)
  ```

- **Task 1 output tidak seperti yang diharapkan:** Ingat, `diff -i` akan **abaikan perbedaan case**. Kalau file A = `Hello` dan file B = `HELLO`, `diff -i` **tidak** akan menampilkan perbedaan. Kalau lab ingin **menampilkan** perbedaan case, jangan pakai `-i`.

- **`awk '{print $2}'` untuk Task 2 salah:** Format output `diff -rq` adalah:
  ```
  Files /path/file1 and /path/file2 differ
  ```
  Kolom ke-2 = `/path/file1`. Tapi kalau path mengandung **spasi**, awk bisa salah. Untuk kasus ini, biasanya path tidak ada spasi.

- **Task 3 file punya subdirektori:** `diff -rq` akan menampilkan `Only in /path/subdir: file.txt`. Kalau kamu hanya ingin **nama file** (bukan path subdir), `$NF` sudah benar (kolom terakhir = nama file).

- **Redirection ke file gagal permission:** Cek kepemilikan direktori `challenge04/`:
  ```bash
  ls -ld /home/student/challenge04/
  ```
  Kalau bukan milik `student`, pakai `sudo` atau ubah owner.

- **File output sudah ada:** `>` akan **overwrite**. Kalau mau append, pakai `>>`.

- **`diff` menampilkan `<` dan `>`:** Itu format normal. `<` = baris dari file pertama, `>` = baris dari file kedua. Bukan error.

---

## 💡 Catatan & Insight Pribadi

### Format Output `diff` — Cara Baca

**Contoh 1 — Baris berubah:**
```
5c5
< ini dari file1
---
> ini dari file2
```
Artinya: baris **ke-5** file1 **diubah** menjadi baris **ke-5** file2.

**Contoh 2 — Baris ditambah:**
```
3a4
> baris tambahan
```
Artinya: setelah baris **ke-3** file1, **ditambahkan** baris baru di file2.

**Contoh 3 — Baris dihapus:**
```
5d4
< baris yang dihapus
```
Artinya: baris **ke-5** file1 **dihapus** di file2.

### `diff` vs `comm` vs `cmp`

| Tool | Fungsi | Untuk |
|---|---|---|
| `diff` | Bandingkan baris per baris | File teks |
| `comm` | Bandingkan **sorted** line | File sorted |
| `cmp` | Bandingkan byte per byte | File binary |

**Rekomendasi:**
- File teks biasa → `diff`
- File sorted (list) → `comm`
- File binary → `cmp`

### `diff -r` untuk Direktori

**Output `diff -rq` (brief):**
```
Files dir1/file1 and dir2/file1 differ
Only in dir1: extra.txt
Only in dir2: new.txt
```

**Output `diff -r` (full):**
Menampilkan **isi** perbedaan juga, bukan hanya nama file.

**Untuk lab ini, `-q` lebih efisien** — kita cuma butuh **nama file**, bukan isinya.

### `grep` + `awk` = Kombinasi Powerful

**Filter baris tertentu:**
```bash
$ diff -rq dir1 dir2 | grep "Only in"
```

**Ambil kolom tertentu:**
```bash
$ diff -rq dir1 dir2 | awk '{print $2}'
```

**Kombinasi:**
```bash
$ diff -rq dir1 dir2 | grep "pattern" | awk '{print $NF}'
```

Ini **pola umum** parsing output command di Unix — pipe, filter, extract.

### Simbol di `diff` Unified (`-u`)

Format unified (dipakai di `git diff`, `patch`):
```
--- file1
+++ file2
@@ -1,5 +1,6 @@
 baris sama
-baris dihapus
+baris ditambah
 baris sama
```

| Simbol | Arti |
|---|---|
| `-` | Baris hanya ada di file1 |
| `+` | Baris hanya ada di file2 |
| ` ` (spasi) | Baris sama |

Berguna untuk **patch** — bisa diterapkan dengan `patch -p1 < file.patch`.

### `diff` dengan "Ignore" Options

| Opsi | Ignore |
|---|---|
| `-i` | Case (huruf besar/kecil) |
| `-w` | Whitespace |
| `-b` | Perubahan jumlah whitespace |
| `-B` | Baris kosong |
| `-Z` | Whitespace di akhir baris |

**Contoh:**
```bash
$ diff -iw file1 file2   # ignore case & whitespace
```

### `comm` — Cara Baca Output

`comm` butuh **2 file sorted**, output **3 kolom**:

```
file1_only    file2_only    common
```

| Opsi | Suppress |
|---|---|
| `-1` | Kolom 1 (file1 only) |
| `-2` | Kolom 2 (file2 only) |
| `-3` | Kolom 3 (common) |

**`comm -13`** = suppress kolom 1 & 3 → tampilkan **file2 only**.
**`comm -12`** = suppress kolom 1 & 2 → tampilkan **common**.
**`comm -23`** = suppress kolom 2 & 3 → tampilkan **file1 only**.

### Real-World Use Cases

- **Config diff:** `diff /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak` — cek perubahan config.
- **Backup verification:** `diff -r /data /backup/data` — pastikan backup identik.
- **Deploy check:** `diff -r app-v1 app-v2` — lihat file mana yang berubah.
- **Code review:** `git diff` (pakai engine `diff` juga).
- **Log comparison:** `diff log-hari-ini.log log-kemarin.log` — cari anomali.

### Kasus Nyata

- **DevOps:** Bandingkan config server A vs server B → cari drift.
- **Sysadmin:** Bandingkan `/etc` sebelum & sesudah update → lihat yang berubah.
- **Security:** Bandingkan file binary original vs current → deteksi modifikasi.
- **Migration:** Bandingkan direktori lama vs baru → pastikan semua file tercopy.

### Pelajaran Kunci dari Challenge Ini

1. **`diff`** = alat utama membandingkan file.
2. **`-i`** = ignore case.
3. **`-r`** = recursive untuk direktori.
4. **`-q`** = brief — hanya tampilkan file yang berbeda.
5. **`grep` + `awk`** = parsing output untuk extract info spesifik.
6. **`comm`** = alternatif untuk file sorted.
7. **Redirection `>`** = simpan hasil.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `diff file1 file2` | Bandingkan 2 file |
| `diff -i file1 file2` | Ignore case |
| `diff -u file1 file2` | Unified format |
| `diff -y file1 file2` | Side-by-side |
| `diff -r dir1 dir2` | Recursive |
| `diff -rq dir1 dir2` | Recursive + brief |
| `diff -w file1 file2` | Ignore whitespace |
| `cmp file1 file2` | Bandingkan byte-by-byte |
| `comm file1 file2` | Bandingkan sorted |
| `comm -13 file1 file2` | Hanya file2 only |
| `comm -23 file1 file2` | Hanya file1 only |
| `grep "pattern"` | Filter baris |
| `awk '{print $N}'` | Extract kolom |
| `cmd > file` | Redirect overwrite |

**Contoh task lengkap:**

```bash
# Task 1 — Ignore case
diff -i fileA.txt fileB.txt > results01.txt

# Task 2 — Recursive, ambil file yang beda (path)
diff -rq compare01 compare02 | grep "^Files" | awk '{print $2}' > results02.txt

# Task 3 — Hanya ada di comdir02 (nama file saja)
diff -rq comdir01 comdir02 | grep "Only in.*comdir02" | awk '{print $NF}' > results03.txt
```

**Alternatif Task 3 pakai `comm`:**

```bash
comm -13 <(ls comdir01 | sort) <(ls comdir02 | sort) > results03.txt
```

---

## 📌 Kesimpulan

Lab Challenge 4 ini menguji kemampuan **membandingkan file & direktori** — skill penting untuk sysadmin, DevOps, dan developer. Yang diuji:

- **`diff`** dengan opsi `-i` (case), `-r` (recursive), `-q` (brief).
- **Parsing output** dengan `grep` + `awk` untuk extract info spesifik.
- **Beda `diff` vs `comm`** — untuk file biasa vs file sorted.
- **Redirection** untuk menyimpan hasil.

Kemampuan ini adalah **fondasi** untuk:
- **Verifikasi backup** — pastikan backup identik dengan sumber.
- **Config drift detection** — cari perbedaan config antar server.
- **Code review** — `git diff` pakai engine yang sama.
- **Security audit** — deteksi modifikasi file penting.

Yang paling berharga dari challenge ini: **belajar membaca output `diff`**. Format `5c5`, `3a4`, `5d4` awalnya membingungkan, tapi setelah paham — sangat informatif.

`diff` adalah **salah satu tool paling sering dipakai sysadmin** — hampir setiap kali ada perubahan config, backup, atau migrasi, `diff` akan dipanggil.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
