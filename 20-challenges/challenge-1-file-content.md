# Lab Challenge 1 — Working with File Content

**Course:** Linux System Administration (Adinusa)
**Topic:** Text processing dengan `grep` — pattern matching & output filtering
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Mengekstrak **log entries** menggunakan perintah text processing (`grep`)
- Memahami **pattern matching**: di posisi mana saja vs hanya di awal baris
- Menyimpan hasil filter ke **file output** tertentu

---

## 📘 Konsep Dasar

**`grep`** = Global Regular Expression Print. Perintah untuk mencari baris yang cocok dengan pola tertentu.

| Opsi | Fungsi |
|---|---|
| `grep "pattern" file` | Cari baris yang mengandung pattern |
| `grep "^pattern" file` | Cari baris yang **diawali** pattern |
| `grep "pattern$" file` | Cari baris yang **diakhiri** pattern |
| `grep -i` | Case-insensitive |
| `grep -v` | Invert match (baris yang **tidak** cocok) |
| `grep -c` | Hitung jumlah baris cocok |
| `grep -n` | Tampilkan nomor baris |
| `grep -E` | Extended regex (untuk pattern kompleks) |

**Anchor `^` di regex:** Menandakan **awal baris**. Tanpa `^`, pattern cocok di posisi mana saja.

**Redirection `>`:** Simpan output ke file (overwrite).

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-challenge-1
```

---

## 🧪 Challenge Tasks

### Soal

1. Ekstrak semua baris dari `/var/log/sample1.log` yang mengandung string **`INFO`**, simpan ke `/home/student/challenge01/output1.log`.

2. Ekstrak semua baris dari `/var/log/sample2.log` yang **diawali** string **`illegal`** (harus di awal baris), simpan ke `/home/student/challenge01/output2.log`.

3. Ekstrak semua baris dari `/var/log/sample1.log` yang mengandung string **`router_forward_getOI`** dengan level **`TRACE`**, simpan ke `/home/student/challenge01/output3.log`.

### ✅ Solusi

```bash
# Pastikan direktori output ada
$ mkdir -p /home/student/challenge01

# 1. INFO di sample1.log
$ grep "INFO" /var/log/sample1.log > /home/student/challenge01/output1.log

# 2. Diawali 'illegal' di sample2.log (pakai anchor ^)
$ grep "^illegal" /var/log/sample2.log > /home/student/challenge01/output2.log

# 3. TRACE + router_forward_getOI di sample1.log
$ grep "TRACE" /var/log/sample1.log | grep "router_forward_getOI" > /home/student/challenge01/output3.log

# Alternatif step 3 (satu grep dengan pattern ganda):
$ grep "TRACE.*router_forward_getOI\|router_forward_getOI.*TRACE" /var/log/sample1.log > /home/student/challenge01/output3.log
```

### 🔍 Penjelasan Opsi Penting

**`grep "INFO" file`**
- Cari baris yang **mengandung** string `INFO` di posisi mana saja.
- Case-sensitive.

**`grep "^illegal" file`**
- `^` = anchor awal baris.
- **Hanya** cocok jika `illegal` ada di **karakter pertama**.
- Baris `ini illegal` **tidak** akan cocok, karena `illegal` bukan di awal.

**Pipe `|` untuk multi-filter**
- Step 3 butuh 2 kondisi: `TRACE` **dan** `router_forward_getOI`.
- Gunakan **pipe** untuk filter bertingkat, atau **regex OR**.

---

## 🔍 Verifikasi

### 1. Cek file output sudah dibuat

```bash
$ ls -la /home/student/challenge01/
-rw-r--r-- 1 student student ... output1.log
-rw-r--r-- 1 student student ... output2.log
-rw-r--r-- 1 student student ... output3.log
```

### 2. Cek isi tiap file

```bash
# Cek output1 — harus mengandung INFO
$ cat /home/student/challenge01/output1.log | head -5
$ grep -c "INFO" /home/student/challenge01/output1.log
# Jumlah baris output1 harus sama dengan jumlah INFO di sample1

# Cek output2 — semua baris diawali 'illegal'
$ head -3 /home/student/challenge01/output2.log
$ grep -c "^illegal" /home/student/challenge01/output2.log
# Jumlah baris = jumlah baris yang diawali illegal

