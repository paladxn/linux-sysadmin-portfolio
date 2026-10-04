# Lab 9.1 — Membuat & Mengaktifkan Swap File

**Course:** Linux System Administration (Adinusa)
**Topic:** Swap file: pembuatan, permission, aktivasi, dan persistensi
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membuat **swap file** di Linux menggunakan `fallocate`
- Mengamankan swap file dengan **permission yang benar**
- Mengaktifkan swap file menggunakan `swapon`
- Mengonfigurasi swap file agar aktif otomatis saat **boot** melalui `/etc/fstab`
- Memverifikasi penggunaan swap dengan `swapon --show` dan `free -h`

---

## 📘 Konsep Dasar

| Konsep | Penjelasan |
|---|---|
| **Swap** | Ruang di disk yang dipakai sebagai "memori cadangan" ketika RAM penuh |
| **Swap Partition** | Partisi khusus untuk swap (cara tradisional) |
| **Swap File** | File biasa yang diformat sebagai swap (lebih fleksibel, mudah di-resize) |
| **`/etc/fstab`** | File konfigurasi filesystem yang dibaca saat boot, termasuk mounting swap |
| **`swapon` / `swapoff`** | Perintah untuk mengaktifkan / menonaktifkan swap |

**Kenapa pakai swap file?**
- Lebih fleksibel dari partisi — bisa dibuat, dihapus, atau di-resize kapan saja.
- Tidak butuh partisi khusus — bisa ditaruh di filesystem mana pun.
- Cocok untuk VM dan container.

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-009-1
```

---

## 📘 Guided Example

### 1. Membuat file untuk swap

```bash
$ sudo fallocate -l 2G /swapfile
```

**Penjelasan:**
- `fallocate -l 2G` → alokasikan file berukuran **2 GB**.
- `/swapfile` → nama file swap di root filesystem.

**Alternatif** kalau `fallocate` tidak tersedia:

```bash
$ sudo dd if=/dev/zero of=/swapfile bs=1M count=2048
```

### 2. Set permission yang aman

```bash
$ sudo chmod 600 /swapfile
```

**Kenapa?** Swap file berisi data memori yang bisa saja mengandung informasi sensitif (password, kunci enkripsi, isi database). Hanya **root** yang boleh membaca/menulis.

Hasilnya:

```
-rw------- 1 root root 2.0G ... /swapfile
```

### 3. Format file sebagai swap area

```bash
$ sudo mkswap /swapfile
```

Contoh output:

```
Setting up swapspace version 1, size = 2 GiB (2147479552 bytes)
no label, UUID=...
```

Perintah ini menulis **header swap** di awal file sehingga kernel mengenali file ini sebagai swap.

### 4. Aktifkan swap file

```bash
$ sudo swapon /swapfile
```

Setelah perintah ini, swap langsung aktif untuk sesi saat ini.

### 5. Buat permanent di `/etc/fstab`

```bash
$ echo '/swapfile swap swap defaults 0 0' | sudo tee -a /etc/fstab
```

**Penjelasan kolom `/etc/fstab`:**

| Kolom | Nilai | Arti |
|---|---|---|
| 1 | `/swapfile` | Device / file |
| 2 | `swap` | Mount point (untuk swap selalu `swap`) |
| 3 | `swap` | Filesystem type |
| 4 | `defaults` | Opsi mount |
| 5 | `0` | Tidak di-dump |
| 6 | `0` | Tidak di-fsck saat boot |

`tee -a` = **append** ke file (tidak menimpa).

### 6. Verifikasi swap

```bash
$ sudo swapon --show
```

Contoh output:

```
NAME      TYPE  SIZE  USED PRIO
/swapfile file    2G    0B   -2
```

### 7. (Opsional) Cek memori total termasuk swap

```bash
$ free -h
```

Contoh output:

```
              total        used        free      shared  buff/cache   available
