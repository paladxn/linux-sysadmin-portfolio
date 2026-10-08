# Lab 13.2 — Chmod Octal Digit (Numeric Mode)

**Course:** Linux System Administration (Adinusa)
**Topic:** Mengubah permission file dengan `chmod` (octal/numeric mode)
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Mengubah permission file menggunakan **octal digit** (numeric mode)
- Memahami konversi permission `rwx` ke angka **4-2-1**
- Membedakan efek permission pada **file** dan **direktori**
- Memverifikasi hasil permission dengan `ls -l` dan menguji akses sebagai user lain

---

## 📘 Konsep Dasar

### Konversi Permission ke Angka

Setiap permission punya nilai:

| Permission | Simbol | Angka |
|---|---|---|
| Read | `r` | **4** |
| Write | `w` | **2** |
| Execute | `x` | **1** |

Jumlahkan untuk kombinasi:

| Kombinasi | Simbol | Angka |
|---|---|---|
| No permission | `---` | **0** |
| Execute only | `--x` | **1** |
| Write only | `-w-` | **2** |
| Write + execute | `-wx` | **3** |
| Read only | `r--` | **4** |
| Read + execute | `r-x` | **5** |
| Read + write | `rw-` | **6** |
| Read + write + execute | `rwx` | **7** |

### Format Octal Digit

`chmod` numeric memakai **3 digit** (atau 4 digit dengan special permission):

```
chmod  7  5  5  file
       │  │  └── others
       │  └───── group
       └──────── user
```

Contoh: `chmod 644 file`
- User = 6 = read + write (`rw-`)
- Group = 4 = read (`r--`)
- Others = 4 = read (`r--`)
- Hasil: `-rw-r--r--`

### Efek pada File vs Direktori

| Permission | Efek pada File | Efek pada Direktori |
|---|---|---|
| `r` | Baca isi file | List isi direktori (`ls`) |
| `w` | Ubah isi file | Buat/hapus file di dalamnya |
| `x` | Jalankan file | Masuk (`cd`) ke direktori |

**Penting:** Untuk direktori, kombinasi `r` dan `x` biasanya dipakai bersama. `r` tanpa `x` = bisa list tapi tidak bisa masuk. `x` tanpa `r` = bisa masuk tapi tidak bisa list.

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-013-2
```

---

## 📘 Guided Example

### 1. Buat direktori kerja dan file

```bash
$ mkdir /tmp/lab-13-2
$ cd /tmp/lab-13-2
$ touch file1.txt file2.txt file3.txt
$ mkdir scripts
```

### 2. Lihat permission default

```bash
$ ls -l
```

Output:

```
-rw-r--r-- 1 student student 0 Oct 22 12:00 file1.txt
-rw-r--r-- 1 student student 0 Oct 22 12:00 file2.txt
-rw-r--r-- 1 student student 0 Oct 22 12:00 file3.txt
drwxr-xr-x 2 student student 60 Oct 22 12:00 scripts
```

**Perhatikan:**
- File default: `644` (`rw-r--r--`)
- Direktori default: `755` (`rwxr-xr-x`)

### 3. `chmod 600` — hanya owner yang bisa baca & tulis

```bash
$ chmod 600 file1.txt
$ ls -l file1.txt
-rw------- 1 student student 0 Oct 22 12:00 file1.txt
```

**Breakdown:**
- `6` → user: read + write
- `0` → group: no access
- `0` → others: no access

**Efek:** File ini **hanya bisa** dibaca dan diubah oleh owner. Group dan others tidak bisa apa-apa.

### 4. `chmod 644` — semua bisa baca, hanya owner yang bisa tulis

```bash
$ chmod 644 file2.txt
$ ls -l file2.txt
-rw-r--r-- 1 student student 0 Oct 22 12:00 file2.txt
```

**Breakdown:**
- `6` → user: read + write
- `4` → group: read
- `4` → others: read

**Efek:** Siapa saja bisa membaca, tapi hanya owner yang bisa mengubah.

### 5. `chmod 755` — semua bisa jalankan

```bash
$ chmod 755 file3.txt
$ ls -l file3.txt
-rwxr-xr-x 1 student student 0 Oct 22 12:00 file3.txt
```

**Breakdown:**
- `7` → user: read + write + execute
- `5` → group: read + execute
- `5` → others: read + execute

**Efek:** Siapa saja bisa membaca dan menjalankan, tapi hanya owner yang bisa mengubah.

### 6. `chmod 700` — direktori hanya bisa diakses owner

```bash
$ chmod 700 scripts
$ ls -ld scripts
drwx------ 2 student student 60 Oct 22 12:00 scripts
```

**Breakdown:**
- `7` → user: full access
- `0` → group: no access
- `0` → others: no access

**Efek:** Hanya owner yang bisa `ls` dan `cd` ke direktori ini.

### 7. Verifikasi hasil akhir

```bash
$ ls -l
```

Output:

```
-rw------- 1 student student 0 ... file1.txt
-rw-r--r-- 1 student student 0 ... file2.txt
-rwxr-xr-x 1 student student 0 ... file3.txt
drwx------ 2 student student 60 ... scripts
```

---

## 🧪 Verifikasi Akses sebagai User Lain

### 1. Buat user baru

```bash
$ useradd tester
$ su - tester
```

### 2. Coba akses file

```bash
tester:~$ cd /tmp/lab-13-2

