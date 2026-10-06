# Lab 10.1 — LVM: Physical Volume, Volume Group & Logical Volume

**Course:** Linux System Administration (Adinusa)
**Topic:** Dasar LVM (PV, VG, LV) + persistent mount
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membuat partisi yang disiapkan untuk **LVM**
- Mengonfigurasi **Physical Volume (PV)**, **Volume Group (VG)**, dan **Logical Volume (LV)**
- Memformat dan me-mount LV dengan entry **persistent** di `/etc/fstab`
- Memverifikasi konfigurasi dengan file uji dan perintah LVM

---

## 📘 Konsep Dasar LVM

**LVM (Logical Volume Manager)** adalah lapisan abstraksi antara disk fisik dan filesystem. Alih-alih memformat partisi langsung, kita membangun "lapisan" LVM di atasnya.

```
Disk (/dev/sdb) → Partisi (/dev/sdb1) → PV → VG → LV → Filesystem → Mount point
                                              ↑
                                       (bisa gabung banyak disk)
```

| Konsep | Penjelasan |
|---|---|
| **PV (Physical Volume)** | Partisi/disk yang diinisialisasi untuk LVM |
| **VG (Volume Group)** | Kumpulan PV yang digabung menjadi satu "kolam" penyimpanan |
| **LV (Logical Volume)** | "Partisi virtual" yang dibuat dari VG — inilah yang diformat & di-mount |
| **PE (Physical Extent)** | Unit terkecil alokasi di LVM (default 4 MiB) |