Mem:           1.9Gi       362Mi       1.3Gi       1.0Mi       369Mi       1.4Gi
Swap:          2.0Gi          0B       2.0Gi
```

**Kolom `Swap`** sekarang menampilkan total 2.0 Gi — artinya swap file sudah dikenali.

---

## 🧪 Practice Task

### Soal

1. Buat swap file baru bernama `/swapfile2` berukuran **1 GB**.
2. Set permission `600` pada file tersebut.
3. Format sebagai swap dengan `mkswap`.
4. Aktifkan dengan `swapon`.
5. Verifikasi dengan `swapon --show` — pastikan ada dua baris (swapfile dan swapfile2).
6. Nonaktifkan dengan `swapoff /swapfile2`.
7. Hapus file `/swapfile2` menggunakan `rm`.

### ✅ Solusi

```bash
# 1. Buat file 1 GB
$ sudo fallocate -l 1G /swapfile2

# 2. Set permission
$ sudo chmod 600 /swapfile2

# 3. Format sebagai swap
$ sudo mkswap /swapfile2

# 4. Aktifkan
$ sudo swapon /swapfile2

# 5. Verifikasi
$ sudo swapon --show
# NAME       TYPE  SIZE  USED PRIO
# /swapfile  file    2G    0B   -2
# /swapfile2 file    1G    0B   -3

# 6. Nonaktifkan
$ sudo swapoff /swapfile2

