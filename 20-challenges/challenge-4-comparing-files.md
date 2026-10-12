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
- Menyimpan hasil perbandingan ke **file output** dengan format yang diminta

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

**Untuk direktori, `diff -rq` punya 2 jenis output:**

| Output | Arti |
|---|---|
| `Files X and Y differ` | File X dan Y **isinya berbeda** |
| `Only in <dir>: <file>` | File **hanya ada di satu direktori** |

> **Pelajaran penting:** "Different file" bisa berarti 2 hal — isinya beda, **atau** keberadaannya cuma di satu sisi. Baca output mentah dulu untuk tahu mana yang dimaksud.

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

# 2. Bandingkan direktori rekursif → extract absolute path file yang berbeda
$ diff -rq /home/student/challenge04/compare01 /home/student/challenge04/compare02 | grep "Only in" | sed 's/Only in \(.*\): \(.*\)/\1\/\2/' > /home/student/challenge04/results02.txt

# 3. Cari file yang HANYA ada di comdir02 → nama file saja
$ diff -rq /home/student/challenge04/comdir01 /home/student/challenge04/comdir02 | grep "Only in /home/student/challenge04/comdir02" | awk '{print $NF}' > /home/student/challenge04/results03.txt
```

### 🔍 Penjelasan Opsi Penting

**Task 1 — `diff -i`**
- `-i` = ignore case (case-insensitive).
- Tanpa `-i`, `Hello` ≠ `hello`. Dengan `-i`, keduanya dianggap sama.

**Task 2 — `diff -rq` + `sed` untuk extract absolute path**

Output `diff -rq` untuk file yang hanya ada di satu sisi:
```
Only in /home/student/challenge04/compare01/ganjil: sample98
```

Kita perlu:
1. Filter baris `Only in`
2. Extract path direktori + nama file
3. Gabungkan jadi absolute path lengkap

**Perintah `sed 's/Only in \(.*\): \(.*\)/\1\/\2/'`:**

| Bagian | Arti |
|---|---|
| `s/.../.../` | Substitusi |
| `Only in \(.*\):` | Capture grup 1 = path direktori |
| ` \(.*\)` | Capture grup 2 = nama file |
| `\1\/\2` | Hasil: `<path_dir>/<nama_file>` |

**Hasil:**
```
/home/student/challenge04/compare01/ganjil/sample98
```

**Task 3 — `grep "Only in...comdir02"` + `awk '{print $NF}'`**

- `$NF` = kolom terakhir = nama file saja.
- Output `Only in /path/comdir02: file_xyz.txt` → `$NF` = `file_xyz.txt`.

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

Isi: format `XcY` (baris yang berubah) atau kosong kalau hanya case yang beda.

### 3. Cek hasil Task 2

```bash
$ cat /home/student/challenge04/results02.txt
```

Harus berisi **satu absolute path**:
```
/home/student/challenge04/compare01/ganjil/sample98
```

### 4. Cek hasil Task 3

```bash
$ cat /home/student/challenge04/results03.txt
```

Harus berisi **nama file saja** (bukan path).

### 5. Grading

```bash
$ nusactl grade linlab-challenge-4
```

---

## 🐛 Troubleshooting & Kesalahan

### Masalah Nyata yang Dialami

- **`results02.txt` kosong (0 bytes) padahal sudah pakai `grep "^Files"`:**

  **Penyebab:** Ternyata perbedaan yang dimaksud bukan `Files ... differ`, melainkan `Only in ...`.

  Output `diff -rq` yang muncul:
  ```
  Only in /home/student/challenge04/compare01/ganjil: sample98
  ```

  `grep "^Files"` tidak menangkap baris `Only in` → hasil kosong.

  **Solusi:** Ganti grep ke `"Only in"` + pakai `sed` untuk extract absolute path:
  ```bash
  diff -rq dir1 dir2 | grep "Only in" | sed 's/Only in \(.*\): \(.*\)/\1\/\2/' > results02.txt
  ```

- **Log grading: "the input file is not appropriate or the path is not absolute":**

  **Penyebab:** Grader ingin **absolute path** (mulai dari `/`), bukan nama file saja atau path relatif.

  **Solusi:** Extract path lengkap dengan `sed`. Contoh hasil yang benar:
  ```
  /home/student/challenge04/compare01/ganjil/sample98
  ```

- **Bingung mana yang dimaksud "different file":**

  Ada 2 jenis perbedaan di `diff -rq`:
  - **`Files X and Y differ`** → file ada di kedua sisi, isinya beda
  - **`Only in <dir>: <file>`** → file hanya ada di satu sisi

  Selalu jalankan `diff -rq` **mentah** dulu sebelum parsing, biar tahu mana yang dimaksud.

### Kesalahan Umum Lainnya

- **`diff: No such file or directory`:** Cek path:
  ```bash
  ls -la /home/student/challenge04/
  ```

- **Output `diff -i` kosong:** Berarti file **identik** (case-insensitive). Bukan error, tapi cek apakah perlu **tanpa** `-i`.

- **Task 3 output ada path:** Berarti `awk` ambil kolom salah. Pastikan `$NF` (kolom terakhir).

- **`grep "Only in"` ambil dari comdir01 juga:** Filter lebih spesifik:
  ```bash
  grep "Only in /home/student/challenge04/comdir02"
  ```

- **`comm` error "file not sorted":** `comm` butuh input **sorted**. Pakai `sort` dulu.

- **`awk '{print $2}'` untuk Task 2 salah:** Kalau output `Files X and Y differ` → `$2` = X. Kalau output `Only in` → format beda, `$2` bukan path.

- **File output sudah ada:** `>` akan **overwrite**. Mau append pakai `>>`.

---

## 💡 Catatan & Insight Pribadi

### Beda `Files differ` vs `Only in`

| Format Output | Arti | Task Type |
|---|---|---|
| `Files X and Y differ` | File ada di **kedua** sisi, isinya beda | Compare contents |
| `Only in <dir>: <file>` | File **hanya ada di satu** sisi | Find unique files |

**Pelajaran:** "Compare directories" bisa berarti 2 hal. Baca output mentah dulu, baru tentukan strategi parsing.

### Parsing `diff -rq` — Pola Umum

**Untuk `Files ... differ`:**
```bash
diff -rq dir1 dir2 | grep "^Files" | awk '{print $2}'
```

**Untuk `Only in ...`:**
```bash
diff -rq dir1 dir2 | grep "Only in" | sed 's/Only in \(.*\): \(.*\)/\1\/\2/'
```

**Untuk nama file saja:**
```bash
diff -rq dir1 dir2 | grep "Only in" | awk '{print $NF}'
```

### `sed` — Substitusi Pattern

**Syntax dasar:**
```bash
sed 's/pattern_lama/pattern_baru/'
```

**Capture group:**
```bash
sed 's/\(grup1\) \(grup2\)/\2 \1/'   # swap grup1 dan grup2
```

**Contoh di lab ini:**
```
Input:  Only in /path/to/dir: file.txt
Pattern: Only in \(.*\): \(.*\)
Output: /path/to/dir/file.txt
```

### `diff` vs `comm` vs `cmp`

| Tool | Fungsi | Untuk |
|---|---|---|
| `diff` | Bandingkan baris per baris | File teks & direktori |
| `comm` | Bandingkan **sorted** line | File sorted (list) |
| `cmp` | Bandingkan byte per byte | File binary |

### `diff` dengan "Ignore" Options

| Opsi | Ignore |
|---|---|
| `-i` | Case (huruf besar/kecil) |
| `-w` | Whitespace |
| `-b` | Perubahan jumlah whitespace |
| `-B` | Baris kosong |
| `-Z` | Whitespace di akhir baris |

### Real-World Use Cases

- **Config diff:** `diff /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak` — cek perubahan config.
- **Backup verification:** `diff -r /data /backup/data` — pastikan backup identik.
- **Deploy check:** `diff -r app-v1 app-v2` — lihat file mana yang berubah.
- **Code review:** `git diff` (pakai engine `diff` juga).
- **Security audit:** Bandingkan file binary original vs current → deteksi modifikasi.

### Kasus Nyata

- **DevOps:** Bandingkan config server A vs server B → cari drift.
- **Sysadmin:** Bandingkan `/etc` sebelum & sesudah update → lihat yang berubah.
- **Migration:** Bandingkan direktori lama vs baru → pastikan semua file tercopy.

### Pelajaran Kunci dari Challenge Ini

1. **`diff`** = alat utama membandingkan file & direktori.
2. **`-i`** = ignore case untuk Task 1.
3. **`-rq`** = recursive + brief untuk direktori.
4. **`Files differ`** = isi beda; **`Only in`** = file unik.
5. **`sed` capture group** = extract absolute path dari `Only in`.
6. **`awk '{print $NF}'`** = ambil nama file saja (kolom terakhir).
7. **Baca output mentah dulu** sebelum parsing — jangan asumsi formatnya.
8. **Log grading** kasih clue format yang diinginkan.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `diff file1 file2` | Bandingkan 2 file |
| `diff -i file1 file2` | Ignore case |
| `diff -u file1 file2` | Unified format |
| `diff -r dir1 dir2` | Recursive |
| `diff -rq dir1 dir2` | Recursive + brief |
| `comm file1 file2` | Bandingkan sorted |
| `comm -13 file1 file2` | Hanya file2 only |
| `cmp file1 file2` | Bandingkan byte-by-byte |
| `grep "pattern"` | Filter baris |
| `awk '{print $N}'` | Extract kolom |
| `sed 's/pattern/replace/'` | Substitusi string |
| `cmd > file` | Redirect overwrite |

**Contoh task lengkap:**

```bash
# Task 1 — Ignore case
diff -i fileA.txt fileB.txt > results01.txt

