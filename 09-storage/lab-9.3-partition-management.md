# Lab 9.3 — Manajemen Partisi Disk (`parted` & `fdisk`)

**Course:** Linux System Administration (Adinusa)
**Topic:** Membuat partisi, format ext4, label, mount, dan persistent mount
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membuat dan mengelola **partisi disk** di Linux menggunakan `parted` dan `fdisk`
- Memformat partisi dengan **ext4** dan memberikan **label**
- Melakukan **mount** partisi ke direktori tertentu
- Mengonfigurasi **persistent mount** menggunakan `/etc/fstab`
- Memahami pentingnya label dan opsi `nofail` untuk keamanan boot

---

## 📘 Konsep Dasar

| Konsep | Penjelasan |
|---|---|
| **Partisi** | Pembagian logis dari sebuah disk fisik menjadi beberapa bagian |
| **Partition Table** | Tabel yang menyimpan informasi partisi: **MBR** (lama) atau **GPT** (modern) |
| **Filesystem** | Struktur data di atas partisi (ext4, xfs, btrfs, dll) |
| **Label** | Nama yang diberikan ke filesystem, memudahkan identifikasi |
| **UUID** | Identifier unik untuk setiap filesystem |
| **`/etc/fstab`** | File konfigurasi mount yang dibaca saat boot |
| **`nofail`** | Opsi mount yang memastikan sistem tetap boot meskipun device tidak ada |

**Kenapa pakai label/UUID di `/etc/fstab`?**
> Nama device seperti `/dev/sdc1` bisa berubah tergantung urutan deteksi kernel. Label dan UUID **tidak berubah**, sehingga lebih aman untuk konfigurasi persistent.

---

## 🔧 Persiapan Lab

### 1. Login & start lab

```bash
$ nusactl login
$ nusactl start linlab-009-3
```

### 2. Tambahkan dua disk virtual (via GUI VM)

Buka **Settings Virtual Machine** → **Storage** → **Add Attachment** → **Add Hard Disk**:

| Disk | Ukuran | Device |
|---|---|---|
| Disk 1 | 1 GB | `/dev/sdb` |
| Disk 2 | 600 MB | `/dev/sdc` |

Disk akan muncul sebagai device block baru di sistem.

---

## 📘 Guided Example

### 1. Verifikasi disk yang tersedia

```bash
$ lsblk
```

Contoh output:

```
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda       8:0    0   15G  0 disk
├─sda1    8:1    0   14G  0 part /
├─sda14   8:14   0    4M  0 part
├─sda15   8:15   0  106M  0 part /boot/efi
└─sda16 259:0    0  913M  0 part /boot
sdb       8:16   0    1G  0 disk
sdc       8:32   0  600M  0 disk
```

**Perhatikan:** `/dev/sdb` dan `/dev/sdc` belum punya partisi (tidak ada sub-entri).

### 2. Buat partisi pertama (256 MB) dengan `parted`

```bash
$ sudo parted /dev/sdc
```

Di dalam prompt `(parted)`:

```
(parted) mklabel gpt

(parted) mkpart
Partition name?  []? primary
File system type?  [ext2]? ext4
Start? 1MiB
End? 257MiB

(parted) quit
```

**Penjelasan:**
- `mklabel gpt` → buat partition table GPT baru (menghapus yang lama).
- `mkpart` → buat partisi baru.
- `1MiB` → mulai dari offset 1 MiB (bukan 0) untuk alignment.
- `257MiB` → akhir partisi, sehingga ukurannya ~256 MB.

Hasilnya: partisi **`/dev/sdc1`** dengan ukuran ~256 MB.

### 3. Verifikasi partisi pertama

```bash
$ lsblk /dev/sdc
```

Output:

```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdc      8:32   0  600M  0 disk
└─sdc1   8:33   0  256M  0 part
```

### 4. Buat partisi kedua (256 MB) dengan `fdisk`

```bash
$ sudo fdisk /dev/sdc
```

Di dalam prompt `fdisk`:

```
Command (m for help): n
Partition number (2-128): 2
First sector (press Enter to accept default)
Last sector, +/-sectors or +/-size{K,M,G,T,P}: +256M

Command (m for help): w
```

**Penjelasan:**
- `n` → new partition.
- `2` → nomor partisi (karena `sdc1` sudah ada).
- `+256M` → ukuran 256 MB.
- `w` → write (simpan perubahan).

Hasilnya: partisi **`/dev/sdc2`** dengan ukuran ~256 MB.

### 5. Verifikasi kedua partisi

```bash
$ lsblk /dev/sdc
```

Output:

```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdc      8:32   0  600M  0 disk
├─sdc1   8:33   0  256M  0 part
└─sdc2   8:34   0  256M  0 part
```

