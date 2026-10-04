# Lab 9.2 — Disk Usage dengan `df` & `du`

**Course:** Linux System Administration (Adinusa)
**Topic:** Analisis penggunaan disk (`df` dan `du`)
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Menggunakan **`df`** untuk melihat penggunaan filesystem secara keseluruhan dan memfilternya berdasarkan tipe
- Menggunakan **`du`** untuk mengukur ukuran direktori dan file individu
- Menyimpan output perintah ke file agar bisa direview dan diverifikasi nanti

---

## 📘 Konsep Dasar

| Perintah | Fungsi |
|---|---|
| **`df`** | Menampilkan penggunaan **filesystem** (total, used, available, mount point) |
| **`du`** | Menampilkan penggunaan **file & direktori** (ukuran aktual dari konten) |

**Perbedaan kunci:**

> `df` menjawab pertanyaan: *"Berapa banyak ruang disk yang masih tersedia di filesystem ini?"*
> `du` menjawab pertanyaan: *"Berapa banyak ruang yang dipakai oleh direktori/file ini?"*

`df` membaca dari **superblock** filesystem — cepat dan akurat untuk keseluruhan disk.
`du` membaca dari **metadata setiap file** — lebih lambat, tapi bisa per direktori.

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-009-2
```

---

## 📘 Guided Example

### 1. Cek penggunaan disk dengan `df`

```bash
$ df
```

Contoh output:

```
Filesystem     1K-blocks      Used Available Use% Mounted on
/dev/nvme0n1p2 490825920 369866676  95953152  80% /
tmpfs            8076184    146552   7929632   2% /tmp
/dev/nvme0n1p1    306584      6356    300228   3% /boot/efi
```

**Penjelasan kolom:**

| Kolom | Arti |
|---|---|
| `Filesystem` | Nama device atau filesystem |
| `1K-blocks` | Total ukuran (dalam 1K block) |
| `Used` | Sudah terpakai |
| `Available` | Masih tersedia |
| `Use%` | Persentase terpakai |
| `Mounted on` | Mount point |

**Catatan:** Output default `df` dalam **1K block** — agak sulit dibaca.

### 2. Format human-readable: `df -h`

```bash
$ df -h
```

Contoh output:

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1p2  469G  353G   92G  80% /
tmpfs           7.8G  144M  7.6G   2% /tmp
/dev/nvme0n1p1  300M  6.3M  294M   3% /boot/efi
```

Jauh lebih mudah dibaca — ukuran dalam G, M, K.

### 3. Filter hanya filesystem `tmpfs`

```bash
$ df -Th | grep tmpfs
```

**Penjelasan opsi:**

| Opsi | Arti |
|---|---|
| `-T` | Tampilkan kolom **Type** (ext4, tmpfs, xfs, dll) |
| `-h` | Human-readable |
| `\| grep tmpfs` | Filter baris yang mengandung `tmpfs` |

### 4. Eksplorasi `du`

```bash
$ du
```

Output default: menampilkan penggunaan **setiap subdirektori** dalam **block** (biasanya 1K atau 4K). Sangat verbose.

### 5. `du -h` — human-readable

```bash
$ du -h
```

Contoh output:

```
4.0K    ./Documents
16K     ./Downloads
20K     .
```

### 6. `du -sh .` — total ukuran direktori saat ini

```bash
$ du -sh .
```

Contoh output:

```
321G    .
```

**Penjelasan opsi:**

| Opsi | Arti |
|---|---|
| `-s` | **Summary** (hanya total, tidak per subdirektori) |
| `-h` | Human-readable |
| `.` | Direktori saat ini |

### 7. Cek ukuran direktori tertentu

```bash
$ du -sh ~/Downloads
```

### 8. Bandingkan `df` dan `du`

| Perintah | Menjawab pertanyaan |
|---|---|
| `df -h` | Total pemakaian disk di seluruh filesystem |
| `du -sh ~/` | Total pemakaian disk oleh home directory |

**Contoh kasus:** Kamu jalankan `df -h` di `/` dan melihat **80% terpakai**, tapi `du -sh /home/*` hasilnya total hanya 5 GB. Kemungkinan besar ruang dipakai oleh direktori lain (`/var`, `/usr`, atau file log). `du` bisa membantu melacak siapa yang memakai ruang.

---

## 🧪 Practice Task

### Soal