# Task 2 — Extract absolute path file yang hanya ada di satu sisi
diff -rq compare01 compare02 | grep "Only in" | sed 's/Only in \(.*\): \(.*\)/\1\/\2/' > results02.txt

# Task 3 — Hanya ada di comdir02 (nama file saja)
diff -rq comdir01 comdir02 | grep "Only in.*comdir02" | awk '{print $NF}' > results03.txt
```

**Alternatif Task 3 pakai `comm`:**

```bash
comm -13 <(ls comdir01 | sort) <(ls comdir02 | sort) > results03.txt
```

**Cek hasil:**

```bash
# Lihat output mentah diff (debug)
diff -rq compare01 compare02

# Lihat hasil parsing
cat results01.txt
cat results02.txt
cat results03.txt
```

---

## 📌 Kesimpulan

Lab Challenge 4 ini menguji kemampuan **membandingkan file & direktori** — skill penting untuk sysadmin, DevOps, dan developer. Yang diuji:

- **`diff`** dengan opsi `-i` (case), `-r` (recursive), `-q` (brief).
- **Parsing output** dengan `grep` + `awk` + `sed` untuk extract info spesifik.
- **Bedakan `Files differ` vs `Only in`** — dua jenis perbedaan yang berbeda.
- **Extract absolute path** dengan `sed` capture group.

Kemampuan ini adalah **fondasi** untuk:
- **Verifikasi backup** — pastikan backup identik dengan sumber.
- **Config drift detection** — cari perbedaan config antar server.
- **Code review** — `git diff` pakai engine yang sama.
- **Security audit** — deteksi modifikasi file penting.

Yang paling berharga dari challenge ini: **pelajaran bahwa asumsi itu bahaya**. Awalnya saya pakai `grep "^Files"` untuk Task 2, tapi ternyata output yang muncul adalah `Only in` — bukan `Files differ`. Setelah baca output mentah, baru tahu format yang benar dan bisa parsing dengan `sed`.

**Motto troubleshooting:** *Jalankan perintah mentah dulu, baca outputnya, baru bikin pipeline.* Jangan asumsi formatnya.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