# Cek output3 — harus mengandung TRACE + router_forward_getOI
$ cat /home/student/challenge01/output3.log | head -5
$ grep -c "TRACE" /home/student/challenge01/output3.log
$ grep -c "router_forward_getOI" /home/student/challenge01/output3.log
```

### 3. Bandingkan dengan sumber

```bash
# Hitung INFO di source
$ grep -c "INFO" /var/log/sample1.log

# Bandingkan dengan output1
$ wc -l /home/student/challenge01/output1.log
```

Jumlah baris **harus sama** — ini bukti filter berhasil.

### 4. Grading

```bash
$ nusactl grade linlab-challenge-1
```

---

## 🐛 Troubleshooting & Kesalahan

- **`No such file or directory`:** Pastikan `/home/student/challenge01/` ada. Buat dengan:
  ```bash
  $ mkdir -p /home/student/challenge01
  ```

- **File output kosong:** Cek:
  - Apakah pattern benar (case-sensitive)?
  - Apakah file source punya pattern itu? Uji dulu: `grep "INFO" /var/log/sample1.log | wc -l`
  - Apakah path source benar? `ls -la /var/log/sample*.log`

- **`illegal` di posisi mana saja tercapture:** Lupa pakai `^`. Tanpa `^`, `grep "illegal"` cocok di posisi mana saja. Untuk awal baris, **wajib** `grep "^illegal"`.

- **Step 3 menghasilkan file kosong:** Kemungkinan:
  - Pattern harus **keduanya** ada di baris yang sama.
  - Gunakan pipe: `grep "TRACE" file | grep "router_forward_getOI"` untuk memastikan keduanya ada.
  - Atau pakai regex OR: `grep -E "TRACE.*router_forward_getOI|router_forward_getOI.*TRACE"`

- **Output ke file timpa isi lama:** `>` **overwrite**. Kalau mau append, pakai `>>`. Untuk challenge ini, overwrite memang yang diinginkan.

- **Case sensitivity:** `grep "info"` ≠ `grep "INFO"`. Gunakan `grep -i` kalau tidak yakin case.

- **Pattern dengan karakter khusus:** Kalau pattern mengandung `.` `*` `[` `]` `\`, perlu escape atau `grep -F` (fixed string).

- **Sudah dijalankan tapi file kosong:** Cek permission write ke direktori output:
  ```bash
  $ ls -ld /home/student/challenge01/
  ```

- **Grading gagal case tertentu:** Cek log:
  ```bash
  $ cat /var/log/nusactl/labs/linlab-challenge-1.log
  ```

- **Perbedaan `grep` vs `grep -E`:**
  - `grep` = Basic Regex (BRE) — `|` perlu `\|`, `+` perlu `\+`.
  - `grep -E` = Extended Regex (ERE) — `|`, `+`, `?` langsung aktif.

---

## 💡 Catatan & Insight Pribadi

### Kapan Pakai `^` dan `$`?

| Pattern | Cocok dengan |
|---|---|
| `grep "illegal"` | Baris apa saja yang mengandung `illegal` |
| `grep "^illegal"` | Baris yang **dimulai** `illegal` |
| `grep "illegal$"` | Baris yang **diakhiri** `illegal` |
| `grep "^illegal$"` | Baris yang **persis** `illegal` (satu kata saja) |

> **Analogi:** `^` = "dari depan pintu", `$` = "sampai belakang pintu".

### Multi-Filter: Pipe vs Regex OR

**Dua kondisi AND (`TRACE` dan `router_forward_getOI`):**

| Cara | Perintah |
|---|---|
| Pipe (gampang dibaca) | `grep "TRACE" file \| grep "router_forward_getOI"` |
| Regex (efisien) | `grep -E "TRACE.*router_forward_getOI\|router_forward_getOI.*TRACE" file` |
| PCRE (paling fleksibel) | `grep -P "(?=.*TRACE)(?=.*router_forward_getOI)" file` |

**Rekomendasi:** Pipe lebih mudah dipahami & debug. Regex lebih cepat kalau file sangat besar.

### Log Analysis — Real World

- **Server log:** Filter by level (`INFO`, `WARN`, `ERROR`) untuk melihat masalah.
- **Security log:** Cari `Failed password` atau `authentication failure`.
- **Access log:** Cari IP tertentu atau status code (`200`, `404`, `500`).
- **Application log:** Filter by request ID atau user ID.

**Contoh nyata:**

```bash
# Cari 10 error terakhir
$ grep "ERROR" /var/log/app.log | tail -10

