# Lab Challenge 3 — Working with Partition

**Course:** Linux System Administration (Adinusa)
**Topic:** Menambah disk, partisi, filesystem, mount, dan fstab dengan LABEL
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Menambahkan **disk baru** ke VM (VirtualBox)
- Membuat **partisi** di disk baru
- Membuat **filesystem ext4** dengan **label**
- **Mount** filesystem ke direktori
- Konfigurasi **auto-mount** via `/etc/fstab` menggunakan **LABEL**

---

## 📘 Konsep Dasar

**Alur lengkap:**

```
Disk baru → Partisi → mkfs + label → mount → fstab (auto-mount)
```

| Tahap | Perintah | Tools |
|---|---|---|
| 1. Tambah disk | GUI VirtualBox | — |
| 2. Buat partisi | `parted` / `fdisk` | GPT/MBR |
| 3. Format + label | `mkfs -t ext4 -L LABEL` | ext4 |
| 4. Mount | `mount LABEL=xxx /path` | — |
| 5. Auto-mount | Edit `/etc/fstab` | `LABEL=` |

**Kenapa pakai LABEL?**
- Nama device (`/dev/sdd1`) bisa **berubah** antar reboot (tergantung urutan deteksi).
- **LABEL** konsisten — tidak berubah.
- Lebih aman untuk sistem dengan banyak disk.

**Kenapa GPT lebih baik dari MBR?**
- MBR max 4 primary partition, max disk 2 TB.
- GPT max 128 partition, max disk sangat besar.

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-challenge-3
```

### Tambah Disk Baru di VirtualBox

1. **Shutdown VM** (matikan, bukan save state).
2. Buka **VirtualBox Manager** → pilih VM → **Settings** → **Storage**.
3. Klik **Add Hard Disk** → **Create new disk**.
4. Pilih **VDI (VirtualBox Disk Image)**.
5. Pilih **Dynamically allocated**.
6. Set size: **2 GB**.
7. **OK** → start VM lagi.

Disk baru akan terdeteksi sebagai `/dev/sdd`.

---

## 🧪 Challenge Tasks

### Soal

1. Tambahkan disk **2 GB** ke VM.
2. Buat partisi **1 GiB** di `/dev/sdd` → `/dev/sdd1` (partisi pertama).
3. Buat filesystem **ext4** di `/dev/sdd1` dengan label **`appcache`**.
4. Mount filesystem ke **`/app`**.
5. Konfigurasi **auto-mount** via `/etc/fstab` menggunakan **`LABEL=appcache`**.

### ✅ Solusi

#### 1. Verifikasi disk terdeteksi

```bash
$ lsblk
```

Cari `sdd` (2 GB, tanpa partisi).

#### 2. Buat partisi 1 GiB di /dev/sdd

```bash
$ sudo parted /dev/sdd
```

Di prompt `(parted)`:

```
(parted) mklabel gpt
(parted) mkpart primary ext4 1MiB 1025MiB
(parted) quit
```

**Penjelasan:**
- `mklabel gpt` → buat table GPT baru.
- `mkpart primary ext4 1MiB 1025MiB` → partisi 1 dari 1 MiB sampai 1025 MiB ≈ **1024 MiB = 1 GiB**.
- Start `1MiB` → alignment optimal.

**Verifikasi:**

```bash
$ lsblk /dev/sdd
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdd      8:48   0    2G  0 disk
└─sdd1   8:49   0 1023M  0 part
```

#### 3. Format ext4 dengan label `appcache`

```bash
$ sudo mkfs -t ext4 -L appcache /dev/sdd1
```

**Penjelasan:**
- `-t ext4` → tipe filesystem.
- `-L appcache` → label.

**Verifikasi:**

```bash
$ sudo blkid /dev/sdd1
/dev/sdd1: LABEL="appcache" UUID="..." TYPE="ext4"
```

#### 4. Buat direktori mount & mount

```bash
$ sudo mkdir -p /app
$ sudo mount LABEL=appcache /app
```

**Verifikasi:**

```bash
$ df -h | grep app
/dev/sdd1        ...   1.0G   ...   /app
```

#### 5. Konfigurasi `/etc/fstab` untuk auto-mount

**Backup dulu:**

```bash
$ sudo cp /etc/fstab /etc/fstab.bak
```

**Edit `/etc/fstab`:**

```bash
$ sudo nano /etc/fstab
```

Tambahkan baris:

```
LABEL=appcache  /app  ext4  defaults  0  2
```

**Penjelasan kolom fstab:**

| Kolom | Nilai | Arti |
|---|---|---|
| 1 | `LABEL=appcache` | Identifikasi device via label |
| 2 | `/app` | Mount point |
| 3 | `ext4` | Tipe filesystem |
| 4 | `defaults` | Opsi mount default |
| 5 | `0` | Tidak di-dump |
| 6 | `2` | Urutan fsck (untuk non-root) |

**Test fstab (WAJIB sebelum reboot):**

```bash
# Unmount dulu
$ sudo umount /app