**Kenapa pakai LVM?**
- **Fleksibel** — LV bisa di-resize tanpa reboot (untuk ext4/xfs).
- **Snapshot** — bisa membuat snapshot untuk backup konsisten.
- **Gabung banyak disk** — beberapa disk bisa disatukan jadi satu VG.
- **Migrasi data** — pindah PV ke disk baru tanpa downtime (`pvmove`).

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-009-4
```

Pastikan ada dua disk virtual: `/dev/sdb` (untuk lab ini). Kalau belum, tambahkan lewat GUI VM seperti di Lab 9.3.

---

## 📘 Guided Example

### 1. Switch ke root user

```bash
$ sudo -i
```

Semua perintah berikut dijalankan sebagai root.

### 2. Buat partisi LVM dengan `parted`

```bash
$ parted -s /dev/sdb mklabel gpt
$ parted -s /dev/sdb mkpart primary 1MiB 257MiB
$ parted -s /dev/sdb set 1 lvm on
$ parted -s /dev/sdb mkpart primary 258MiB 514MiB
$ parted -s /dev/sdb set 2 lvm on
```

**Penjelasan opsi `-s`:** mode script — tidak masuk ke prompt interaktif, langsung eksekusi.

**Penjelasan langkah:**
- `mklabel gpt` → buat partition table GPT baru.
- `mkpart primary 1MiB 257MiB` → partisi 1, ukuran ~256 MiB.
- `set 1 lvm on` → set **flag LVM** pada partisi 1. Ini **wajib** agar bisa dipakai sebagai PV.
- `mkpart primary 258MiB 514MiB` → partisi 2, ukuran ~256 MiB, mulai dari 258 MiB (ada jeda 1 MiB untuk alignment).

### 3. Register partisi baru ke kernel

```bash
$ udevadm settle
```

**Penjelasan:** Menunggu semua event udev selesai, sehingga device `/dev/sdb1` dan `/dev/sdb2` benar-benar siap dipakai. Tanpa ini, `pvcreate` kadang gagal karena device belum terdaftar.

### 4. Buat Physical Volume

```bash
$ pvcreate /dev/sdb1 /dev/sdb2
```

Output:

```
Physical volume "/dev/sdb1" successfully created.
Physical volume "/dev/sdb2" successfully created.
```

### 5. Buat Volume Group

```bash
$ vgcreate username_01_vg /dev/sdb1 /dev/sdb2
```

Output:

```
Volume group "username_01_vg" successfully created
```

**Ganti `username`** dengan username Adinusa-mu (contoh: `arijfarid69_01_vg`).

VG ini sekarang berisi **dua PV** — total kapasitas ~512 MiB.

### 6. Buat Logical Volume

```bash
$ lvcreate -n username_01_lv -L 400M username_01_vg
```

Output:

```
Logical volume "username_01_lv" created.
```

**Penjelasan opsi:**

| Opsi | Arti |
|---|---|
| `-n username_01_lv` | Nama LV |
| `-L 400M` | Ukuran 400 MiB |
| `username_01_vg` | VG sumber |

Hasilnya: device `/dev/username_01_vg/username_01_lv` — **belum ada filesystem**-nya.

### 7. Format LV dengan ext4

```bash
$ mkfs -t ext4 /dev/username_01_vg/username_01_lv
```

### 8. Buat mount point

```bash
$ mkdir /data
```

### 9. Tambahkan entry ke `/etc/fstab`

Tambahkan baris berikut di akhir file:

```
/dev/username_01_vg/username_01_lv   /data   ext4   defaults   1 2
```

Bisa lewat `vim` atau `tee`:

```bash
$ echo '/dev/username_01_vg/username_01_lv   /data   ext4   defaults   1 2' >> /etc/fstab
```

**Penjelasan kolom:**
| Kolom | Nilai | Arti |
|---|---|---|
| 1 | `/dev/username_01_vg/username_01_lv` | Device LV |
| 2 | `/data` | Mount point |
| 3 | `ext4` | Filesystem type |
| 4 | `defaults` | Opsi mount |
| 5 | `1` | Di-dump oleh `dump` |
| 6 | `2` | Urutan fsck |

### 10. Reload systemd

```bash
$ systemctl daemon-reload
```

**Penjelasan:** Memberi tahu systemd bahwa `/etc/fstab` berubah, sehingga unit `.mount` di-generate ulang.

### 11. Mount dengan `mount /data`

```bash
$ mount /data
```

Karena `/data` sudah ada di `/etc/fstab`, `mount /data` akan mencari device yang sesuai dan me-mount-nya tanpa perlu menentukan device.

Verifikasi:

```bash
$ df -h | grep data
```

### 12. Test dengan file

```bash
$ cp -a /etc/*.conf /data
$ ls /data | wc -l
```

Contoh output: `34` (jumlah bervariasi).

---

## 🧪 Practice Task

### Soal

1. Buat PV dari `/dev/sdb1` dan `/dev/sdb2` (asumsi sudah dipartisi LVM).
2. Buat VG bernama `<username>_02_vg`.
3. Buat LV bernama `<username>_02_lv` berukuran 300 MiB.
4. Format dengan ext4.
5. Mount ke `/data2` dengan entry di `/etc/fstab`.
6. Salin 10 file dari `/etc` ke `/data2`.
7. Verifikasi semua konfigurasi.

### ✅ Solusi

```bash
# 1. Buat PV (kalau belum ada)
$ pvcreate /dev/sdb1 /dev/sdb2

# 2. Buat VG
$ vgcreate arijfarid69_02_vg /dev/sdb1 /dev/sdb2

# 3. Buat LV
$ lvcreate -n arijfarid69_02_lv -L 300M arijfarid69_02_vg

# 4. Format
$ mkfs -t ext4 /dev/arijfarid69_02_vg/arijfarid69_02_lv

# 5. Mount point + fstab
$ mkdir /data2
$ echo '/dev/arijfarid69_02_vg/arijfarid69_02_lv   /data2   ext4   defaults   1 2' >> /etc/fstab
$ systemctl daemon-reload
$ mount /data2

# 6. Test
$ cp /etc/*.conf /data2 2>/dev/null
$ ls /data2 | head -10

# 7. Verifikasi
$ pvdisplay
$ vgdisplay
$ lvdisplay
$ df -h | grep data2
```

---

## 🔍 Verification Commands

```bash
# Cek partisi
$ parted /dev/sdb print

# Cek physical volume
$ pvdisplay /dev/sdb2
$ pvs

# Cek volume group
$ vgdisplay username_01_vg
$ vgs

# Cek logical volume
$ lvdisplay /dev/username_01_vg/username_01_lv
$ lvs

# Cek mount
$ df -h | grep data
$ mount | grep data

# Cek fstab
$ grep data /etc/fstab
```

---

## 🐛 Troubleshooting & Kesalahan

- **`pvcreate` gagal "Device is mounted":** Kalau partisi sudah ter-mount atau terpakai filesystem lain, `pvcreate` akan menolak. Pastikan partisi **kosong** (belum diformat atau sudah di-wipe). Cek dengan `lsblk -f`. Kalau ada filesystem, hapus dulu:
  ```bash
  $ wipefs -a /dev/sdb1
  ```

- **`pvcreate` gagal "No such file or directory":** Device `/dev/sdb1` belum terdaftar di kernel. Solusinya: jalankan `partprobe /dev/sdb` atau `udevadm settle`, atau reboot.

- **Lupa set flag `lvm on` di parted:** Partisi tetap bisa dipakai sebagai PV, tapi beberapa tools (terutama LVM-aware) akan memperingatkan. Biasakan selalu `set N lvm on` setelah `mkpart`.

- **`mkfs` tidak bisa jalan di LV baru:** Kadang LV baru belum "terlihat" oleh kernel. Jalankan:
  ```bash
  $ udevadm settle
  $ partprobe
  ```
  lalu coba lagi.

- **`mount /data` gagal "special device does not exist":** Biasanya karena nama device di `/etc/fstab` salah. Cek dengan:
  ```bash
  $ ls -l /dev/username_01_vg/username_01_lv
  $ lvs
  ```
  Pastikan nama VG dan LV persis sama seperti di fstab.

- **Lupa `systemctl daemon-reload`:** Setelah edit `/etc/fstab`, systemd masih memegang versi lama. Tanpa reload, `mount /data` bisa gagal atau muncul warning. Selalu jalankan `systemctl daemon-reload` setelah edit fstab.

- **`/etc/fstab` typo → sistem tidak boot:** Ini masalah serius. Kalau ada typo di fstab, reboot bisa berujung ke emergency mode. **Selalu test dengan `mount -a`** sebelum reboot:
  ```bash
  $ umount /data
  $ mount -a
  $ df -h | grep data
  ```
  Kalau `mount -a` sukses, berarti fstab valid.

- **Ukuran LV lebih besar dari kapasitas VG:** Kalau `lvcreate -L 600M` di VG yang hanya punya 512 MiB, akan muncul error *"Volume group ... has insufficient free space"*. Cek dulu kapasitas VG dengan `vgdisplay` atau `vgs`.

- **LV lebih kecil dari filesystem:** Kalau LV di-resize ke bawah tanpa resize filesystem dulu, data bisa korup. Untuk resize aman:
  - Perbesar: `lvextend` dulu → `resize2fs`.
  - Perkecil: `resize2fs` dulu → `lvreduce`.

- **Nama VG/LV mengandung spasi atau karakter aneh:** LVM tidak mengizinkan spasi. Gunakan underscore atau dash. Nama case-sensitive.

- **Ganti nama VG/LV setelah fstab sudah diisi:** Kalau nama VG/LV berubah, update juga `/etc/fstab`. Kalau tidak, mount akan gagal.

- **`parted -s` gagal "unrecognised disk label":** Partisi pertama kali di disk yang masih MBR. `mklabel gpt` akan mengatasi ini, tapi **data lama akan hilang**.

- **Disk di passthrough VM:** Di VM, kadang disk tambahan perlu di-rescan agar muncul:
  ```bash
  $ echo "- - -" > /sys/class/scsi_host/host0/scan
  ```
  atau reboot VM.

---

## 💡 Catatan & Insight Pribadi

- **Kenapa LVM lebih baik dari partisi biasa?**
  - **Resize online** tanpa reboot (ext4/xfs).
  - **Snapshot** untuk backup.
  - **Gabung disk** tanpa harus RAID.
  - **Migrasi** data antar disk tanpa downtime (`pvmove`).
  - **Thin provisioning** untuk efisiensi storage.

- **Alur LVM yang harus dihafal:**
  ```
  partisi → pvcreate → vgcreate → lvcreate → mkfs → mount → fstab
  ```

- **Perintah cek cepat (shortcut):**

  | Full | Shortcut | Isi |
  |---|---|---|
  | `pvdisplay` | `pvs` | Physical Volumes |
  | `vgdisplay` | `vgs` | Volume Groups |
  | `lvdisplay` | `lvs` | Logical Volumes |
  | `pvscan` | — | Scan PV |
  | `vgscan` | — | Scan VG |
  | `lvscan` | — | Scan LV |

  Shortcut (`pvs`, `vgs`, `lvs`) lebih ringkas untuk overview cepat.

- **`lsblk` dengan LVM:** Menampilkan struktur LVM dengan jelas:
  ```bash
  $ lsblk
  sdb
  ├─sdb1  lvm  └─username_01_vg-username_01_lv
  └─sdb2  lvm  └─username_01_vg-username_01_lv
  ```
  LV muncul sebagai child dari VG yang menaungi PV.

- **Path LV ada dua:**
  - `/dev/username_01_vg/username_01_lv` (format direktori)
  - `/dev/mapper/username_01_vg-username_01_lv` (format mapper)
  Keduanya menunjuk device yang sama.

- **`vgextend` — tambah PV ke VG yang sudah ada:**
  ```bash
  $ vgextend username_01_vg /dev/sdc1
  ```

- **`lvextend` — perbesar LV:**
  ```bash
  $ lvextend -L +200M /dev/username_01_vg/username_01_lv
  $ resize2fs /dev/username_01_vg/username_01_lv
  ```
  Untuk ext4, `resize2fs` bisa dilakukan **online** (saat ter-mount).

- **`lvreduce` — perkecil LV:**
  ```bash
  $ umount /data
  $ e2fsck -f /dev/username_01_vg/username_01_lv
  $ resize2fs /dev/username_01_vg/username_01_lv 200M
  $ lvreduce -L 200M /dev/username_01_vg/username_01_lv
  ```
  **Urutan penting**: filesystem dulu, baru LV. Kalau dibalik, data bisa korup.

- **Snapshot LVM:**
  ```bash
  $ lvcreate -s -n snap_lv -L 100M /dev/username_01_vg/username_01_lv
  ```
  Snapshot ini bisa di-mount untuk backup konsisten.

- **Hapus LVM dengan urutan benar:**
  ```bash
  $ umount /data
  $ lvremove /dev/username_01_vg/username_01_lv
  $ vgremove username_01_vg
  $ pvremove /dev/sdb1 /dev/sdb2
  ```

- **`nofail` untuk LVM di fstab:** Seperti di Lab 9.3, pertimbangkan menambahkan `nofail` di opsi mount. Kalau VG tidak aktif (misal karena disk rusak), sistem tetap bisa boot:
  ```
  /dev/username_01_vg/username_01_lv  /data  ext4  defaults,nofail  1  2
  ```

- **Alternatif: gunakan `UUID=` di fstab:** Lebih aman kalau nama VG/LV berubah:
  ```bash
  $ blkid /dev/username_01_vg/username_01_lv
  ```

- **Kasus nyata:** Di server produksi, LVM hampir selalu dipakai untuk:
  - Partisi root `/` (agar mudah di-resize).
  - Volume data (`/var`, `/home`, `/data`) yang butuh fleksibilitas.
  - Volume database yang butuh snapshot untuk backup.
  - Migrasi storage antar server (dengan `pvmove`).

- **LVM vs ZFS/Btrfs:** LVM adalah **volume manager**, bukan filesystem. Untuk fitur seperti checksum, compression, dan snapshot native, ZFS/Btrfs lebih unggul. Tapi LVM lebih matang, lebih ringan, dan didukung di mana-mana.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `parted -s /dev/sdb mkpart` | Buat partisi mode script |
| `parted -s /dev/sdb set N lvm on` | Set flag LVM |
| `udevadm settle` | Tunggu event udev selesai |
| `pvcreate /dev/sdb1` | Buat Physical Volume |
| `vgcreate <vg> /dev/sdb1` | Buat Volume Group |
| `lvcreate -n <lv> -L 400M <vg>` | Buat Logical Volume |
| `mkfs -t ext4 /dev/<vg>/<lv>` | Format LV |
| `mount /data` | Mount via fstab |
| `systemctl daemon-reload` | Reload konfigurasi systemd |
| `pvdisplay` / `pvs` | Info Physical Volume |
| `vgdisplay` / `vgs` | Info Volume Group |
| `lvdisplay` / `lvs` | Info Logical Volume |
| `vgextend` | Tambah PV ke VG |
| `lvextend` | Perbesar LV |
| `lvreduce` | Perkecil LV |
| `lvremove` / `vgremove` / `pvremove` | Hapus LVM dengan urutan benar |

---

## 📌 Kesimpulan

Lab ini memperkenalkan **LVM** — salah satu fitur paling powerful di Linux untuk manajemen storage. Dengan LVM, kita bisa membangun "storage virtual" yang jauh lebih fleksibel dari partisi biasa: bisa di-resize online, digabung dari banyak disk, dan di-snapshot untuk backup. Alur **partisi → PV → VG → LV → filesystem → mount** adalah fondasi yang wajib dihafal sysadmin, karena hampir semua server produksi modern menggunakan pola ini.

Selain itu, lab ini memperkuat kebiasaan baik yang sudah dipelajari di Lab 9.3: **selalu test `/etc/fstab` dengan `mount -a` sebelum reboot**, dan **selalu jalankan `systemctl daemon-reload` setelah edit fstab**.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
