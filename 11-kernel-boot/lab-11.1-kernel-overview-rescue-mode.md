# Lab 10.1 — Kernel Overview, Boot Parameters & Rescue Mode

**Course:** Linux System Administration (Adinusa)
**Topic:** Kernel Linux, parameter boot, dan rescue mode
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Memahami peran **kernel Linux** dalam sebuah sistem operasi
- Mengenali **parameter boot kernel** dan cara membacanya
- Memahami fungsi **Rescue Mode** di Ubuntu dan kapan menggunakannya
- Menggunakan **Rescue Shell** untuk perbaikan sistem yang rusak

---

## 📘 Bagian 1 — Kernel Overview

### Apa itu Linux, sebenarnya?

Secara teknis, **Linux** merujuk pada **kernel** saja — inti dari sistem operasi. Kernel bukanlah sistem operasi yang lengkap. Dia menyediakan fondasi yang menghubungkan hardware dan software.

Sistem operasi lengkap berbasis Linux (Ubuntu, Fedora, Debian) terdiri dari:

| Komponen | Contoh |
|---|---|
| **Kernel** | Linux kernel |
| **Libraries** | GNU C Library (glibc) |
| **Utilities & aplikasi** | shell (bash), compiler (gcc), package manager (apt) |
| **System services** | daemon untuk logging, networking, printing |

### Peran Kernel

Kernel adalah komponen pusat yang mengelola resource sistem dan memastikan aplikasi berjalan lancar dan aman. Tanggung jawab utamanya:

| Tanggung Jawab | Penjelasan |
|---|---|
| **System Initialization & Boot** | Memuat driver hardware, menyiapkan sistem untuk memulai proses user-level |
| **Process Scheduling** | Menentukan proses mana yang dapat CPU time, kapan, dan berapa lama |
| **Memory Management** | Alokasi/dealokasi memori untuk proses, mengelola virtual memory & paging |
| **Hardware Access Control** | Lewat device driver, menyediakan interface konsisten ke hardware |
| **I/O Management** | Transfer data antara aplikasi dan storage/peripheral, termasuk buffering, caching, queuing |
| **Filesystem Implementation** | Mendukung berbagai filesystem (ext4, xfs, btrfs, nfs, dll) |
| **Security Control** | Enforce permission, autentikasi, isolasi proses, firewall (Netfilter/iptables) |
| **Networking Control** | Mengelola stack TCP/IP, socket interface, routing, filtering |

### Kernel dalam Konteks

- Sistem yang menjalankan **hanya kernel** punya fungsionalitas sangat terbatas — biasanya untuk perangkat embedded (router, IoT).
- Sistem general-purpose menjalankan kernel **bersama** library dan user-space tools → membentuk distribusi Linux lengkap.

---

## 📘 Bagian 2 — Kernel Boot Parameters

### Apa itu Kernel Boot Parameters?

Parameter yang dilewatkan ke kernel saat boot. Mempengaruhi bagaimana kernel:
- Menginisialisasi hardware
- Mengelola memori
- Mengonfigurasi device
- Mengontrol perilaku sebelum user-space dimulai

### Di Mana Parameter Didefinisikan?

Di **GRUB** (bootloader umum Linux), parameter diletakkan di baris `linux` atau `linux16` pada file konfigurasi:

- `/boot/grub/grub.cfg` — Debian/Ubuntu
- `/boot/grub2/grub.cfg` — RHEL/CentOS
- `/boot/efi/EFI/<distro>/grub.cfg` — sistem UEFI

### Contoh Kernel Command Line

**Debian-based:**

```
linux /boot/vmlinuz-4.19.0 root=UUID=7ef4e747-... ro quiet crashkernel=384M-:128M
```

**CentOS/RHEL (UEFI):**

```
linuxefi /boot/vmlinuz-5.2.9 root=UUID=77461ee7-... ro crashkernel=auto rhgb quiet
```

### Breakdown Parameter

| Parameter | Arti |
|---|---|
| `/boot/vmlinuz-<versi>` | Kernel image yang di-boot |
| `root=UUID=<uuid>` | Filesystem root yang harus di-mount |
| `ro` | Mount root filesystem **read-only** dulu (nanti di-remount rw) |
| `quiet` | Mengurangi output console saat boot |
| `rhgb` | Red Hat Graphical Boot — tampilkan splash screen |
| `crashkernel=...` | Reserve memori untuk crash dump (kexec/kdump) |
| `resume=UUID=<uuid>` | Resume dari hibernasi menggunakan swap partition |