# file1.txt (600) → tidak bisa dibaca
tester:/tmp/lab-13-2$ cat file1.txt
cat: file1.txt: Permission denied

# file2.txt (644) → bisa dibaca
tester:/tmp/lab-13-2$ cat file2.txt
# (output kosong karena file kosong)

# scripts (700) → tidak bisa masuk
tester:/tmp/lab-13-2$ cd scripts
bash: cd: scripts: Permission denied
```

### 3. Kesimpulan Verifikasi

| File/Direktori | Permission | Hasil untuk `tester` |
|---|---|---|
| `file1.txt` | `600` | ❌ Tidak bisa dibaca |
| `file2.txt` | `644` | ✅ Bisa dibaca |
| `file3.txt` | `755` | ✅ Bisa dibaca & dijalankan |
| `scripts/` | `700` | ❌ Tidak bisa diakses |

---

## 🧪 Practice Task

### Soal

1. Buat file `data.txt` dan direktori `private/` di `/tmp/lab-13-2/`.
2. Set permission:
   - `data.txt` → `640` (owner: rw, group: r, others: none)
   - `private/` → `750` (owner: rwx, group: rx, others: none)
3. Verifikasi dengan `ls -l`.
4. Buat user `tester2` dan uji akses.
5. Dokumentasikan hasilnya.

### ✅ Solusi

```bash
# 1. Buat file & direktori
$ cd /tmp/lab-13-2
$ touch data.txt
$ mkdir private

# 2. Set permission
$ chmod 640 data.txt
$ chmod 750 private

# 3. Verifikasi
$ ls -l data.txt private
-rw-r----- 1 student student 0 ... data.txt
drwxr-x--- 2 student student 60 ... private