# Cari login gagal hari ini
$ grep "Failed password" /var/log/auth.log | grep "$(date +%b\ %d)"

# Cari request yang lambat (>1000ms)
$ grep -E "response_time=[0-9]{4,}" /var/log/app.log
```

### `>` vs `>>`

| Opsi | Efek |
|---|---|
| `>` | Overwrite — timpa isi lama |
| `>>` | Append — tambah di akhir |

Untuk challenge ini, **`>`** digunakan karena output harus fresh dari filter.

### Contoh Pattern Regex Berguna

| Pattern | Arti |
|---|---|
| `^abc` | Dimulai `abc` |
| `abc$` | Diakhiri `abc` |
| `^$` | Baris kosong |
| `.` | Satu karakter apa saja |
| `.*` | Nol atau lebih karakter |
| `[0-9]+` | Satu atau lebih digit |
| `\bword\b` | Word boundary |

### Kasus Nyata

- **Debug aplikasi:** `grep "ERROR" app.log > error.log` — pisahkan error.
- **Compliance:** `grep "^illegal" transactions.log` — cari pelanggaran.
- **Analytics:** `grep "TRACE" api.log | grep "endpoint_X"` — analisis endpoint.
- **Backup:** Filter log lama sebelum diarsipkan.

### Pelajaran Kunci dari Challenge Ini

1. **`grep`** = alat utama text processing.
2. **`^`** = anchor awal baris — penting untuk pattern yang harus di posisi tertentu.
3. **Pipe `|`** = filter bertingkat (AND logic).
4. **Redirection `>`** = simpan hasil ke file.
5. **Verifikasi dengan `wc -l`** untuk cek jumlah baris.
6. **`grep -c`** untuk hitung cepat.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `grep "pattern" file` | Cari baris mengandung pattern |
| `grep "^pattern" file` | Cari baris diawali pattern |
| `grep "pattern$" file` | Cari baris diakhiri pattern |
| `grep -c "pattern" file` | Hitung baris cocok |
| `grep -i "pattern" file` | Case-insensitive |
| `grep -v "pattern" file` | Invert (baris tidak cocok) |
| `grep -n "pattern" file` | Tampilkan nomor baris |
| `grep -E "pattern1\|pattern2" file` | Extended regex (OR) |
| `cmd > file` | Redirect output (overwrite) |
| `cmd >> file` | Redirect output (append) |
| `cmd \| cmd` | Pipe — output satu ke input lain |
| `wc -l file` | Hitung jumlah baris |

**Contoh kombinasi:**

```bash
# Cari pattern di banyak file
$ grep "ERROR" /var/log/*.log

# Filter bertingkat (AND)
$ grep "TRACE" file | grep "router_forward_getOI"

# Cari pattern + tampilkan nomor baris
$ grep -n "INFO" /var/log/sample1.log

# Cari pattern, hitung berapa banyak
$ grep -c "INFO" /var/log/sample1.log

# Filter, lalu simpan hasil
$ grep "ERROR" file.log > error_report.log

# Cari baris kosong
$ grep "^$" file.txt

# Cari pattern dengan regex
$ grep -E "^[0-9]+" file.txt      # baris diawali angka
```

---

## 📌 Kesimpulan

Lab Challenge 1 ini menguji kemampuan **text processing dengan `grep`** — salah satu skill paling dasar dan paling sering dipakai sysadmin. Yang diuji:

- **Pattern matching** — beda antara match di mana saja vs awal baris.
- **Anchor regex** (`^`, `$`) — untuk kontrol posisi.
- **Pipe (`|`)** — filter bertingkat untuk kondisi kompleks (AND).
- **Redirection (`>`)** — simpan hasil ke file.

Kemampuan ini adalah **fondasi log analysis** — skill yang sangat penting di dunia kerja. Setiap sysadmin pasti berurusan dengan log server, dan `grep` adalah alat pertama yang dipakai untuk **mencari jarum di tumpukan jerami**.

Dari challenge ini, saya belajar bahwa:
- **Detail kecil penting** — beda `^illegal` dan `illegal` bisa menghasilkan output yang sangat berbeda.
- **Pipe bukan sekadar penghubung** — ini cara membangun query log yang kompleks.
- **Verifikasi itu wajib** — jangan asumsikan hasil filter benar tanpa cek `wc -l` atau `grep -c`.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