1. Buat file `df_all.txt` yang berisi daftar semua mounted filesystem dalam **human-readable** format.
2. Buat file `df_ext4.txt` yang berisi **hanya** filesystem dengan tipe `ext4`.
3. Cek total ukuran direktori `~/lab92/special/` dan simpan hasilnya ke `du_special.txt`.
4. Cek ukuran **setiap file** di dalam direktori `special/` dan simpan hasilnya ke `du_each.txt`.
5. Tampilkan semua file di direktori `lab92` untuk memastikan file hasil sudah dibuat.

### ✅ Solusi

```bash
# 1. Simpan daftar semua filesystem dalam human-readable
$ df -h > ~/lab92/df_all.txt

# 2. Simpan hanya filesystem bertipe ext4
$ df -Th | grep ext4 > ~/lab92/df_ext4.txt

# 3. Simpan total ukuran direktori special/
$ du -sh ~/lab92/special/ > ~/lab92/du_special.txt

# 4. Simpan ukuran setiap file di dalam special/
$ du -h ~/lab92/special/* > ~/lab92/du_each.txt

# 5. Verifikasi file hasil
$ ls -l ~/lab92/
```

### 🔍 Hasil Akhir yang Diharapkan

```bash
$ ls -l ~/lab92/
-rw-rw-r-- 1 student student ... df_all.txt
-rw-rw-r-- 1 student student ... df_ext4.txt
-rw-rw-r-- 1 student student ... du_special.txt
-rw-rw-r-- 1 student student ... du_each.txt
```