### 6. Format kedua partisi dan set label

**Partisi dari parted:**

```bash
$ sudo mkfs -t ext4 -L PARTED_DISK /dev/sdc1
```

**Partisi dari fdisk:**

```bash
$ sudo mkfs -t ext4 -L FDISK_DISK /dev/sdc2
```

**Penjelasan opsi:**

| Opsi | Arti |
|---|---|
| `-t ext4` | Tipe filesystem |
| `-L PARTED_DISK` | Label filesystem |

### 7. Verifikasi label

```bash
$ sudo blkid /dev/sdc1 /dev/sdc2
```

Output:

```
/dev/sdc1: LABEL="PARTED_DISK" UUID="7b135bd8-..." TYPE="ext4"
/dev/sdc2: LABEL="FDISK_DISK" UUID="f57be1e6-..." TYPE="ext4"
```

### 8. Mount kedua partisi

**Buat mount point:**

```bash
$ sudo mkdir -p /mnt/parted /mnt/fdisk
```

**Mount dengan label:**

```bash
$ sudo mount LABEL=PARTED_DISK /mnt/parted
$ sudo mount LABEL=FDISK_DISK /mnt/fdisk
```

**Cek hasil mount:**

```bash
$ df -h | grep sdc
```

Output:

```
/dev/sdc1       223M   24K  205M   1% /mnt/parted
/dev/sdc2       224M   24K  206M   1% /mnt/fdisk
```

### 9. Konfigurasi persistent mount di `/etc/fstab`

```bash
$ sudo vim /etc/fstab
```

Tambahkan **dua baris** di akhir file:

```
LABEL=PARTED_DISK  /mnt/parted  ext4  defaults,nofail  0  2
LABEL=FDISK_DISK   /mnt/fdisk   ext4  defaults,nofail  0  2
```

**Penjelasan kolom:**

| Kolom | Nilai | Arti |
|---|---|---|
| 1 | `LABEL=PARTED_DISK` | Identifikasi device (bisa pakai `LABEL=`, `UUID=`, atau `/dev/sdc1`) |
| 2 | `/mnt/parted` | Mount point |
| 3 | `ext4` | Filesystem type |
| 4 | `defaults,nofail` | Opsi mount |
| 5 | `0` | Tidak di-dump oleh `dump` |
| 6 | `2` | Urutan fsck (untuk non-root, gunakan 2) |

**Kenapa pakai `LABEL=` dan `nofail`?**
- **`LABEL=`** → device tetap dikenali meskipun nama device berubah (`sdc1` bisa jadi `sdd1` setelah reboot).
- **`nofail`** → sistem tetap boot meskipun partisi tidak ada. **Ini krusial** — kalau tidak ada `nofail` dan partisi hilang, sistem bisa masuk emergency mode.

### 10. Test konfigurasi `/etc/fstab`

**Unmount dulu:**

```bash
$ sudo umount /mnt/parted /mnt/fdisk
```

**Mount semua dari fstab:**

```bash
$ sudo mount -a
```

**Verifikasi:**

```bash
$ df -h | grep sdc
```

Jika kedua partisi muncul kembali, berarti konfigurasi `/etc/fstab` sudah benar.

---

## 🧪 Practice Task

### Soal

1. Buat partisi 128 MB di `/dev/sdb` menggunakan `parted` dengan label GPT.
2. Buat partisi kedua 256 MB di `/dev/sdb` menggunakan `fdisk`.
3. Format partisi pertama dengan ext4 + label `SB1`, dan partisi kedua dengan ext4 + label `SB2`.
4. Buat mount point `/mnt/sb1` dan `/mnt/sb2`.
5. Mount kedua partisi menggunakan label.
6. Konfigurasi `/etc/fstab` agar keduanya ter-mount otomatis saat boot, dengan opsi `defaults,nofail`.
7. Test dengan `mount -a` dan verifikasi hasilnya.

### ✅ Solusi

```bash
# 1. Partisi pertama dengan parted
$ sudo parted /dev/sdb
(parted) mklabel gpt
(parted) mkpart
Partition name?  []? primary
File system type?  [ext2]? ext4
Start? 1MiB
End? 129MiB
(parted) quit

# 2. Partisi kedua dengan fdisk
$ sudo fdisk /dev/sdb
Command (m for help): n
Partition number (2-128): 2
First sector: <Enter>
Last sector: +256M
Command (m for help): w

# 3. Format & label
$ sudo mkfs -t ext4 -L SB1 /dev/sdb1
$ sudo mkfs -t ext4 -L SB2 /dev/sdb2

# 4. Buat mount point
$ sudo mkdir -p /mnt/sb1 /mnt/sb2

# 5. Mount dengan label
$ sudo mount LABEL=SB1 /mnt/sb1
$ sudo mount LABEL=SB2 /mnt/sb2

# 6. Tambah ke /etc/fstab
$ sudo tee -a /etc/fstab << 'EOF'
LABEL=SB1  /mnt/sb1  ext4  defaults,nofail  0  2
LABEL=SB2  /mnt/sb2  ext4  defaults,nofail  0  2
EOF

# 7. Test
$ sudo umount /mnt/sb1 /mnt/sb2
$ sudo mount -a
$ df -h | grep sdb
```

