# Lab Challenge 2 — Finding Files

**Course:** Linux System Administration (Adinusa)
**Topic:** `find` command — filter by permission, size, ownership, extension
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Mencari file berdasarkan **permission** (SUID, SGID, mode spesifik)
- Mencari file berdasarkan **size**
- Mencari file berdasarkan **ownership** dan **extension**
- Menyimpan hasil pencarian dengan **absolute paths** ke file
- **Copy file** ke target directory sambil **mempertahankan permission**

---

## 📘 Konsep Dasar

**`find`** = perintah untuk mencari file/direktori berdasarkan kriteria (nama, ukuran, permission, owner, waktu, dll).

**Syntax:**
```bash
find <path> [options] <expression>
```

| Filter | Opsi | Contoh |
|---|---|---|
| **Nama** | `-name` / `-iname` | `-name "*.txt"` |
| **Permission** | `-perm` | `-perm 0600` |
| **Size** | `-size` | `-size +8k -size -10k` |
| **Owner** | `-user` / `-group` | `-user student` |
| **Tipe** | `-type` | `-type f` (file), `-type d` (dir) |
| **Aksi** | `-exec` / `-print` | `-exec cp {} dest \;` |

**Permission mode di `find`:**

| Cara | Arti |
|---|---|
| `-perm 0644` | Persis `0644` |
| `-perm -u+s` | Bit SUID aktif (setidaknya) |
| `-perm /u+s` | Bit SUID aktif (salah satu) |
| `-perm -4000` | Numeric SUID (setidaknya) |
| `-perm -2000` | Numeric SGID (setidaknya) |
| `-perm -6000` | SUID **dan** SGID |

**Size di `find`:**

| Format | Arti |
|---|---|
| `-size 8k` | Persis 8 KB |
| `-size +8k` | Lebih besar dari 8 KB |
| `-size -8k` | Lebih kecil dari 8 KB |
| `+8k -10k` | Antara 8 KB dan 10 KB |

**Redirection ke file:**
- `>` = overwrite
- `>>` = append
- Untuk output satu per baris, tambahkan `-print` (default untuk `find`).

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-challenge-2
```

---

## 🧪 Challenge Tasks

### Soal

1. **Cari file SUID + SGID** di `/home/student/challenge02/spesials/` → simpan ke `/home/student/challenge02/suid.txt` (absolute paths, satu per baris).

2. **Cari file dengan mode 0600** di `/home/student/challenge02/spesials/` → **copy** ke `/home/student/challenge02/backup/` sambil **pertahankan permission**.

3. **Cari file dengan size antara 8K dan 10K** di `/home/student/challenge02/spesials/` → **copy** ke `/home/student/challenge02/size/`.

4. **Cari file `.txt` yang BUKAN milik user `student`** di `/home/student/challenge02/spesials/` → simpan ke `/home/student/challenge02/extension.txt`.

### ✅ Solusi

```bash
# Pastikan direktori target ada
$ mkdir -p /home/student/challenge02/backup
$ mkdir -p /home/student/challenge02/size

# 1. SUID + SGID → suid.txt
$ sudo find /home/student/challenge02/spesials/ -type f -perm -6000 > /home/student/challenge02/suid.txt

# 2. Mode 0600 → copy ke backup/ (pertahankan permission)
$ sudo find /home/student/challenge02/spesials/ -type f -perm 0600 -exec cp -p {} /home/student/challenge02/backup/ \;

# 3. Size 8K–10K → copy ke size/
$ sudo find /home/student/challenge02/spesials/ -type f -size +8k -size -10k -exec cp {} /home/student/challenge02/size/ \;