# 7. Hapus file
$ sudo rm /swapfile2
```

### 🔍 Hasil Akhir yang Diharapkan

Setelah semua langkah:

```bash
$ sudo swapon --show
NAME      TYPE  SIZE  USED PRIO
/swapfile file    2G    0B   -2
```

Hanya `/swapfile` yang tersisa (dari guided example).

---

## 🐛 Troubleshooting & Kesalahan

- **`fallocate` gagal di filesystem tertentu:** `fallocate` tidak bekerja di semua filesystem (misal ZFS, atau ext4 dengan `extent` tertentu). Kalau gagal dengan error *"fallocate failed: Operation not supported"*, gunakan `dd` sebagai alternatif:
  ```bash
  $ sudo dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress
  ```

- **Swap file sudah pernah dipakai:** Kalau `/swapfile` sudah ada dan sudah jadi swap, `mkswap` akan menampilkan peringatan. Untuk memaksa format ulang, tambahkan `-f`:
  ```bash
  $ sudo mkswap -f /swapfile
  ```

- **Permission 600 tapi owner bukan root:** `chmod 600` hanya mengatur permission, bukan owner. Pastikan file dimiliki root:
  ```bash
  $ sudo chown root:root /swapfile
  $ sudo chmod 600 /swapfile
  ```
  Kalau owner-nya user biasa, kernel akan menolak swap dengan error *"swapon failed: Operation not permitted"*.

- **Lupa tambah di `/etc/fstab`:** Swap akan aktif **hanya sampai reboot**. Setelah reboot, swap hilang. Solusinya: tambahkan entry di `/etc/fstab` seperti di langkah 5.

- **Salah edit `/etc/fstab` bisa bikin sistem tidak boot:** Ini file kritis. Kalau ada typo, sistem bisa gagal boot. Selalu **verifikasi dengan `sudo mount -a`** setelah edit:
  ```bash
  $ sudo mount -a
  ```
  Kalau tidak ada output/error, berarti fstab valid. Kalau ada error, perbaiki sebelum reboot.

- **`tee` tanpa `sudo` gagal:** `/etc/fstab` dimiliki root. Kalau lupa `sudo`, akan muncul *"Permission denied"*. Solusinya:
  ```bash
  $ echo '/swapfile swap swap defaults 0 0' | sudo tee -a /etc/fstab
  ```

- **Swap tidak muncul di `swapon --show`:** Cek apakah `swapon` benar-benar berhasil. Kadang errornya tidak terlihat karena output-nya di stderr. Cek dengan:
  ```bash
  $ sudo swapon /swapfile
  $ echo $?    # harus 0
  ```

- **Swap file di filesystem yang di-mount dengan `noexec` atau `nosuid`:** Meskipun biasanya tidak masalah, beberapa filesystem modern membatasi swap file. Kalau ragu, taruh di root filesystem yang aman.

---

## 💡 Catatan & Insight Pribadi

- **Swap vs RAM:** Swap bukan pengganti RAM. Akses ke swap jauh lebih lambat dari RAM (karena lewat disk). Swap hanya membantu mencegah **OOM (Out of Memory)** saat RAM sudah penuh.

- **Ukuran swap yang ideal:**
  - **RAM ≤ 2 GB** → swap = 2× RAM.
  - **RAM 2–8 GB** → swap = RAM.
  - **RAM > 8 GB** → swap = setengah RAM, minimal 4 GB.
  - **Untuk server produksi** → swap bisa kecil saja (1–2 GB) karena tujuannya hanya mencegah crash, bukan menambah performa.

- **Swap file vs swap partition:**

  | Aspek | Swap File | Swap Partition |
  |---|---|---|
  | Fleksibilitas | ✅ Tinggi (mudah resize) | ❌ Rendah |
  | Performa | Sedikit lebih lambat | Sedikit lebih cepat |
  | Cocok untuk | VM, container, modern Linux | Server kuno, LVM |
  | Setup | Mudah | Butuh partisi khusus |

  **Rekomendasi modern:** Pakai swap file, kecuali butuh performa maksimal (misal untuk hibernation).

- **`swappiness`:** Parameter kernel yang mengatur seberapa agresif kernel memindahkan data dari RAM ke swap. Cek dengan:
  ```bash
  $ cat /proc/sys/vm/swappiness
  # default biasanya 60
  ```
  Untuk server dengan RAM besar, nilai 10–20 sering disarankan.

- **Hibernation butuh swap ≥ RAM:** Kalau kamu ingin bisa **hibernate**, swap harus ≥ ukuran RAM total.

- **Cek swap aktif di `/proc/swaps`:**
  ```bash
  $ cat /proc/swaps
  ```

- **Hapus swap dengan aman:**
  ```bash
  $ sudo swapoff /swapfile        # matikan dulu
  $ sudo rm /swapfile             # baru hapus
  ```
  Jangan hapus file swap sebelum `swapoff` — kernel masih menggunakannya.

- **Cek UUID swap:** Kalau swap file punya UUID, bisa dipakai di `/etc/fstab` sebagai alternatif path:
  ```bash
  $ sudo blkid /swapfile
  ```
  Contoh:
  ```
  /swapfile: UUID="abc-123-def" TYPE="swap"
  ```
  Lalu di fstab:
  ```
  UUID=abc-123-def none swap sw 0 0
  ```

- **Swap di VM cloud:** Di VM cloud (AWS, GCP, Azure), biasanya **swap dinonaktifkan secara default**. Membuat swap file sangat berguna untuk VM kecil (RAM 1 GB) yang sering kehabisan memori.

- **Kasus nyata:** Server dengan RAM kecil (1–2 GB) yang menjalankan database atau aplikasi Java sering crash karena OOM. Menambahkan swap file 2 GB bisa jadi "penyelamat" sementara sambil menambah RAM. Tapi jangan jadikan solusi permanen — kalau swap sering dipakai, artinya RAM memang kurang.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `fallocate -l 2G /swapfile` | Buat file berukuran 2 GB |
| `dd if=/dev/zero of=/swapfile bs=1M count=2048` | Alternatif pembuatan file |
| `chmod 600 /swapfile` | Set permission ketat (hanya root) |
| `chown root:root /swapfile` | Set owner ke root |
| `mkswap /swapfile` | Format file sebagai swap |
| `swapon /swapfile` | Aktifkan swap |
| `swapoff /swapfile` | Nonaktifkan swap |
| `swapon --show` | Tampilkan daftar swap aktif |
| `free -h` | Tampilkan total memori termasuk swap |
| `cat /proc/swaps` | Info swap dari kernel |
| `sudo mount -a` | Uji validitas `/etc/fstab` |
| `blkid /swapfile` | Cek UUID swap file |

---

## 📌 Kesimpulan

Lab ini memberikan pemahaman praktis tentang **manajemen swap di Linux** — keterampilan yang sangat penting terutama di server dengan RAM terbatas. Yang paling berharga dari lab ini adalah pemahaman bahwa swap file **harus dibuat persistent** melalui `/etc/fstab`, karena tanpa itu swap akan hilang setelah reboot. Selain itu, perhatian pada **permission** (600, owner root) menunjukkan betapa seriusnya Linux memperlakukan keamanan data memori.

Konsep ini juga membuka pintu untuk topik lanjutan seperti **`swappiness` tuning**, **hibernation**, dan **manajemen memori di server produksi**.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