### 🔍 Hasil Akhir yang Diharapkan

```
$ lsblk /dev/sdb
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb      8:16   0    1G  0 disk
├─sdb1   8:17   0  128M  0 part /mnt/sb1
└─sdb2   8:18   0  256M  0 part /mnt/sb2
```

---

## 🐛 Troubleshooting & Kesalahan

- **`mklabel gpt` menghapus data lama:** Perintah ini **menghapus seluruh partition table** di disk. Kalau disk sudah punya data, semua partisi akan hilang. Selalu pastikan disk yang di-target memang kosong. Cek dulu dengan `lsblk` atau `sudo parted /dev/sdc print`.

- **`parted` vs `fdisk` — mana yang dipakai?** Keduanya bisa membuat partisi, tapi:
  - `parted` → mendukung **GPT** (untuk disk > 2 TB), antarmuka baris per baris.
  - `fdisk` → mendukung MBR dan GPT, antarmuka menu interaktif.
  - **Tips:** Pakai `parted` untuk disk besar/modern, `fdisk` untuk disk kecil/MBR.

- **Partisi mulai dari 0 MiB, bukan 1 MiB:** Kalau start di `0`, partisi tidak ter-*align* dengan baik, sehingga performa bisa turun. Biasakan mulai dari **1 MiB** (atau 2048 sectors) untuk alignment optimal.

- **`mkfs` gagal "device is mounted":** Kalau partisi sudah ter-mount, `mkfs` menolak memformat. Unmount dulu:
  ```bash
  $ sudo umount /dev/sdc1
  ```

- **`mount -a` gagal setelah edit `/etc/fstab`:** Kalau ada typo di `/etc/fstab`, `mount -a` akan gagal dan menampilkan pesan error. **JANGAN reboot** sebelum memperbaiki — kalau reboot dengan fstab rusak, sistem bisa masuk emergency mode. Selalu **test `mount -a` dulu** setiap kali edit fstab.

- **Partisi tidak muncul setelah `fdisk w`:** Kernel kadang tidak langsung membaca perubahan partition table. Paksa kernel reload dengan:
  ```bash
  $ sudo partprobe /dev/sdc
  ```
  Atau reboot.

- **`LABEL=` tidak ditemukan:** Kalau `mount LABEL=PARTED_DISK` gagal dengan *"can't find LABEL"*, cek dulu label dengan `blkid`. Kemungkinan typo atau label belum di-set.

- **`nofail` vs tanpa `nofail`:** Tanpa `nofail`, kalau device hilang saat boot, sistem akan **gagal boot** dan masuk emergency mode. Dengan `nofail`, sistem tetap boot normal meskipun partisi tidak ada. **Selalu pakai `nofail`** untuk partisi non-esensial.

- **Opsi `defaults`:** Ini adalah gabungan dari `rw,suid,dev,exec,auto,nouser,async`. Untuk sebagian besar kasus, `defaults` sudah cukup.

- **Urutan fsck (kolom 6):**
  - `0` → tidak di-fsck (untuk swap, tmpfs).
  - `1` → prioritas tinggi (biasanya untuk `/`).
  - `2` → prioritas rendah (untuk partisi lain).

- **Nama device berubah setelah reboot:** Ini masalah klasik. Device `/dev/sdc1` bisa berubah menjadi `/dev/sdd1` tergantung urutan deteksi. Inilah kenapa **pakai LABEL atau UUID** di `/etc/fstab` sangat dianjurkan.

- **Lupa `sudo`:** Hampir semua perintah di lab ini butuh root: `parted`, `fdisk`, `mkfs`, `mount`, edit `fstab`. Kalau lupa `sudo`, akan muncul *"Permission denied"*.

- **Data hilang setelah `mkfs`:** `mkfs` **menghapus semua data** di partisi target. Pastikan tidak ada data penting sebelum format.

---

## 💡 Catatan & Insight Pribadi