### Bagaimana Kernel Menangani Parameter?

1. Apapun setelah kernel image (`vmlinuz`) dianggap sebagai opsi.
2. Kalau kernel **tidak mengerti** opsi tersebut, opsi itu dilewatkan ke **init (PID 1)** — proses pertama yang dijalankan sistem.
3. Ini memungkinkan parameter kernel & parameter init system hidup bersama di satu baris yang sama.

### Melihat Parameter Boot yang Sedang Digunakan

```bash
$ cat /proc/cmdline
```

Contoh output:

```
BOOT_IMAGE=(hd0,msdos1)/boot/vmlinuz-5.11.0 root=UUID=7a8244d5-... ro resume=UUID=d602c4e1-... rhgb quiet
```

### Syntax Parameter

- **Standalone flag**: `quiet`, `noapic`
- **Key-value pair**: `root=/dev/sda6`, `crashkernel=256M`
- Dipisahkan dengan **spasi**

### Dokumentasi Kernel Parameters

- **Kernel source**: `Documentation/admin-guide/kernel-parameters.txt`
- **Package**: `kernel-doc` atau `linux-doc`
- **Manual page**: `man bootparam`

### Parameter Umum Lainnya

| Parameter | Fungsi |
|---|---|
| `noapic` | Menonaktifkan APIC (untuk troubleshooting hardware) |
| `vconsole.keymap=us` | Set layout keyboard console |
| `vconsole.font=latarcyrheb-sun16` | Set font console |
| `LANG=en_US.UTF-8` | Set bahasa sistem |

---

## 📘 Bagian 3 — Rescue Mode Ubuntu

### Apa itu Rescue Mode?

**Rescue Mode** (atau Recovery Mode) adalah opsi boot khusus di Ubuntu untuk memperbaiki, melakukan troubleshooting, atau memulihkan sistem yang rusak. Sistem dimulai di environment minimal dengan layanan terbatas dan akses root.

### Kapan Rescue Mode Berguna?

- Sistem gagal boot normal
- Perlu memperbaiki bootloader (GRUB)
- Filesystem korup perlu diperbaiki
- Lupa password root, butuh emergency access
- Troubleshooting driver atau konfigurasi bermasalah

### Cara Masuk Rescue Mode

1. **Reboot** sistem
2. Saat menu **GRUB** muncul, pilih **Advanced options for Ubuntu**
3. Dari submenu, pilih kernel dengan label **(recovery mode)**

   Contoh:
   ```
   Ubuntu, with Linux 5.15.0-76-generic (recovery mode)
   ```

4. Akan muncul **Recovery Menu**

### Opsi di Recovery Menu

| Opsi | Fungsi |
|---|---|
| `resume` | Lanjutkan boot normal |
| `clean` | Bersihkan disk space kalau filesystem penuh |
| `dpkg` | Perbaiki paket yang rusak |
| `fsck` | Cek & perbaiki error filesystem |
| `grub` | Update/reinstall bootloader GRUB |
| `network` | Aktifkan networking |
| `root` | Masuk ke **root shell** untuk perbaikan manual |

### Rescue Shell (Root Access)

Kalau memilih **root**, akan masuk ke shell minimal dengan hak akses root. Dari sini bisa:

- Mount filesystem manual
- Edit file konfigurasi (dengan `nano` atau `vi`)
- Reset password user
- Jalankan perintah perbaikan filesystem (`fsck`)
- Reinstall GRUB kalau bootloader korup

**Contoh perintah di rescue mode:**

```bash
# 1. Remount root filesystem sebagai read-write
# (karena saat masuk rescue mode, root filesystem di-mount read-only)
# mount -o remount,rw /

# 2. Reset password user
# passwd username

# 3. Update konfigurasi GRUB
# update-grub

# 4. Reinstall GRUB ke disk
# grub-install /dev/sda
```

---

## 💡 Catatan & Insight Pribadi

### Kenapa Linux disebut "kernel"?