# Test mount dari fstab
$ sudo mount -a

# Verifikasi
$ df -h | grep app
```

Kalau `mount -a` **berhasil tanpa error**, fstab valid. Kalau ada error, **perbaiki dulu** sebelum reboot — kalau tidak, sistem bisa **gagal boot**.

---

## 🔍 Verifikasi Akhir

### 1. Cek partisi

```bash
$ lsblk /dev/sdd
$ sudo parted /dev/sdd print
```

### 2. Cek filesystem & label

```bash
$ sudo blkid /dev/sdd1
```

Harus menampilkan `LABEL="appcache"` dan `TYPE="ext4"`.

### 3. Cek mount

```bash
$ df -h | grep app
$ mount | grep app
```

### 4. Cek fstab

```bash
$ grep appcache /etc/fstab
LABEL=appcache  /app  ext4  defaults  0  2
```

### 5. Test reboot (opsional tapi disarankan)

```bash
$ sudo reboot
```

Setelah login ulang:

```bash
$ df -h | grep app
```

Filesystem harus **otomatis ter-mount**.

### 6. Grading

```bash
$ nusactl grade linlab-challenge-3
```

---

## 🐛 Troubleshooting & Kesalahan

- **Disk `/dev/sdd` tidak muncul:** Cek `lsblk` — mungkin nama device berbeda (`sdc`, `sde`). Atau disk belum di-attach dengan benar di VirtualBox.

- **`parted: unrecognised disk label`:** Disk masih MBR atau kosong. Jalankan `mklabel gpt` dulu.

- **`mkfs.ext4: Device or resource busy`:** Partisi sedang ter-mount. Unmount dulu: `sudo umount /dev/sdd1`.

- **`mount: /app: special device LABEL=appcache does not exist`:** Label salah atau belum di-set. Cek `sudo blkid /dev/sdd1`.

- **`mount -a` gagal dengan error:** Ada typo di `/etc/fstab`. **JANGAN reboot** sebelum diperbaiki — bisa gagal boot. Perbaiki, test `mount -a` lagi.

- **Setelah reboot, `/app` tidak ter-mount:** Cek:
  - `dmesg | tail` untuk error kernel
  - `systemctl status` untuk error service
  - Kemungkinan label hilang (jarang) atau fstab typo.

- **`fstab` rusak = sistem tidak boot:** Kalau sudah terjadi, boot ke **rescue mode**:
  ```
  1. GRUB → Advanced options → (recovery mode)
  2. Pilih 'root' → masuk root shell
  3. mount -o remount,rw /
  4. nano /etc/fstab   → perbaiki
  5. reboot
  ```

- **Salah device `/dev/sdd` (bukan sdb/sdc):** Di lab ini, disk baru biasanya `sdd` karena `sda` (system), `sdb`/`sdc` (dari lab sebelumnya). Cek dengan `lsblk` — device yang belum punya partisi.

- **`parted mkpart` tidak menerima `ext4` sebagai argumen:** Syntax `parted` beda versi. Coba:
  ```
  (parted) mkpart primary 1MiB 1025MiB
  ```
  Tanpa `ext4`. Tipe filesystem di-set saat `mkfs`.

- **Partisi tidak terdeteksi setelah `parted`:** Jalankan `sudo partprobe /dev/sdd` untuk reload partition table.

- **fstab pakai UUID vs LABEL:** Keduanya valid. LABEL lebih mudah dibaca, UUID lebih unik. Untuk lab, LABEL sesuai permintaan.

- **`mkdir -p /app` bikin direktori, tapi `mount` overwrite isinya:** Kalau `/app` sudah ada isinya, akan "tertutup" oleh mount. Cek dengan `mount` — isi asli masih ada di bawah mount point.

---

## 💡 Catatan & Insight Pribadi

### Alur Partisi yang Benar

```
1. Cek disk: lsblk
2. Buat table: parted mklabel gpt
3. Buat partisi: parted mkpart
4. Reload table: partprobe (opsional)
5. Format: mkfs -t ext4 -L LABEL
6. Cek label: blkid
7. Mount sementara: mount LABEL=xxx /path
8. Tambah ke fstab: LABEL=xxx /path ext4 defaults 0 2
9. Test: mount -a
10. Reboot (kalau perlu)
```

### `parted` vs `fdisk`

| Aspek | `parted` | `fdisk` |
|---|---|---|
| GPT | ✅ | ✅ (modern) |
| MBR | ✅ | ✅ |
| Ukuran disk | Sanga besar | Terbatas |
| Alignment | Otomatis | Manual |
| Mode script | `-s` | Tidak |

**Rekomendasi:** Pakai `parted` untuk disk modern (GPT), `fdisk` untuk MBR legacy.

### Satuan di `parted`

| Unit | Arti |
|---|---|
| `MiB` | Mebibyte (1024 × 1024) |
| `MB` | Megabyte (1000 × 1000) |
| `GiB` | Gibibyte (1024 × 1024 × 1024) |
| `GB` | Gigabyte (1000 × 1000 × 1000) |

**Contoh:** `1MiB` = 1.048.576 byte, `1MB` = 1.000.000 byte. Untuk partisi, pakai **MiB/GiB** (binary) biar rapi.

### Kolom di `/etc/fstab`

```
LABEL=appcache  /app  ext4  defaults  0  2
    │            │     │      │       │  │
    │            │     │      │       │  └── fsck pass
    │            │     │      │       └───── dump
    │            │     │      └───────────── opsi
    │            │     └──────────────────── tipe
    │            └────────────────────────── mount point
    └─────────────────────────────────────── device