- **MBR vs GPT:**

  | Aspek | MBR | GPT |
  |---|---|---|
  | Maksimum partisi | 4 primary | 128 |
  | Maksimum ukuran disk | 2 TB | 9.4 ZB |
  | Kompatibilitas | BIOS lama | UEFI modern |
  | Backup table | ❌ | ✅ (di akhir disk) |
  | Rekomendasi | Lama | **Modern** |

- **`parted` — perintah berguna di dalam prompt:**

  | Perintah | Fungsi |
  |---|---|
  | `print` | Tampilkan partition table |
  | `mklabel gpt` / `msdos` | Buat table GPT / MBR |
  | `mkpart` | Buat partisi |
  | `rm <n>` | Hapus partisi nomor n |
  | `resizepart` | Resize partisi |
  | `quit` | Keluar |

- **`fdisk` — perintah berguna di dalam prompt:**

  | Perintah | Fungsi |
  |---|---|
  | `p` | Print partition table |
  | `n` | New partition |
  | `d` | Delete partition |
  | `t` | Change type |
  | `w` | Write & exit |
  | `q` | Quit tanpa save |

- **Label vs UUID:** Keduanya sama-sama persistent. UUID lebih unik (tidak mungkin bentrok), label lebih mudah dibaca. Untuk sistem dengan banyak disk, UUID lebih aman; untuk sistem kecil, label lebih praktis.

- **Cek UUID/LABEL dengan:**
  ```bash
  $ sudo blkid /dev/sdc1
  $ lsblk -f /dev/sdc
  ```

- **Mount sementara vs permanent:**
  - `mount` → hanya berlaku sampai reboot.
  - `/etc/fstab` → berlaku setiap boot.

- **Hapus partisi dengan `fdisk`:**
  ```
  Command (m for help): d
  Partition number: 2
  Command (m for help): w
  ```

- **`lsblk -f`:** Menampilkan filesystem + label + UUID + mount point — sangat berguna untuk overview cepat:
  ```bash
  $ lsblk -f
  ```

- **`df -h` vs `lsblk`:** `df` menampilkan yang **sudah di-mount**, `lsblk` menampilkan **semua block device** termasuk yang belum di-mount.

- **Kasus nyata:** Di server produksi, disk tambahan (untuk database, backup, atau log) hampir selalu:
  1. Dipartisi dengan GPT.
  2. Di-format dengan ext4 atau xfs.
  3. Diberi label yang deskriptif (`DB_DATA`, `BACKUP`, `LOG`).
  4. Di-mount dengan `LABEL=` di `/etc/fstab`.
  5. Ditambahkan `nofail` untuk partisi yang tidak esensial.

  Ini memastikan sistem tetap boot meskipun ada masalah disk, dan identifikasi device tetap konsisten meskipun urutan disk berubah.

- **Backup partition table:** Sebelum melakukan perubahan besar, backup dulu:
  ```bash
  $ sudo sfdisk -d /dev/sdc > /root/sdc-partition-backup.txt
  ```
  Kalau ada masalah, bisa di-restore:
  ```bash
  $ sudo sfdisk /dev/sdc < /root/sdc-partition-backup.txt
  ```

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `lsblk` | Tampilkan semua block device & partisi |
| `lsblk -f` | Tampilkan dengan filesystem, label, UUID |
| `sudo parted /dev/sdc` | Buka `parted` untuk disk tertentu |
| `(parted) mklabel gpt` | Buat partition table GPT |
| `(parted) mkpart` | Buat partisi baru |
| `sudo fdisk /dev/sdc` | Buka `fdisk` |
| `(fdisk) n` | Buat partisi baru |
| `(fdisk) w` | Simpan & keluar |
| `mkfs -t ext4 -L LABEL /dev/sdc1` | Format partisi + label |
| `blkid /dev/sdc1` | Lihat label & UUID |
| `mkdir -p /mnt/parted` | Buat mount point |
| `mount LABEL=PARTED_DISK /mnt/parted` | Mount dengan label |
| `umount /mnt/parted` | Unmount |
| `mount -a` | Mount semua dari fstab |
| `partprobe /dev/sdc` | Reload partition table |
| `sfdisk -d /dev/sdc` | Backup partition table |

---

## 📌 Kesimpulan

Lab ini memberikan keterampilan praktis yang sangat penting untuk sysadmin: **mengelola disk dan partisi**. Mulai dari membuat partisi dengan `parted` dan `fdisk`, memformat dengan ext4, memberi label, hingga mengonfigurasi persistent mount di `/etc/fstab`. Yang paling berharga dari lab ini adalah pemahaman tentang **label vs device name** dan **opsi `nofail`** — dua hal kecil yang bisa menyelamatkan sistem dari gagal boot.

Konsep ini akan terus dipakai di topik lanjutan seperti **LVM, RAID, dan manajemen storage di server produksi**.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