Isi `df_all.txt` (contoh):

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1p2  469G  353G   92G  80% /
tmpfs           7.8G  144M  7.6G   2% /tmp
/dev/nvme0n1p1  300M  6.3M  294M   3% /boot/efi
```

Isi `df_ext4.txt` (contoh):

```
/dev/nvme0n1p2  ext4  469G  353G   92G  80% /
```

Isi `du_special.txt` (contoh):

```
4.0K    /home/student/lab92/special/
```

Isi `du_each.txt` (contoh):

```
4.0K    /home/student/lab92/special/file1.txt
8.0K    /home/student/lab92/special/file2.txt
```

---

## 🐛 Troubleshooting & Kesalahan

- **`df -Th | grep ext4` tidak menampilkan apa-apa:** Kemungkinan di environment lab-mu tidak ada filesystem bertipe `ext4` (misalnya pakai `overlay`, `xfs`, atau `btrfs`). Cek dulu tipe filesystem yang tersedia:
  ```bash
  $ df -Th
  ```
  Lalu sesuaikan filter dengan tipe yang ada. Kalau memang tidak ada ext4, file `df_ext4.txt` boleh kosong atau berisi header saja.

- **`du -sh ~/lab92/special/` error "No such file or directory":** Pastikan direktori `special/` benar-benar ada. Kalau belum, buat dulu:
  ```bash
  $ mkdir -p ~/lab92/special
  $ touch ~/lab92/special/file1.txt ~/lab92/special/file2.txt
  ```

- **Redirection `>` menimpa file lama:** Kalau file sudah ada, `>` akan **menimpa**, bukan menambahkan. Kalau ingin append, gunakan `>>`. Untuk file hasil lab ini, penimpaan adalah perilaku yang diinginkan.

- **`df -h` dan `du -sh` hasilnya berbeda:** Ini normal dan sering membingungkan. `df` menghitung berdasarkan **superblock** (termasuk inode, metadata, reserved blocks), sedangkan `du` menghitung **ukuran aktual file**. Perbedaan bisa mencapai 5–10% pada filesystem besar.

- **`du` sangat lambat di direktori besar:** `du` membaca setiap file untuk mendapatkan ukurannya. Di direktori dengan jutaan file, ini bisa memakan waktu lama. Solusinya: gunakan `du -sh` (summary) untuk menghindari traversal berlebih, atau `du --max-depth=1` untuk membatasi kedalaman.

- **`du` menghitung hard link berkali-kali:** Kalau ada hard link, `du` akan menghitung file yang sama lebih dari sekali secara default. Gunakan opsi `-l` (count links) atau batasi dengan `--count-links` sesuai kebutuhan.

- **Block size membingungkan:** Default `du` menampilkan dalam block (biasanya 4K). Ini kenapa file 1 byte bisa muncul sebagai `4.0K` — karena filesystem mengalokasikan minimum 1 block. Gunakan `du -b` untuk melihat ukuran **aktual dalam byte**:
  ```bash
  $ du -b ~/lab92/special/file1.txt
  ```

- **Symlink dihitung sebagai target:** Secara default, `du` mengikuti symlink. Untuk menampilkan symlink sebagai dirinya sendiri, gunakan `-P` (physical) atau `--no-dereference`:
  ```bash
  $ du -sh --no-dereference ~/lab92/special/
  ```

- **Filter grep case-sensitive:** `grep ext4` tidak akan cocok dengan `EXT4` atau `Ext4`. Kalau tidak yakin, gunakan `grep -i ext4` untuk case-insensitive.

---

## 💡 Catatan & Insight Pribadi

- **`df` vs `du` — kapan pakai yang mana?**
  - **`df`** → saat ingin tahu *"berapa banyak disk yang masih tersisa?"* atau *"apakah disk sudah penuh?"*
  - **`du`** → saat ingin tahu *"direktori mana yang paling banyak memakai ruang?"*

- **Kombinasi klasik untuk mencari "siapa yang memakan disk":**
  ```bash
  $ du -sh /* 2>/dev/null | sort -h
  ```
  Ini menampilkan direktori root yang paling besar, diurutkan dari terkecil ke terbesar.

- **Mencari file terbesar:**
  ```bash
  $ du -ah ~ | sort -h | tail -20
  ```
  Menampilkan 20 file/direktori terbesar di home.

- **Cek inode usage:** `df -i` menampilkan penggunaan **inode** — berguna kalau filesystem penuh tapi tidak ada file besar (kemungkinan terlalu banyak file kecil).

- **`df -hT` — kombinasi paling berguna:**
  - `-h` → human-readable
  - `-T` → tampilkan tipe filesystem

- **`du --max-depth=1`:**
  ```bash
  $ du -h --max-depth=1 ~/
  ```
  Menampilkan ukuran per subdirektori **satu level** saja — sangat berguna untuk overview.

- **Menyimpan output ke file:**
  ```bash
  $ df -h > ~/laporan-disk-$(date +%Y%m%d).txt
  ```
  Teknik ini berguna untuk membuat laporan berkala. Bisa dikombinasikan dengan cron untuk monitoring otomatis.

- **Kasus nyata:**
  - **Server tiba-tiba penuh** → jalankan `df -h` untuk cek filesystem mana yang penuh, lalu `du -sh /* | sort -h` untuk melacak siapa pelakunya.
  - **Log yang membengkak** → biasanya di `/var/log`. Gunakan `du -sh /var/log/*` untuk cek.
  - **Snapshot backup** → biasanya di direktori tersembunyi seperti `.snapshot/` yang tidak muncul di `du` biasa tanpa `-a`.

- **`du` dan `df` berbeda hasilnya di ZFS/Btrfs:** Filesystem modern dengan **deduplication** dan **compression** bisa menampilkan angka berbeda antara `du` dan `df`. `du` melaporkan ukuran logis, `df` melaporkan ukuran fisik.

- **Hati-hati redirection di file yang sama:** Menjalankan `du -sh . > file.txt` di dalam direktori yang sedang diukur bisa membuat `du` membaca `file.txt` yang sedang ditulis — hasilnya bisa aneh. Gunakan `du -sh . > ../file.txt` untuk menghindari.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `df` | Penggunaan filesystem (default 1K block) |
| `df -h` | Penggunaan filesystem (human-readable) |
| `df -Th` | Penggunaan filesystem + tipe |
| `df -i` | Penggunaan inode |
| `df -Th \| grep ext4` | Filter berdasarkan tipe filesystem |
| `du` | Penggunaan per direktori (block) |
| `du -h` | Penggunaan per direktori (human-readable) |
| `du -sh .` | Total ukuran direktori saat ini |
| `du -sh ~/Downloads` | Total ukuran direktori tertentu |
| `du -h dir/*` | Ukuran setiap file di dalam direktori |
| `du -ah \| sort -h \| tail` | File terbesar |
| `du --max-depth=1` | Ukuran per subdirektori (1 level) |

---

## 📌 Kesimpulan

Lab ini memberikan dua alat penting untuk **manajemen disk**: `df` dan `du`. Meskipun sekilas mirip, keduanya menjawab pertanyaan berbeda — `df` untuk melihat gambaran besar filesystem, `du` untuk melacak siapa yang memakai ruang. Kemampuan mengombinasikan keduanya (dan menyimpan output ke file) adalah keterampilan wajib sysadmin, terutama ketika server tiba-tiba kehabisan disk dan kita perlu melacak penyebabnya dengan cepat.

Teknik menyimpan output ke file juga sangat berguna untuk **laporan berkala** — bisa dikombinasikan dengan `cron` untuk membuat log penggunaan disk harian.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