```

| Kolom | Nilai umum |
|---|---|
| Device | `/dev/sdX1`, `LABEL=xxx`, `UUID=xxx` |
| Mount point | `/app`, `/data`, `/mnt/backup` |
| Tipe | `ext4`, `xfs`, `btrfs`, `swap` |
| Opsi | `defaults`, `noatime`, `nofail` |
| Dump | `0` (tidak di-dump), `1` (dump) |
| Fsck | `0` (tidak), `1` (root), `2` (lain) |

### Opsi `nofail` — Kapan Dipakai?

Untuk partisi **non-esensial** (misal USB, disk tambahan), tambahkan `nofail`:

```
LABEL=appcache  /app  ext4  defaults,nofail  0  2
```

**Efek:** Kalau partisi tidak ada saat boot, sistem **tetap boot** (tidak stuck di emergency mode).

**Tanpa `nofail`:** Kalau partisi hilang, sistem **gagal boot**.

**Rekomendasi:** Untuk disk tambahan, **selalu pakai `nofail`**.

### `mount -a` — Wajib Test Sebelum Reboot

**Ini aturan emas sysadmin.** Setelah edit `/etc/fstab`:
1. `sudo umount /path`
2. `sudo mount -a`
3. Kalau sukses → aman reboot
4. Kalau gagal → **perbaiki dulu**

Kalau tidak test dan fstab rusak, sistem bisa masuk **emergency mode** saat boot.

### Kasus Nyata

- **Server file:** Tambah disk untuk `/data` dengan LABEL `DATA`, auto-mount via fstab.
- **Database:** Disk terpisah untuk `/var/lib/mysql` dengan `noatime` untuk performa.
- **Backup:** Mount disk eksternal via USB dengan `nofail` (bisa dicabut kapan saja).
- **Cloud VM:** Tambah EBS/disk tambahan, format, mount, add to fstab.

### Pelajaran Kunci dari Challenge Ini

1. **Alur partisi** harus lengkap: disk → partisi → fs → mount → fstab.
2. **LABEL** lebih stabil dari nama device.
3. **`mkfs -L`** untuk set label.
4. **`mount LABEL=xxx`** untuk mount via label.
5. **`/etc/fstab`** untuk persistent mount.
6. **`mount -a`** WAJIB test sebelum reboot.
7. **`nofail`** untuk partisi non-esensial.
8. **GPT** modern, **MBR** legacy.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `lsblk` | Lihat disk & partisi |
| `sudo parted /dev/sdX` | Tool partisi GPT/MBR |
| `(parted) mklabel gpt` | Buat table GPT |
| `(parted) mkpart primary ext4 1MiB 1025MiB` | Buat partisi 1 GiB |
| `(parted) print` | Lihat partition table |
| `sudo mkfs -t ext4 -L LABEL /dev/sdX1` | Format + label |
| `sudo blkid /dev/sdX1` | Cek label & UUID |
| `sudo mkdir -p /path` | Buat mount point |
| `sudo mount LABEL=xxx /path` | Mount via label |
| `sudo umount /path` | Unmount |
| `sudo mount -a` | Test fstab |
| `df -h \| grep app` | Cek mount |
| `sudo cp /etc/fstab /etc/fstab.bak` | Backup fstab |

**Format `/etc/fstab`:**

```
LABEL=appcache  /app  ext4  defaults,nofail  0  2
```

**File penting:**

| File | Isi |
|---|---|
| `/etc/fstab` | Konfigurasi mount persistent |
| `/etc/fstab.bak` | Backup fstab |

**Urutan perintah lengkap:**

```bash
# 1. Tambah disk via VirtualBox GUI (2 GB)