# 4. Uji akses
$ useradd tester2
$ su - tester2
tester2:~$ cd /tmp/lab-13-2
tester2:/tmp/lab-13-2$ cat data.txt
cat: data.txt: Permission denied
tester2:/tmp/lab-13-2$ cd private
bash: cd: private: Permission denied
```

### 🔍 Hasil yang Diharapkan

`tester2` (bukan owner, bukan group) tidak bisa membaca `data.txt` maupun masuk ke `private/`.

---

## 🐛 Troubleshooting & Kesalahan

- **Hanya 2 digit:** Kalau kamu tulis `chmod 64 file`, ini akan diinterpretasi sebagai `064` — user=0, group=6, others=4. Selalu tulis **3 digit** untuk hasil yang diharapkan.

- **4 digit (special permission):** `chmod 4755` → digit pertama adalah SUID. Kalau tidak sengaja menambahkan digit, hasilnya bisa jadi aneh. Fokus dulu pada 3 digit.

- **Direktori tanpa `x`:** Kalau kamu set direktori ke `644`, orang tidak bisa `cd` ke dalamnya, meskipun bisa `ls` isinya. Untuk direktori, pastikan `x` ada di setidaknya user (biasanya `755` atau `700`).

- **File script tanpa `x`:** Memberi permission `644` pada script membuatnya tidak bisa dijalankan. Butuh `755` atau setidaknya `744`.

- **File data dengan `x`:** Memberi `x` pada file yang bukan script tidak berbahaya, tapi tidak ada gunanya. Biasakan `644` untuk file data.

- **User tidak bisa membaca file yang di-set `640`:** Karena user tersebut bukan owner maupun anggota group. Cek dengan `id user` dan `ls -l` untuk memastikan owner & group-nya.

- **Perubahan tidak langsung terasa:** Kadang kernel butuh waktu singkat untuk flush permission cache. Tunggu beberapa detik, atau coba logout-login ulang untuk user yang terpengaruh.

- **`Permission denied` untuk direktori meskipun ada `r`:** Kalau `x` tidak ada di direktori, kamu bisa `ls` tapi tidak bisa `cd` atau akses file di dalamnya. Pastikan `x` ada.

- **`chmod` di symlink mengubah target:** Ingat bahwa `chmod` pada symlink akan mengubah permission **target**, bukan symlink-nya. Gunakan `chmod -h` di sistem yang mendukung.

- **Ownership salah:** Kalau file dimiliki root tapi kamu login sebagai student, kamu tidak bisa `chmod`-nya. Cek dengan `ls -l` siapa owner-nya. Gunakan `sudo chown` untuk mengubah owner (kalau perlu).

- **Umask mempengaruhi default:** Default permission file baru (`644`) dan direktori baru (`755`) dipengaruhi **umask**. Cek dengan `umask`. Nilai umum adalah `022`.

---

## 💡 Catatan & Insight Pribadi

### Octal vs Symbolic — Kapan Pakai Apa?

| Aspek | Octal | Symbolic |
|---|---|---|
| Singkat | ✅ (`chmod 755`) | ❌ (`chmod u=rwx,g=rx,o=rx`) |
| Jelas | ❌ (harus hafal angka) | ✅ (`u=rwx` jelas) |
| Granular | ❌ (set semua sekaligus) | ✅ (bisa ubah satu bagian) |
| Umum dipakai | ✅ untuk permission standar | ✅ untuk tweak kecil |

**Rekomendasi praktis:**
- **Octal** → untuk set permission standar (644, 755, 600).
- **Symbolic** → untuk tweak kecil (`chmod g+w`) atau reset satu kelompok.

### Konversi Cepat yang Sering Dipakai

| Permission | Angka | Kapan dipakai |
|---|---|---|
| `rw-r--r--` | **644** | File data, config umum |
| `rwxr-xr-x` | **755** | Script, direktori publik |
| `rw-------` | **600** | File sensitif (SSH key, .env) |
| `rwx------` | **700** | Direktori private |
| `rw-rw-r--` | **664** | File kolaborasi (group bisa write) |
| `rwxrwxr-x` | **775** | Direktori kolaborasi |
| `rwxrwxrwx` | **777** | 🚨 **Hindari!** Terbuka untuk semua |

### Cara Hafal Angka 4-2-1

- **4** = Read → ingat "**R**" di awal "**R**ead" = 4 huruf?
- **2** = Write → ingat "**W**" = "double U" = 2?
- **1** = Execute → ingat "**E**" = "**E**" di angka 1 (paling dasar)?

Atau cara paling cepat: hafalkan tabel 0-7 di bagian konsep dasar.

### Kenapa Direktori Butuh `x`?

Untuk direktori, `x` artinya **bisa "traverse"** — masuk dan akses file di dalamnya. Tanpa `x`:
- Kamu bisa `ls` (kalau ada `r`)
- Tapi tidak bisa `cd` atau akses file spesifik

Contoh: direktori dengan `r--` (400) → `ls` bisa, `cd` gagal.
Direktori dengan `--x` (100) → `ls` gagal, `cd` bisa (kalau tahu nama file).

### Special Permission (Preview)

Selain 3 digit, ada **digit ke-4** untuk special permission:

| Digit | Nama | Efek |
|---|---|---|
| `4xxx` | SUID | File dijalankan sebagai owner |
| `2xxx` | SGID | File dijalankan sebagai group |
| `1xxx` | Sticky bit | Hanya owner yang bisa hapus file di direktori |

Contoh: `/usr/bin/passwd` biasanya `4755` (SUID) — supaya user biasa bisa ubah password meskipun file `/etc/shadow` dimiliki root.

Akan dipelajari di lab selanjutnya.

### Best Practice Permission

| Tipe | Rekomendasi |
|---|---|
| File data | `644` |
| Script executable | `755` |
| File rahasia (`.env`, key) | `600` |
| Direktori publik | `755` |
| Direktori private | `700` |
| Shared folder (group) | `775` |
| **Jangan** | `777` di production |

### Kasus Nyata

- **SSH key:** `~/.ssh/id_rsa` harus `600`. Kalau permission longgar, SSH akan menolak.
- **Web server:** File HTML biasanya `644`, direktori `755`.
- **Deployment script:** File script harus `755` agar bisa dieksekusi.
- **Config rahasia:** `.env`, `credentials.json` sebaiknya `600`.
- **Shared folder tim:** Direktori `775` + group yang tepat.

### Pelajaran Kunci dari Lab Ini

1. **Octal digit** = cara singkat set permission.
2. **4-2-1** adalah basis konversi permission.
3. **Efek `x` berbeda** untuk file vs direktori.
4. **`chmod 777` bukan solusi** — itu masalah.
5. **Verifikasi dengan `ls -l`** dan uji sebagai user lain.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `chmod 600 file` | Owner: rw, group/others: none |
| `chmod 644 file` | Owner: rw, group/others: r |
| `chmod 755 file` | Owner: rwx, group/others: rx |
| `chmod 700 dir` | Owner: rwx, group/others: none |
| `chmod 640 file` | Owner: rw, group: r, others: none |
| `chmod 750 dir` | Owner: rwx, group: rx, others: none |
| `ls -l` | Verifikasi permission |
| `ls -ld` | Verifikasi permission direktori |
| `useradd tester` | Buat user untuk testing |
| `su - tester` | Login sebagai user lain |

**Tabel konversi cepat:**

| Angka | Binary | Permission |
|---|---|---|
| 0 | 000 | `---` |
| 1 | 001 | `--x` |
| 2 | 010 | `-w-` |
| 3 | 011 | `-wx` |
| 4 | 100 | `r--` |
| 5 | 101 | `r-x` |
| 6 | 110 | `rw-` |
| 7 | 111 | `rwx` |

---

## 📌 Kesimpulan

Lab ini melengkapi pemahaman tentang **file permission** yang dimulai di Lab 13.1 (symbolic mode). Kalau symbolic mode lebih fleksibel untuk tweak kecil, **octal mode** lebih singkat dan sering dipakai untuk set permission standar.

Yang paling berharga dari lab ini adalah pemahaman bahwa **`chmod` bukan sekadar angka** — di baliknya ada konsep **keamanan berlapis**. Memberi permission yang terlalu longgar (`777`) sama dengan membuka pintu rumah untuk siapa saja. Memberi permission yang terlalu ketat bisa membuat user tidak bisa bekerja.

Kemampuan mengelola permission adalah **inti dari keamanan Linux**, dan akan terus dipakai di topik lanjutan seperti SUID, SGID, sticky bit, ACL, dan SELinux.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