# 4. .txt bukan milik student → extension.txt
$ sudo find /home/student/challenge02/spesials/ -type f -name "*.txt" ! -user student > /home/student/challenge02/extension.txt
```

### 🔍 Penjelasan Opsi Penting

**`-perm -6000`**
- Numeric: `6` = `SUID + SGID` (`4` + `2`).
- Tanda `-` (minus) di depan = **"setidaknya"** — file harus punya bit SUID & SGID.
- Contoh: file dengan `-rwsr-sr-x` (`6755`) akan cocok.

**`-perm 0600`**
- **Persis** `0600` — tanpa tanda `-`.
- File `-rw-------` akan cocok. File `-rw-r-----` (0640) **tidak** cocok.

**`-size +8k -size -10k`**
- `+8k` = lebih besar dari 8 KB.
- `-10k` = lebih kecil dari 10 KB.
- Dua kondisi digabung = **AND**.

**`! -user student`**
- `!` = negasi (NOT).
- `-user student` = milik user `student`.
- `! -user student` = **BUKAN** milik user `student`.

**`-exec cp -p {} dest \;`**
- `-exec` = jalankan perintah untuk setiap file yang cocok.
- `{}` = placeholder untuk file yang ditemukan.
- `\;` = akhir perintah `-exec` (wajib diakhiri).
- `cp -p` = copy sambil **pertahankan** permission, ownership (jika bisa), timestamp.

**`-exec cp {} dest \;`** (tanpa `-p`)
- Copy biasa — permission **tidak** dipertahankan (mengikuti umask).
- Untuk challenge 3, tidak disebutkan harus preserve, jadi `cp` biasa cukup.

---

## 🔍 Verifikasi

### 1. Cek file output dibuat

```bash
$ ls -la /home/student/challenge02/
$ ls -la /home/student/challenge02/backup/
$ ls -la /home/student/challenge02/size/
```

### 2. Cek isi `suid.txt`

```bash
$ cat /home/student/challenge02/suid.txt
$ wc -l /home/student/challenge02/suid.txt
```

Setiap baris harus **absolute path** ke file dengan SUID+SGID.

### 3. Cek hasil copy di `backup/`

```bash
$ ls -la /home/student/challenge02/backup/
```

Permission file harus **`-rw-------`** (`0600`) — sesuai aslinya.

### 4. Cek hasil copy di `size/`

```bash
$ ls -la /home/student/challenge02/size/
$ stat /home/student/challenge02/size/* | grep Size
```

Setiap file harus berukuran **8K–10K**.

### 5. Cek isi `extension.txt`

```bash
$ cat /home/student/challenge02/extension.txt
```

Setiap baris harus file `.txt` yang **bukan** milik `student`.

### 6. Grading

```bash
$ nusactl grade linlab-challenge-2
```

---

## 🐛 Troubleshooting & Kesalahan

- **`find: paths must precede expression`:** Opsi `find` harus dalam urutan: **path dulu, baru ekspresi**. Contoh salah: `find -name "*.txt" /path`. Benar: `find /path -name "*.txt"`.

- **`-perm 0600` tidak match:** Karena kamu mungkin menulis `-perm 600` (tanpa `0` di depan). `find` butuh **4 digit** untuk mode spesifik: `0600`, bukan `600`.

- **`-perm -6000` tidak match:** Cek dengan `stat file` apakah file benar-benar punya bit SUID+SGID. Contoh output: `Access: (6755/-rwsr-sr-x)`.

- **`find` jalan lambat:** Cek path — kalau mencari di `/` (root), bisa sangat lambat. Batasi ke `/home/student/challenge02/spesials/`.

- **`cp -p` gagal karena permission:** Kalau file milik root, butuh `sudo`. Cek dengan `sudo find ... -exec sudo cp -p {} dest/ \;`.

- **`-exec` lupa `\;`:** Wajib diakhiri `\;` (backslash + semicolon). Tanpa ini, `find` akan menunggu input.

- **`-exec` lambat:** Untuk file banyak, gunakan `-exec ... +` (plus) yang lebih efisien:
  ```bash
  find /path -name "*.txt" -exec cp {} /dest/ +
  ```
  `+` = jalankan sekali untuk semua file, `\;` = jalankan per file.

- **`! -user` vs `-not -user`:** Keduanya sama. `!` lebih ringkas.

- **File output kosong:** Cek dulu apakah ada file yang cocok:
  ```bash
  $ sudo find /path -type f -perm -6000
  ```
  Kalau kosong, berarti tidak ada file yang memenuhi kriteria — bukan salah perintah.

- **Redirection gagal permission:** Kalau file output di `/home/student/` (bukan root), tidak butuh `sudo` untuk redirect. Tapi `find` mungkin butuh `sudo` untuk baca direktori.

- **`cp` tanpa `-p` mengubah permission:** Karena `cp` default mengikuti umask. Untuk preserve, **wajib** `-p`.

- **Path `spesials/` bukan `specials/`:** Perhatikan ejaan. Lab pakai `spesials` (dengan `e`), bukan `specials`.

---

## 💡 Catatan & Insight Pribadi

### Permission SUID, SGID, Sticky di Octal

| Mode | Bit | Deskripsi |
|---|---|---|
| `4000` | SUID | Set User ID |
| `2000` | SGID | Set Group ID |
| `1000` | Sticky | Sticky bit |
| `6000` | SUID + SGID | Kombinasi |
| `7000` | SUID + SGID + Sticky | Semua |

**Contoh file dengan SUID+SGID:**
```
-rwsr-sr-x   → 6755  → SUID (s) + SGID (s)
-rwsr-xr-x   → 4755  → SUID saja
-rwxr-sr-x   → 2755  → SGID saja
-rwsr-sr-t   → 7755  → SUID + SGID + Sticky
```

### Perbedaan `-perm` di `find`

| Cara | Arti | Contoh |
|---|---|---|
| `-perm 0644` | **Persis** 0644 | Hanya `-rw-r--r--` |
| `-perm -0644` | **Setidaknya** 0644 | 0644, 0655, 0755 (superset) |
| `-perm /0644` | **Salah satu** bit | 0400, 0040, 0004, 0644, dll |

**Rekomendasi:**
- Untuk cari **persis** → `-perm 0644`
- Untuk cari **punya bit SUID/SGID** → `-perm -4000` (dengan minus)
- Untuk cari file dengan **salah satu** permission → `-perm /u+x`

### Size di `find` — Sering Bingung

| Kode | Arti |
|---|---|
| `c` | Byte |
| `k` | Kilobyte (1024 byte) |
| `M` | Megabyte |
| `G` | Gigabyte |

**Konversi:**
- `8k` = 8192 byte
- `8c` = 8 byte
- `8M` = 8 MB

**Tips:** Kalau file terasa tidak match, cek dengan `stat file | grep Size`:
```bash
$ stat file.txt
  Size: 9216  Blocks: 24  IO Block: 4096  regular file
```

### `-exec` — Dua Varian

**Varian 1 — `\;` (per file):**
```bash
find /path -name "*.txt" -exec cp {} /dest/ \;
```
Jalankan `cp` untuk **setiap** file. Cocok jika perintah harus jalan satu per satu.

**Varian 2 — `+` (batch):**
```bash
find /path -name "*.txt" -exec cp {} /dest/ +
```
Jalankan `cp` **sekali** dengan banyak argumen. Lebih cepat, tapi tidak semua perintah mendukung.

**Kapan pakai:**
- `\;` → perintah yang tidak bisa handle multiple file (misal `rm -i`).
- `+` → perintah yang bisa batch (misal `cp`, `rm`).

### Kombinasi Filter `find`

Filter `find` digabung dengan **AND** (default), atau pakai:

| Operator | Arti |
|---|---|
| `-a` | AND (default) |
| `-o` | OR |
| `!` / `-not` | NOT |
| `\( \)` | Grouping |

**Contoh:**
```bash
# File .txt ATAU .log
find /path -name "*.txt" -o -name "*.log"

# File .txt yang BUKAN milik student
find /path -name "*.txt" ! -user student

# File .txt milik student ATAU root
find /path -name "*.txt" \( -user student -o -user root \)
```

### `cp -p` — Preserve Attribute

| Opsi | Preserve |
|---|---|
| `cp` | (default — umask) |
| `cp -p` | Mode + ownership + timestamp |
| `cp -a` | Archive — recursive + semua atribut |
| `cp -r` | Recursive (isi direktori) |

`cp -p` = **p**reserve. Sering dipakai untuk backup yang butuh identik dengan aslinya.

### Real-World Use Cases

- **Audit SUID/SGID:** Cari semua binary SUID di sistem untuk audit keamanan:
  ```bash
  find / -type f -perm -4000 2>/dev/null
  ```
- **Backup file config:** Cari semua `.conf` yang dimodifikasi 7 hari terakhir:
  ```bash
  find /etc -name "*.conf" -mtime -7
  ```
- **Cari file besar:** Untuk housekeeping disk:
  ```bash
  find /var -type f -size +100M
  ```
- **Cari file dengan permission longgar:**
  ```bash
  find / -type f -perm -o+w 2>/dev/null
  ```
- **Cari file milik user yang sudah dihapus** (orphan):
  ```bash
  find / -nouser -o -nogroup 2>/dev/null
  ```

### Kasus Nyata

- **Security audit:** `find -perm -4000` untuk cari SUID binary yang tidak seharusnya ada.
- **Disk cleanup:** `find -size +100M` untuk cari file besar.
- **Migration:** `find -user old_user` untuk cari file milik user lama.
- **Compliance:** `find -name "*.log" ! -user root` untuk cek file log yang tidak seharusnya.

### Pelajaran Kunci dari Challenge Ini

1. **`find`** adalah tool serbaguna untuk filter file.
2. **`-perm`** punya 3 varian: persis, setidaknya, salah satu.
3. **`-size`** pakai `+` (lebih), `-` (kurang), tanpa prefix (persis).
4. **`!`** = negasi, penting untuk filter "BUKAN".
5. **`-exec`** untuk aksi setelah filter — `\;` atau `+`.
6. **`cp -p`** untuk preserve permission saat copy.
7. **Absolute path** penting untuk output yang akan dipakai script.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `find /path -type f` | Cari file saja |
| `find /path -name "*.txt"` | Filter by nama |
| `find /path -iname "*.txt"` | Case-insensitive |
| `find /path -perm 0644` | Persis mode 0644 |
| `find /path -perm -4000` | Setidaknya SUID |
| `find /path -perm -6000` | Setidaknya SUID + SGID |
| `find /path -size +8k` | Lebih besar dari 8K |
| `find /path -size -10k` | Lebih kecil dari 10K |
| `find /path -size +8k -size -10k` | Antara 8K dan 10K |
| `find /path -user student` | Milik user student |
| `find /path ! -user student` | BUKAN milik student |
| `find /path -exec cp {} /dest/ \;` | Copy per file |
| `find /path -exec cp {} /dest/ +` | Copy batch |
| `find /path -exec cp -p {} /dest/ \;` | Copy preserve attribute |
| `cmd > file` | Redirect overwrite |
| `cmd >> file` | Redirect append |

**Contoh pattern lengkap:**

```bash
# SUID + SGID, hanya file
find /path -type f -perm -6000

# Mode 0600
find /path -type f -perm 0600

# Size 8K-10K
find /path -type f -size +8k -size -10k

# .txt bukan milik student
find /path -type f -name "*.txt" ! -user student

# Kombinasi: SUID di /usr/bin, 7 hari terakhir, bukan milik root
find /usr/bin -type f -perm -4000 -mtime -7 ! -user root
```

---

## 📌 Kesimpulan

Lab Challenge 2 ini menguji kemampuan menggunakan **`find`** — salah satu tool paling powerful di Linux. Yang diuji:

- **Filter berdasarkan permission** — beda antara `-perm 0644` (persis), `-perm -0644` (setidaknya), dan `-perm /0644` (salah satu).
- **Filter berdasarkan size** — pakai `+` dan `-` untuk range.
- **Filter berdasarkan ownership** — dengan `!` untuk negasi.
- **Filter berdasarkan extension** — dengan `-name "*.txt"`.
- **Aksi via `-exec`** — untuk copy file, dengan atau tanpa preserve.
- **Output dengan absolute path** — penting untuk integrasi ke script.

Kemampuan ini adalah **fondasi** untuk:
- **Security audit** (cari SUID, cari file world-writable).
- **Disk management** (cari file besar, cari file lama).
- **Migration** (cari file milik user lama).
- **Backup** (cari file yang baru dimodifikasi).

`find` bisa sangat kompleks, tapi kombinasi filter dasar (permission, size, owner, name) sudah cukup untuk 80% kebutuhan.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