# 2. Verifikasi
lsblk

# 3. Buat partisi
sudo parted /dev/sdd
(parted) mklabel gpt
(parted) mkpart primary ext4 1MiB 1025MiB
(parted) quit

# 4. Format + label
sudo mkfs -t ext4 -L appcache /dev/sdd1

# 5. Verifikasi label
sudo blkid /dev/sdd1

# 6. Buat mount point & mount
sudo mkdir -p /app
sudo mount LABEL=appcache /app

# 7. Verifikasi
df -h | grep app

# 8. Backup fstab
sudo cp /etc/fstab /etc/fstab.bak

# 9. Edit fstab
sudo nano /etc/fstab
# Tambah: LABEL=appcache  /app  ext4  defaults,nofail  0  2

# 10. Test fstab
sudo umount /app
sudo mount -a
df -h | grep app

# 11. Reboot (opsional)
sudo reboot
```

---

## 📌 Kesimpulan

Lab Challenge 3 ini menguji kemampuan **manajemen storage** — dari menambah disk sampai auto-mount. Yang diuji:

- **Partisi disk** dengan `parted` (GPT).
- **Format** dengan `mkfs` + **label**.
- **Mount** manual via `LABEL=`.
- **Auto-mount** via `/etc/fstab`.
- **Test fstab** dengan `mount -a` sebelum reboot.

Kemampuan ini adalah **inti dari sysadmin** — hampir setiap server produksi butuh disk tambahan untuk data, backup, atau database. Pola yang benar:

1. **Partisi** dengan alignment `1MiB`.
2. **Format** dengan ext4 + label deskriptif.
3. **Mount** via label, bukan device name.
4. **Auto-mount** via fstab dengan `nofail` untuk keamanan.
5. **Test** dengan `mount -a` — jangan pernah skip ini.

Yang paling berharga dari challenge ini: **pengalaman langsung** melihat bagaimana satu kesalahan kecil di fstab bisa membuat sistem tidak bisa boot. **Test dulu, reboot kemudian** — itulah moto sysadmin.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