Istilah **kernel** diambil dari analogi biji (kernel) pada buah — bagian inti yang menjadi pusat. Dalam sistem operasi, kernel adalah inti yang mengelola seluruh resource.

### Monolithic kernel vs microkernel

Linux adalah **monolithic kernel** — semua layanan inti (driver, filesystem, networking) berjalan di kernel space. Ini berbeda dari **microkernel** (seperti Mach) yang hanya menaruh fungsi minimum di kernel space dan sisanya di user space. Monolithic lebih cepat, tapi satu bug di driver bisa crash seluruh sistem.

### Kernel module

Meskipun monolithic, Linux mendukung **loadable kernel modules (LKM)** — driver bisa di-load/unload saat runtime tanpa reboot. Perintah terkait:

```bash
$ lsmod              # daftar module aktif
$ modprobe <module>  # load module
$ modinfo <module>   # info module
```

### Kenapa root filesystem di-mount `ro` saat boot?

Alasannya: untuk **keamanan dan konsistensi**. Saat boot, kernel belum tahu apakah filesystem bersih. Kalau di-mount `rw` dan ternyata ada korupsi, bisa makin parah. Setelah `fsck` selesai dan sistem yakin bersih, baru di-remount `rw`:

```bash
$ mount -o remount,rw /
```

### Membaca `/proc/cmdline`

`/proc/cmdline` adalah cara paling cepat untuk melihat parameter boot yang **sedang aktif** (bukan yang tertulis di GRUB). Ini berguna saat debugging — kadang GRUB di-update tapi parameter lama masih berlaku.

### Kapan pakai `man bootparam`?

Kalau butuh parameter kernel spesifik untuk troubleshooting (misal `noapic`, `nomodeset`, `acpi=off`), `man bootparam` adalah referensi pertama yang harus dibuka.

### Rescue mode vs Live USB

| Aspek | Rescue Mode | Live USB |
|---|---|---|
| Kecepatan akses | Cepat (dari GRUB) | Perlu boot dari USB |
| Environment | Minimal, dari sistem yang ada | Lengkap, independen |
| Cocok untuk | Perbaikan ringan-menengah | Perbaikan berat / sistem benar-benar tidak bisa boot |
| Akses root | Ya | Ya |

Rescue mode berguna kalau GRUB masih bisa diakses. Kalau GRUB sendiri rusak, harus pakai Live USB.

### Kasus nyata

- **Lupa password root**: Masuk rescue mode → `root` shell → `passwd username` → reboot.
- **Filesystem korup**: Masuk rescue mode → `fsck` → perbaiki.
- **GRUB rusak setelah update Windows**: Masuk Live USB → mount partisi → `grub-install` → `update-grub`.
- **Sistem hang saat boot karena driver baru**: Edit GRUB → tambahkan `nomodeset` atau `acpi=off` di kernel command line → boot sementara untuk memperbaiki.

---

## 🧠 Perintah & Konsep yang Dikuasai

| Perintah / Konsep | Fungsi |
|---|---|
| `cat /proc/cmdline` | Lihat kernel boot parameters yang aktif |
| `man bootparam` | Dokumentasi parameter kernel |
| `lsmod` | Daftar kernel module aktif |
| `modprobe` | Load/unload kernel module |
| `modinfo` | Info kernel module |
| `mount -o remount,rw /` | Remount root filesystem sebagai read-write |
| `passwd <user>` | Reset password user |
| `update-grub` | Update konfigurasi GRUB |
| `grub-install /dev/sda` | Reinstall GRUB ke disk |
| **Recovery Menu** | Akses via GRUB → Advanced options → recovery mode |

---

## 📌 Kesimpulan

Lab ini memberikan pemahaman konseptual yang penting bagi sysadmin: **Linux hanyalah kernel**, dan sistem operasi lengkap terbentuk dari kernel + library + user-space tools. Memahami peran kernel (scheduling, memory, I/O, security) membantu kita men-debug masalah sistem dengan lebih efektif.

Yang paling praktis dari lab ini adalah **Rescue Mode** — pintu darurat ketika sistem tidak bisa boot normal. Kemampuan masuk ke rescue mode, remount root filesystem sebagai read-write, reset password, dan memperbaiki GRUB adalah keterampilan wajib yang bisa menyelamatkan sistem (dan pekerjaan) di saat kritis.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
