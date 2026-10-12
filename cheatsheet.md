# Linux SysAdmin Cheatsheet

**Catatan belajar Linux System Administration**
Dari kursus Adinusa (Lab 3.x – 19.x) + praktik pribadi.

> **Cara pakai:** Bagian 1–2 = fondasi. Bagian 3–4 = sistem & storage. Bagian 5–6 = security & boot. Bagian 7 = referensi cepat & troubleshooting. Bagian 8 = networking & web server. Bagian 9 = challenges.

> **Penanda:** `[OK]` aman · `[!]` hati-hati · `[X]` bahaya

> **Encoding file:** UTF-8

---

## Progress Belajar

| Area | Lab | Status |
|---|---|---|
| Fondasi (file, editor, pipe) | 3.1 – 3.4 | Selesai |
| Resource & Process | 4.1 – 4.2, 5.1, 7.1 | Selesai |
| Package & Storage | 6.1 – 6.2, 9.1 – 9.3, 10.1 | Selesai |
| Filesystem & Links | 8.1 – 8.3 | Selesai |
| User & Security | 11.1, 12.0 – 12.5 | Selesai |
| Permissions | 13.0 – 13.6 | Selesai |
| Networking | 14.0 – 15.2 | Selesai |
| Web Security | 16.0 – 16.3 | Selesai |
| System Boot & systemd | 17.0, 18.0 – 18.1 | Selesai |
| Troubleshooting | 19.0 – 19.1 | Selesai |
| Challenges | Challenge 1 – 5 | Selesai |

**Terakhir update:** 2026-10-12 (Challenge 5 User Accounts)

---

## Daftar Isi

**Fondasi**
- [1.1 Navigasi Direktori](#11-navigasi-direktori)
- [1.2 Manajemen File & Direktori](#12-manajemen-file--direktori)
- [1.3 Melihat Isi File](#13-melihat-isi-file)
- [1.4 Membuat & Menulis File](#14-membuat--menulis-file)
- [1.5 Nano](#15-nano--editor-ramah-pemula)
- [1.6 Vim](#16-vim--editor-powerful)
- [1.7 Find — Mencari File](#17-find--mencari-file)

**Data & Teks**
- [2.1 Pipe, Filter & Hitung](#21-pipe-filter--hitung)
- [2.2 Opsi Penting](#22-opsi-penting-yang-wajib-dihafal)
- [2.3 Kombinasi Pola Umum](#23-kombinasi-pola-yang-sering-dipakai)
- [2.4 Grep — Text Processing](#24-grep--text-processing)
- [2.5 Diff — Membandingkan File](#25-diff--membandingkan-file)

**Sistem & Proses**
- [3.1 Resource Limits](#31-resource-limits-ulimit)
- [3.2 Process Management](#32-process-management)
- [3.3 Monitoring CPU & Load](#33-monitoring-cpu--load)

**Package & Storage**
- [4.1 APT](#41-package-management-apt)
- [4.2 External Repository](#42-external-repository--version-pinning)
- [4.3 Swap File](#43-swap-file)
- [4.4 df & du](#44-disk-usage-df--du)
- [4.5 Partition](#45-partition-management)
- [4.6 LVM](#46-lvm-logical-volume-manager)
- [4.7 Disk Baru + Auto-Mount](#47-disk-baru--auto-mount)

**User & Security**
- [5.1 Inode & Links](#51-inode--links)
- [5.2 User & Group](#52-user--group-management)
- [5.3 Password Aging](#53-password-aging-chage)
- [5.4 Restricted User](#54-restricted-user--access-control)
- [5.5 SSH](#55-ssh--ssh-key-authentication)
- [5.6 Kernel & Boot](#56-kernel--boot)
- [5.7 System Startup & GRUB (Teori)](#57-system-startup--grub-teori)
- [5.8 systemd Service](#58-systemd-service)
- [5.9 System Init (Teori)](#59-system-init-teori)
- [5.10 User Management — Advanced](#510-user-management--advanced)

**Permissions**
- [6.1 chmod Symbolic](#61-chmod--symbolic-mode)
- [6.2 chmod Octal](#62-chmod--numeric-mode-octal)
- [6.3 ACL](#63-acl-access-control-list)
- [6.4 chown & chgrp](#64-chown--chgrp)
- [6.5 Special Permissions](#65-special-permissions-suid-sgid-sticky)
- [6.6 File Attributes](#66-file-attributes-chattr--lsattr)

**Referensi Cepat & Troubleshooting**
- [7.1 Perbedaan Penting](#71-perbedaan-penting)
- [7.2 Danger Zone](#72-danger-zone--perintah-berbahaya)
- [7.3 Alur Troubleshooting Umum](#73-alur-troubleshooting-umum)
- [7.4 Best Practice](#74-best-practice)
- [7.5 Glosarium](#75-glosarium)
- [7.6 Kalau Panic](#76-kalau-panic)
- [7.7 Troubleshooting Apache](#77-troubleshooting-apache)
- [7.8 Basic Troubleshooting (Teori)](#78-basic-troubleshooting-teori)

**Networking & Web Server**
- [8.0 IP Address](#80-ip-address--konsep-dasar)
- [8.1 Hostname](#81-hostname)
- [8.2 Network Device & `ip` Command](#82-network-device--ip-command)
- [8.3 Name Resolution & Diagnostics](#83-name-resolution--diagnostics)
- [8.4 Static Configuration (Netplan)](#84-static-configuration-netplan)
- [8.5 Apache Virtual Host](#85-apache-virtual-host)
- [8.6 Apache HTTPS (Self-Signed)](#86-apache-https-self-signed)
- [8.7 phpMyAdmin + Firewalld](#87-phpmyadmin--firewalld)
- [8.8 Nginx Custom Port](#88-nginx-custom-port--document-root)
- [8.9 Firewalld & Zone (Teori)](#89-firewalld--zone-teori)

**Panduan Update**
- [9.1 Cara Nambah Section](#91-cara-nambah-section)
- [9.2 Changelog](#92-changelog)

---

# BAGIAN 1 — FONDASI

## 1.1 Navigasi Direktori

**Kapan dipakai:** Setiap saat. Ini GPS-nya Linux.

| Perintah | Fungsi |
|---|---|
| `pwd` | Posisi saat ini |
| `cd folder` | Masuk folder |
| `cd ..` | Naik satu level |
| `cd ~` | Ke home |
| `cd -` | Ke direktori sebelumnya |

```bash
$ pwd
/home/student
$ cd lab5 && pwd
/home/student/lab5
```

---

## 1.2 Manajemen File & Direktori

| Perintah | Fungsi |
|---|---|
| `mkdir nama` | Buat direktori |
| `mkdir -p a/b/c` | Buat + parent sekaligus |
| `touch file` | Buat file kosong |
| `rm file` | Hapus file |
| `rmdir folder` | Hapus folder kosong |
| `rm -r folder` | Hapus folder + isinya `[X]` |
| `rm -ri folder` | Hapus dengan konfirmasi `[OK]` |

> **Tips:** Biasakan `rm -ri` — konfirmasi per file menyelamatkan dari bencana.

---

## 1.3 Melihat Isi File

| Perintah | Fungsi |
|---|---|
| `ls` | List isi |
| `ls -a` | Tampilkan hidden |
| `ls -l` | Detail (permission, owner, size) |
| `ls -lah` | Detail + hidden + human-readable |
| `ls -li` | Detail + inode |
| `cat file` | Seluruh isi |
| `less file` | Per halaman (untuk log) |

> **Tips:** `ls -lah` = perintah paling sering dipakai sysadmin.

---

## 1.4 Membuat & Menulis File

| Perintah | Fungsi |
|---|---|
| `echo "teks" > file` | Timpa isi file |
| `echo "teks" >> file` | Tambah di akhir |
| `cat f1 f2 > f3` | Gabung file |
| `nano file` | Editor ramah pemula |
| `vim file` | Editor powerful |

> [!] `>` menimpa, `>>` menambah. Salah pakai `>` bisa hilangkan data.

---

## 1.5 Nano — Editor Ramah Pemula

| Shortcut | Fungsi |
|---|---|
| `CTRL + O` → Enter | Simpan |
| `CTRL + X` | Keluar |
| `CTRL + W` | Cari |
| `CTRL + K` / `CTRL + U` | Cut / Paste |

---

## 1.6 Vim — Editor Powerful

**Mode:** `Esc` (normal) · `i` (insert) · `:` (command)

| Perintah | Fungsi |
|---|---|
| `:wq` | Save & exit |
| `:q!` | Keluar tanpa save |
| `/kata` | Cari |
| `yy` / `dd` / `p` | Copy / cut / paste baris |
| `G` / `gg` | Ke akhir / awal |
| `o` | Baris baru di bawah |

> **Tips:** Kalau bingung di Vim, tekan `Esc` dulu.

---

## 1.7 Find — Mencari File

**Konsep:** `find` = cari file berdasarkan kriteria. Tool wajib sysadmin.

| Filter | Opsi | Contoh |
|---|---|---|
| Nama | `-name` / `-iname` | `-name "*.txt"` |
| Type | `-type` | `-type f` (file), `d` (dir) |
| Permission | `-perm` | `-perm 0644` |
| Size | `-size` | `-size +8k -size -10k` |
| User | `-user` | `-user student` |
| Group | `-group` | `-group root` |
| Waktu | `-mtime` | `-mtime -7` (7 hari terakhir) |
| Aksi | `-exec` | `-exec cp {} dest/ \;` |

**Permission di `find`:**

| Cara | Arti |
|---|---|
| `-perm 0644` | **Persis** 0644 |
| `-perm -4000` | **Setidaknya** SUID |
| `-perm -6000` | **Setidaknya** SUID + SGID |
| `-perm /u+x` | **Salah satu** bit |

**Size:**

| Format | Arti |
|---|---|
| `-size 8k` | Persis 8 KB |
| `-size +8k` | > 8 KB |
| `-size -10k` | < 10 KB |
| `+8191c -10241c` | Range inklusif via byte |

**Operators:**

| Operator | Arti |
|---|---|
| `-a` | AND (default) |
| `-o` | OR |
| `!` / `-not` | NOT |
| `\( \)` | Grouping |

**Contoh:**

```bash
# File SUID + SGID
find /path -type f -perm -6000

# Mode 0600
find /path -type f -perm 0600

# Size 8K-10K (inklusif)
find /path -type f -size +8191c -size -10241c

# .txt bukan milik student
find /path -name "*.txt" ! -user student

# Copy preserve permission
find /path -perm 0600 -exec cp -p {} /dest/ \;
```

**Tips:**
- **Path dulu, ekspresi kemudian.** `find /path -name "*.txt"` ✅
- `-perm` numeric butuh 4 digit: `0600`, bukan `600`.
- `-exec ... \;` (per file) vs `-exec ... +` (batch, lebih cepat).
- `cp -p` untuk preserve permission + timestamp.
- **`find -size` pakai pembulatan ke atas** — untuk range inklusif, pakai byte (`c`).

---

# BAGIAN 2 — DATA & TEKS

## 2.1 Pipe, Filter & Hitung

**Konsep:** Pipe (`|`) menghubungkan output satu perintah ke input perintah lain.

| Perintah | Fungsi |
|---|---|
| `cmd1 \| cmd2` | Output cmd1 ke cmd2 |
| `cat file \| grep kata` | Filter baris |
| `ls \| grep -c pola` | Hitung yang cocok |
| `ls \| wc -l` | Hitung item |
| `wc -l -w -c` | Hitung baris / kata / byte |

```bash
# Cari error di log
$ cat /var/log/*.log | grep ERROR

# Hitung user dengan shell bash
$ cat /etc/passwd | grep "/bin/bash" | wc -l
```

> **Analogi:** Pipe = pipa air — output keran pertama masuk ke keran berikutnya.

---

## 2.2 Opsi Penting yang Wajib Dihafal

| Opsi | Arti | Dipakai di |
|---|---|---|
| `-a` | All (termasuk hidden) | `ls` |
| `-l` | Long format | `ls` |
| `-h` | Human-readable | `ls`, `df`, `du` |
| `-r` | Recursive | `cp`, `rm` |
| `-i` | Interactive | `rm`, `cp` |
| `-p` | Buat parent | `mkdir` |
| `-c` | Count | `grep` |
| `-v` | Invert match | `grep` |

---

## 2.3 Kombinasi Pola yang Sering Dipakai

```bash
# Hitung file dengan pola
$ ls | grep -c client

# Scroll file panjang
$ cat /etc/passwd | less

# Gabung file
$ cat notes.txt data.txt > combined.txt

# Buang error
$ ls pola* 2>/dev/null

# Top 20 file/direktori terbesar
$ du -ah ~ | sort -h | tail -20
```

---

## 2.4 Grep — Text Processing

**Konsep:** `grep` = cari baris yang cocok dengan pattern. Alat utama log analysis.

| Perintah | Fungsi |
|---|---|
| `grep "pattern" file` | Cari baris mengandung pattern |
| `grep "^pattern" file` | Baris **diawali** pattern |
| `grep "pattern$" file` | Baris **diakhiri** pattern |
| `grep -c "pattern" file` | Hitung jumlah baris cocok |
| `grep -i "pattern" file` | Case-insensitive |
| `grep -v "pattern" file` | Invert — baris yang tidak cocok |
| `grep -n "pattern" file` | Tampilkan nomor baris |
| `grep -E "p1\|p2" file` | Extended regex (OR) |

**Anchor regex:**

| Pattern | Cocok dengan |
|---|---|
| `^abc` | Baris dimulai `abc` |
| `abc$` | Baris diakhiri `abc` |
| `^$` | Baris kosong |

**Multi-filter (AND):**

```bash
# Cara 1: pipe (gampang dibaca)
$ grep "TRACE" file | grep "router_forward"

# Cara 2: regex OR (efisien)
$ grep -E "TRACE.*router_forward|router_forward.*TRACE" file
```

**Simpan hasil:**

```bash
$ grep "ERROR" app.log > error.log          # overwrite
$ grep "INFO" app.log >> info.log           # append
```

**Tips:**
- `^` = anchor awal baris. Tanpa `^`, pattern cocok di posisi mana saja.
- `grep -c` untuk hitung cepat, `wc -l` untuk cek jumlah baris file.
- Pipe lebih mudah dibaca, regex lebih efisien untuk file besar.
- `grep` default BRE (Basic Regex), `grep -E` untuk ERE.

---

## 2.5 Diff — Membandingkan File

**Konsep:** `diff` = bandingkan file baris per baris. Tool wajib untuk cek perubahan.

| Perintah | Fungsi |
|---|---|
| `diff file1 file2` | Bandingkan 2 file |
| `diff -i file1 file2` | Ignore case |
| `diff -w file1 file2` | Ignore whitespace |
| `diff -u file1 file2` | Unified format (patch) |
| `diff -y file1 file2` | Side-by-side |
| `diff -r dir1 dir2` | Recursive |
| `diff -rq dir1 dir2` | Recursive + brief |
| `comm file1 file2` | Bandingkan sorted |
| `cmp file1 file2` | Bandingkan byte-by-byte |

**Format output `diff`:**

| Simbol | Arti |
|---|---|
| `XcY` | Baris X di file1 **diubah** → baris Y di file2 |
| `XaY` | Baris Y **ditambahkan** setelah baris X |
| `XdY` | Baris X **dihapus** |
| `<` | Baris dari file1 |
| `>` | Baris dari file2 |

**Untuk direktori (`-rq`) — 2 jenis output:**

```
Files dir1/file and dir2/file differ      ← isi beda
Only in dir1: extra.txt                   ← file unik
```

**Contoh:**

```bash
# Bandingkan 2 file, ignore case
diff -i fileA.txt fileB.txt > result.txt

# Bandingkan direktori, ambil file yang isinya beda
diff -rq dir1 dir2 | grep "^Files" | awk '{print $2}'

# Cari file yang hanya ada di dir2 (nama saja)
diff -rq dir1 dir2 | grep "Only in.*dir2" | awk '{print $NF}'

# Extract absolute path dari "Only in"
diff -rq dir1 dir2 | grep "Only in" | sed 's/Only in \(.*\): \(.*\)/\1\/\2/'

# Alternatif: pakai comm untuk file sorted
comm -13 <(ls dir1 | sort) <(ls dir2 | sort)
```

**Tips:**
- `-q` untuk brief — cuma nama file, bukan isi.
- `-r` untuk recursive directory tree.
- **Baca output mentah dulu** — bisa `Files differ` atau `Only in`.
- `awk '{print $NF}'` untuk ambil nama file (kolom terakhir).
- `comm -13` = file only di argumen kedua.
- Untuk file binary, pakai `cmp`, bukan `diff`.

---

# BAGIAN 3 — SISTEM & PROSES

## 3.1 Resource Limits (ulimit)

**Konsep:** Linux membatasi resource per user — mencegah satu user menghabiskan resource sistem.

| Perintah | Fungsi |
|---|---|
| `ulimit -a` | Semua limit |
| `ulimit -n` | Open files |
| `ulimit -u` | Proses per user |
| `ulimit -c` | Core file size |

**Permanent di `/etc/security/limits.conf`:**

```
<domain>   <type>   <item>   <value>
@student   hard     nproc    100
student    soft     nofile   2000
student    hard     nofile   3000
```

| Field | Isi |
|---|---|
| domain | user, `@group`, atau `*` |
| type | `soft` (aktif) atau `hard` (batas) |
| item | `nproc`, `nofile`, `core`, dll |

> [!] Perubahan berlaku setelah logout & login ulang.

> **Tips:** Kalau dapat error *"fork: Resource temporarily unavailable"*, cek `ulimit -u` dulu.

> **Tips:** Kalau `ulimit -u` tidak bisa dinaikkan (`Operation not permitted`), cek `ulimit -Hu` — hard limit yang jadi batas atas. Bisa naikkan soft limit sampai setinggi hard limit.

---

## 3.2 Process Management

| Perintah | Fungsi |
|---|---|
| `cmd &` | Background |
| `ps aux \| grep nama` | Cari proses |
| `pgrep -af pola` | Dapatkan PID |
| `kill PID` | SIGTERM (sopan) |
| `kill -9 PID` | SIGKILL (paksa) |
| `killall nama` | Semua dengan nama sama |
| `pkill -f pola` | Berdasarkan pola |

**Sinyal:** `15` SIGTERM · `9` SIGKILL · `1` SIGHUP · `2` SIGINT

> **Tips:** Coba `kill` dulu, baru `kill -9` kalau tidak merespons.

**Trik hindari baris grep di output:**
```bash
$ ps aux | grep [k]illing     # bracket trick
$ pgrep -af killing           # atau pakai pgrep
```

---

## 3.3 Monitoring CPU & Load

| Perintah | Fungsi |
|---|---|
| `lscpu` | Info CPU |
| `nproc` | Jumlah CPU |
| `top` | Monitor real-time |
| `top -bn1` | Batch 1 iterasi |

**Tombol `top`:** `l` `t` `m` (toggle) · `P` `M` (sort) · `1` (per-CPU) · `k` (kill) · `q` (quit)

**Load average:** `1min, 5min, 15min`

> **Aturan:** Load ≈ jumlah CPU → CPU 100% penuh.

> **Analogi:** Load = panjang antrean di kasir.

---

# BAGIAN 4 — PACKAGE & STORAGE

## 4.1 Package Management (APT)

**Konsep:** APT = cara install/uninstall software di Debian/Ubuntu.

| Perintah | Fungsi |
|---|---|
| `sudo apt update` | Refresh daftar paket |
| `sudo apt upgrade -y` | Upgrade paket |
| `apt search <kata>` | Cari paket |
| `apt show <paket>` | Detail paket |
| `sudo apt install <paket> -y` | Install |
| `apt list --installed` | Paket terinstal |
| `sudo apt remove <paket>` | Hapus (config sisa) |
| `sudo apt purge <paket>` | Hapus + config |
| `sudo apt autoremove` | Bersihkan dependensi |

> [!] `remove` sisa config · `purge` bersih total.

> **Tips:** Kalau dapat error `dpkg was interrupted`, jalankan `sudo dpkg --configure -a`.

---

## 4.2 External Repository & Version Pinning

| Perintah | Fungsi |
|---|---|
| `curl -LsS <url> -o script` | Download script |
| `sudo tee /etc/apt/sources.list.d/<name>.list` | Buat repo manual |
| `apt-cache policy <paket>` | Cek versi |
| `sudo apt install <paket>=<versi>` | Install versi spesifik |
| `apt-mark hold <paket>` | Kunci versi |
| `lsb_release -cs` | Codename distro |

> [!] Repo eksternal bisa bentrok. Cek `apt-cache policy` setelah tambah.

> [!] Konflik MariaDB vs MySQL: `mysql-common` dari MariaDB tidak punya file `configure-symlinks` yang dibutuhkan MySQL Oracle. Hapus repo MariaDB dulu sebelum install MySQL.

---

## 4.3 Swap File

**Konsep:** Swap = "memori cadangan" di disk saat RAM penuh.

| Perintah | Fungsi |
|---|---|
| `fallocate -l 2G /swapfile` | Buat file swap |
| `chmod 600 /swapfile` | Permission ketat |
| `mkswap /swapfile` | Format swap |
| `swapon /swapfile` | Aktifkan |
| `swapoff /swapfile` | Nonaktifkan |
| `swapon --show` | Swap aktif |
| `free -h` | Total memori |

**Fstab:** `/swapfile swap swap defaults 0 0`

| RAM | Swap |
|---|---|
| ≤ 2 GB | 2× RAM |
| 2–8 GB | 1× RAM |
| > 8 GB | 0.5× RAM (min 4 GB) |

---

## 4.4 Disk Usage (df & du)

**Perbedaan kunci:**

| Perintah | Menjawab | Sumber |
|---|---|---|
| `df -h` | Sisa disk | Superblock (cepat) |
| `df -Th` | + tipe filesystem | Superblock |
| `df -i` | Inode | Superblock |
| `du -sh .` | Ukuran direktori ini | Setiap file (lambat) |
| `du -h dir/*` | Tiap file | Setiap file |
| `du --max-depth=1` | Per subdir (1 level) | Setiap file |

> **Tips:** Disk penuh? `df -h` → `du -sh /* \| sort -h`.

---

## 4.5 Partition Management

| Perintah | Fungsi |
|---|---|
| `lsblk` / `lsblk -f` | Lihat disk & partisi |
| `sudo parted /dev/sdX` | Tool partisi GPT/MBR |
| `(parted) mklabel gpt` | Table GPT |
| `(parted) mkpart` | Buat partisi |
| `(parted) set N lvm on` | Flag LVM |
| `sudo fdisk /dev/sdX` | Tool partisi |
| `mkfs -t ext4 -L LABEL /dev/sdX1` | Format + label |
| `blkid /dev/sdX1` | Label + UUID |
| `mount LABEL=NAME /mnt/point` | Mount dengan label |
| `mount -a` | Mount semua dari fstab |
| `partprobe /dev/sdX` | Reload table |

**Fstab aman:** `LABEL=NAMA /mnt/point ext4 defaults,nofail 0 2`

> [X] Selalu test `mount -a` sebelum reboot — fstab rusak = sistem tidak boot.

---

## 4.6 LVM (Logical Volume Manager)

**Konsep:** LVM = "partisi virtual" yang bisa di-resize tanpa reboot.

**Alur:** `partisi → pvcreate → vgcreate → lvcreate → mkfs → mount → fstab`

| Perintah | Fungsi |
|---|---|
| `pvcreate /dev/sdb1` | Physical Volume |
| `vgcreate <vg> /dev/sdb1 /dev/sdb2` | Volume Group |
| `lvcreate -n <lv> -L 400M <vg>` | Logical Volume |
| `mkfs -t ext4 /dev/<vg>/<lv>` | Format LV |
| `pvs` / `vgs` / `lvs` | Info ringkas |
| `vgextend <vg> /dev/sdc1` | Tambah PV |
| `lvextend -L +200M <lv>` → `resize2fs` | Perbesar |
| `resize2fs` → `lvreduce -L 200M <lv>` | Perkecil (FS dulu!) |
| `lvremove` → `vgremove` → `pvremove` | Hapus |

**Aturan:**
- Flag `lvm on` wajib
- `udevadm settle` sebelum `pvcreate`
- `systemctl daemon-reload` setelah edit fstab
- Perbesar: `lvextend` → `resize2fs`
- Perkecil: `resize2fs` → `lvreduce`

---

## 4.7 Disk Baru + Auto-Mount

**Alur:** `Tambah disk → Partisi → Format + label → Mount → fstab`

**Langkah lengkap:**

```bash
# 1. Verifikasi disk
lsblk

# 2. Buat partisi (GPT, 1 GiB)
sudo parted /dev/sdd
(parted) mklabel gpt
(parted) mkpart primary ext4 1MiB 1025MiB
(parted) quit

# 3. Format + label
sudo mkfs -t ext4 -L appcache /dev/sdd1

# 4. Cek label
sudo blkid /dev/sdd1

# 5. Buat mount point & mount
sudo mkdir -p /app
sudo mount LABEL=appcache /app

# 6. Edit fstab (backup dulu!)
sudo cp /etc/fstab /etc/fstab.bak
sudo nano /etc/fstab
# Tambah: LABEL=appcache  /app  ext4  defaults,nofail  0  2

# 7. Test fstab (WAJIB!)
sudo umount /app
sudo mount -a
df -h | grep app
```

**Format `/etc/fstab`:**

```
LABEL=appcache  /app  ext4  defaults,nofail  0  2
```

| Kolom | Arti |
|---|---|
| LABEL=xxx | Identifikasi device |
| /app | Mount point |
| ext4 | Tipe FS |
| defaults,nofail | Opsi |
| 0 | Dump |
| 2 | Fsck pass |

**Tips:**
- Pakai **MiB/GiB** (binary), bukan MB/GB.
- Start partisi di **1MiB** untuk alignment.
- **LABEL** lebih stabil dari `/dev/sdX`.
- **`nofail`** untuk disk non-esensial → sistem tetap boot kalau disk hilang.
- **`mount -a`** WAJIB test sebelum reboot — fstab rusak = gagal boot.
- Kalau sudah gagal boot: rescue mode → `mount -o remount,rw /` → fix fstab.

---

# BAGIAN 5 — USER & SECURITY

## 5.1 Inode & Links

**Konsep:** Inode = "KTP" file. Nama file hanya label yang menunjuk inode.

| Perintah | Fungsi |
|---|---|
| `ls -li file` | Inode + link count |
| `ln source target` | Hard link |
| `ln -s source target` | Symlink |
| `readlink link` | Target symlink |
| `stat file` | Info lengkap |

| Aspek | Hard | Symbolic |
|---|---|---|
| Menunjuk ke | Inode | Path |
| Inode number | Sama | Beda |
| Lintas FS | Tidak | Ya |
| Ke direktori | Tidak | Ya |
| Target hapus | Data tetap | Rusak (dangling) |

> [X] Jangan pakai trailing slash saat hapus symlink direktori: `rm -r symlink/` bisa hapus isi target!

---

## 5.2 User & Group Management

| Perintah | Fungsi |
|---|---|
| `useradd -m -s /bin/bash <user>` | Buat user + home + shell |
| `userdel -r <user>` | Hapus + home |
| `usermod -c "comment" <user>` | Set comment |
| `usermod -aG <group> <user>` | Tambah ke group |
| `usermod -L` / `-U` | Lock / unlock |
| `passwd <user>` | Set password |
| `passwd -S <user>` | Cek status |
| `groupadd -g <gid> <group>` | Buat group |
| `id <user>` / `groups <user>` | Info group |
| `sudo -l -U <user>` | Cek sudo |
| `visudo -c` | Validasi sudoers |

**File penting:**

| File | Perm | Isi |
|---|---|---|
| `/etc/passwd` | 644 | Data user |
| `/etc/shadow` | 400 | Hash password |
| `/etc/group` | 644 | Data group |
| `/etc/gshadow` | 400 | Password group |
| `/etc/skel/` | — | Template home |

**Sudoers:**
```
%group  ALL=(ALL)  NOPASSWD:ALL
user    ALL=(root) NOPASSWD:/usr/bin/systemctl restart nginx
```

> **Aturan emas:**
> - Selalu `usermod -aG` (append), bukan `-G`
> - User harus logout & login ulang setelah ganti group
> - Permission `/etc/sudoers.d/` harus `0440`
> - Hindari `NOPASSWD:ALL` di production

> [X] Jangan edit `/etc/passwd`, `/etc/shadow`, `/etc/group` langsung — pakai tools resmi.

---

## 5.3 Password Aging (chage)

| Perintah | Fungsi |
|---|---|
| `chage -l <user>` | Info aging |
| `chage -M 30 -m 7 -W 5 <user>` | Set max/min/warning |
| `chage -d 0 <user>` | Paksa ganti saat login |
| `chage -E YYYY-MM-DD <user>` | Account expiration |

**Default di `/etc/login.defs`:**
```
PASS_MAX_DAYS   90
PASS_MIN_DAYS   1
PASS_WARN_AGE   7
```
> Hanya untuk user baru.

---

## 5.4 Restricted User & Access Control

**Kapan dipakai:** Guest account, akun demo, akun untuk otomasi terbatas.

```bash
# 1. Buat user
useradd -m -s /bin/bash <user> && passwd <user>

# 2. Ganti ke rbash
usermod -s /bin/rbash <user>

# 3. Direktori command
mkdir -p /home/<user>/bin
chown <user>:<user> /home/<user>/bin
chmod 755 /home/<user>/bin

# 4. Batasi PATH
echo 'export PATH=$HOME/bin' >> /home/<user>/.bash_profile

# 5. Symlink command yang diizinkan
ln -s /bin/ls /home/<user>/bin/ls
ln -s /bin/cat /home/<user>/bin/cat
ln -s /usr/bin/sudo /home/<user>/bin/sudo

# 6. Group
groupadd -f limited<user>
usermod -aG limited<user> <user>

# 7. Sudoers
visudo -f /etc/sudoers.d/limited<user>
# %limited<user> ALL=(ALL) NOPASSWD: /bin/ls, /bin/cat
chmod 440 /etc/sudoers.d/limited<user>

# 8. Batasi SSH
echo "-:limited<user>:ALL EXCEPT LOCAL" >> /etc/security/access.conf
```

**Batasan rbash:** tidak bisa `cd`, set env, command dengan `/`, redirect.

> [!] rbash bukan security tool — mudah di-bypass.

---

## 5.5 SSH & SSH Key Authentication

**Konsep:** SSH = akses remote aman. Key-based auth = login tanpa password.

| Perintah | Fungsi |
|---|---|
| `ssh user@host` | Login remote |
| `ssh user@host "cmd"` | Perintah remote |
| `scp file user@host:/path` | Copy file |
| `ssh-keygen -t rsa -b 4096 -N ""` | Generate key pair |
| `ssh-copy-id -i key.pub -p PORT user@host` | Salin public key |
| `ssh -p PORT -o BatchMode=yes user@host "cmd"` | Verifikasi passwordless |

**File & permission:**

| File | Perm |
|---|---|
| `~/.ssh` | 700 |
| `~/.ssh/id_rsa` | 600 |
| `~/.ssh/id_rsa.pub` | 644 |
| `~/.ssh/authorized_keys` | 600 |

> **Tips:** Private key tidak pernah dikirim ke server — hanya public key.

---

## 5.6 Kernel & Boot

| Perintah | Fungsi |
|---|---|
| `cat /proc/cmdline` | Boot parameters aktif |
| `man bootparam` | Dokumentasi |
| `lsmod` | Kernel modules |
| `modprobe <module>` | Load module |
| `mount -o remount,rw /` | Remount read-write |

**Parameter umum:** `root=UUID=...` · `ro` · `quiet` · `nomodeset` · `crashkernel=`

**Rescue Mode:** GRUB → Advanced options → recovery mode

| Opsi | Fungsi |
|---|---|
| `resume` | Boot normal |
| `clean` | Bersihkan disk |
| `dpkg` | Perbaiki paket |
| `fsck` | Cek filesystem |
| `grub` | Update bootloader |
| `root` | Root shell |

---

## 5.7 System Startup & GRUB (Teori)

**Boot Sequence:**
```
1. BIOS/UEFI (POST)
2. Boot Loader (GRUB)
3. Kernel Initialization
4. Init/systemd
```

**BIOS vs UEFI:**

| Aspek | BIOS | UEFI |
|---|---|---|
| Boot table | MBR | GPT |
| Max disk | 2 TB | Sangat besar |
| Secure Boot | ❌ | ✅ |

**Shutdown & Reboot:**

| Perintah | Fungsi |
|---|---|
| `sudo shutdown -h now` | Shutdown |
| `sudo shutdown -r now` | Reboot |
| `sudo shutdown -h +1 "msg"` | Shutdown dalam 1 menit |
| `sudo shutdown -c` | Cancel |
| `sudo reboot` / `halt` / `poweroff` | Legacy wrapper |

**GRUB:**

| File | Fungsi |
|---|---|
| `/etc/default/grub` | Config utama (edit di sini) |
| `/boot/grub/grub.cfg` | Auto-generated (jangan edit) |
| `/etc/grub.d/` | Script generator menu |

**Regenerate GRUB:**
```bash
$ sudo update-grub                            # Debian/Ubuntu
$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg # RHEL/Fedora
```

**GRUB interaktif:**
- `e` → edit entry sementara (tambah `single` untuk rescue)
- `c` → GRUB shell
- `Ctrl+X` / `F10` → boot dengan parameter baru

**Tips:**
- Jangan edit `grub.cfg` langsung.
- Backup GRUB sebelum ubah: `cp /boot/grub/grub.cfg /boot/grub/grub.cfg.bak`
- Single-user mode untuk reset password root.
- Reboot tidak selalu fix masalah — investigasi dulu.

---

## 5.8 systemd Service

**Konsep:** systemd = init + service manager modern. Unit file di `/etc/systemd/system/`.

**3 section unit file:**

```ini
[Unit]
Description=<nama>
After=network.target

[Service]
ExecStart=<path-absolut-perintah>
ExecStop=<path-absolut-perintah>

[Install]
WantedBy=multi-user.target
```

**Perintah `systemctl`:**

| Perintah | Fungsi |
|---|---|
| `systemctl start <svc>` | Jalankan |
| `systemctl stop <svc>` | Hentikan |
| `systemctl restart <svc>` | Restart |
| `systemctl status <svc>` | Cek status |
| `systemctl enable <svc>` | Auto-start saat boot |
| `systemctl disable <svc>` | Matikan auto-start |
| `systemctl is-active <svc>` | Cek running |
| `systemctl is-enabled <svc>` | Cek auto-start |
| `systemctl daemon-reload` | Baca ulang unit file |
| `systemctl mask <svc>` | Block service |
| `systemctl edit <svc>` | Override service bawaan |
| `journalctl -u <svc> -f` | Log real-time |

**Target vs Runlevel:**

| Runlevel | Target |
|---|---|
| 0 | poweroff.target |
| 1 | rescue.target |
| 3 | multi-user.target |
| 5 | graphical.target |
| 6 | reboot.target |

**Tips:**
- `daemon-reload` **wajib** setelah edit unit file.
- `ExecStart` pakai **path absolut** (`/bin/echo`, bukan `echo`).
- Service yang selesai cepat akan tampak `inactive` — normal.
- `systemctl edit <svc>` untuk override tanpa edit file asli.
- Cek log dengan `journalctl -u <svc>`.

---

## 5.9 System Init (Teori)

**Konsep:** `init` = proses pertama user-space. Orkestrator startup.

**Evolusi init:**

| Aspek | SysVinit | Upstart | systemd |
|---|---|---|---|
| Model | Serial | Event | Paralel |
| Startup | Lambat | Sedang | Cepat |
| Config | Shell script | `.conf` | Unit file |
| Status | Legacy | Obsolete | **Standard** |

**Runlevel vs Target:**

| Runlevel | Target systemd |
|---|---|
| 0 | poweroff.target |
| 1 | rescue.target |
| 3 | multi-user.target |
| 5 | graphical.target |
| 6 | reboot.target |

**Fitur utama systemd:**
- Paralelisasi startup
- Dependency management
- Socket & D-Bus activation
- cgroups tracking
- Binary + config (bukan shell script)
- Transactional service control
- SysVinit compatibility

**File konfigurasi:**

| File | Fungsi |
|---|---|
| `/etc/hostname` | Hostname |
| `/etc/vconsole.conf` | Keyboard & font console |
| `/etc/sysctl.d/*.conf` | Kernel parameter |
| `/etc/os-release` | Info OS |

**`journalctl` — log systemd:**

| Perintah | Fungsi |
|---|---|
| `journalctl` | Semua log |
| `journalctl -u <svc>` | Log service |
| `journalctl -f` | Real-time |
| `journalctl -b` | Sejak boot terakhir |
| `journalctl -n 50` | 50 baris terakhir |

**Tips:**
- `systemctl get-default` → cek target default.
- `systemctl set-default multi-user.target` → ubah target.
- `systemd-analyze blame` → cek service yang lambat.
- systemd = standar industri, dipakai hampir semua distro.

---

## 5.10 User Management — Advanced

**Alur lengkap:**

```bash
# 1. Buat user dengan shell
sudo useradd -m -s /bin/bash trainer1
sudo useradd -m -s /bin/bash trainer2
sudo useradd -m -s /bin/bash trainer3

# 2. Set password non-interaktif (PAKAI KUTIP TUNGGAL!)
echo 'trainer1:Train!@#123' | sudo chpasswd
echo 'trainer2:Train!@#123' | sudo chpasswd
echo 'trainer3:Train!@#123' | sudo chpasswd

# 3. Comment
sudo usermod -c "Trainer One" trainer1

# 4. Password policy
sudo chage -M 7 trainer1       # max 7 hari
sudo usermod -L trainer2       # lock
sudo chage -d 0 trainer3       # force change

# 5. Group
sudo groupadd coaches
sudo usermod -aG coaches trainer1
sudo usermod -aG coaches trainer2

# 6. Ubah shell
sudo usermod -s /bin/sh trainer3
```

**Perintah penting:**

| Perintah | Fungsi |
|---|---|
| `useradd -m -s /bin/bash <user>` | Buat user + home + shell |
| `echo 'user:pass' \| chpasswd` | Set password non-interaktif |
| `usermod -c "Comment" <user>` | Set comment (GECOS) |
| `chage -M N <user>` | Max password age |
| `chage -d 0 <user>` | Force change password |
| `usermod -L` / `-U` | Lock / unlock |
| `passwd -l` / `-e` | Lock / expire password |
| `groupadd -f <group>` | Buat group (idempotent) |
| `usermod -aG <group> <user>` | Append ke group (**wajib `-a`**) |
| `usermod -s /bin/sh <user>` | Ubah shell |
| `chage -l <user>` | Lihat aging info |
| `passwd -S <user>` | Cek status password |
| `id <user>` / `groups <user>` | Verifikasi group |

**⚠️ Urutan Sangat Penting:**

```
1. useradd         → buat user
2. chpasswd        → set password DULU ←
3. usermod -c      → comment
4. chage -M        → policy
5. usermod -L      → LOCK SETELAH password
6. chage -d 0      → FORCE CHANGE SETELAH password
7. groupadd + -aG  → group
8. usermod -s      → shell
```

**Kenapa?** `chpasswd` **menghapus** flag `!` (locked) di `/etc/shadow` dan **menimpa** tanggal "last change". Kalau `usermod -L` atau `chage -d 0` dijalankan **sebelum** `chpasswd`, konfigurasinya akan **hilang**.

**Tips:**
- **`usermod -aG` (append)** vs **`usermod -G` (overwrite)** — jangan salah!
- Password dengan `!` pakai **kutip tunggal**: `echo 'user:Train!@#123'`.
- Lock lebih baik daripada delete untuk user production.
- Group baru aktif setelah **logout-login ulang**.
- Verifikasi: `passwd -S` untuk status, `chage -l` untuk aging, `id` untuk group.

---

# BAGIAN 6 — PERMISSIONS

## 6.1 chmod — Symbolic Mode

**Konsep:** `chmod` mengubah permission file. Symbolic mode pakai huruf + operator.

| Simbol | Arti |
|---|---|
| `u` / `g` / `o` / `a` | User / Group / Others / All |
| `r` / `w` / `x` | Read / Write / Execute |
| `=` / `+` / `-` | Set / Tambah / Hapus |

```bash
$ chmod u=r,g=w,o=x file      # set
$ chmod u+w,g-w file          # modifikasi
$ chmod ug-rwx,o-rw file      # hapus
```

**Baca `ls -l`:**
```
-rw-rw-r-- 1 student student 0 ... file
│└┬┘└┬┘└┬┘
│ │  │  └── others
│ │  └───── group
│ └──────── user
└────────── tipe (- file, d dir, l symlink)
```

> **Tips:** `=` reset, `+`/`-` modifikasi.

> [X] Jangan `chmod 777` di production.

---

## 6.2 chmod — Numeric Mode (Octal)

**Konversi:** `r=4` · `w=2` · `x=1`

| r | w | x | Angka |
|---|---|---|---|
| ✓ | ✓ | ✓ | 7 |
| ✓ | ✓ | — | 6 |
| ✓ | — | ✓ | 5 |
| ✓ | — | — | 4 |

**Kombinasi umum:**

| Angka | Permission | Kapan |
|---|---|---|
| `644` | `rw-r--r--` | File data |
| `755` | `rwxr-xr-x` | Script, direktori |
| `600` | `rw-------` | File sensitif |
| `700` | `rwx------` | Direktori private |
| `640` | `rw-r-----` | File group-shared |

**Format:** `chmod [user][group][others] file`

> **Tips:** Selalu 3 digit. Digit ke-4 = special permission.

---

## 6.3 ACL (Access Control List)

**Konsep:** Permission tambahan di luar user/group/others. Untuk akses ke user spesifik.

**Tanda file punya ACL:** `+` di `ls -l`

| Perintah | Fungsi |
|---|---|
| `sudo apt install acl -y` | Install paket |
| `setfacl -m u:user:r file` | ACL user (read) |
| `setfacl -m u:user:rw file` | ACL user (read+write) |
| `setfacl -m g:group:r file` | ACL group |
| `setfacl -x u:user file` | Hapus ACL user |
| `setfacl -b file` | Hapus semua ACL |
| `setfacl -d -m u:user:r dir/` | Default ACL |
| `setfacl -R -m u:user:r dir/` | Rekursif |
| `getfacl file` | Lihat ACL |

**Struktur:**
```
user::rw-          ← owner
user:user2:r--     ← ACL user
group::rw-         ← group owner
mask::rw-          ← batas max ACL
other::---         ← others
```

> **Tips:** Kalau ACL tidak bekerja, cek `mask`. `-d` = default, `-R` = rekursif.

---

## 6.4 chown & chgrp

| Perintah | Fungsi |
|---|---|
| `chown user file` | Ubah owner |
| `chown user:group file` | Ubah owner & group |
| `chown -R user:group dir/` | Rekursif |
| `chgrp group file` | Ubah group saja |

> **Catatan:** Hanya root yang bisa `chown`. `chgrp` bisa oleh root atau anggota group.

---

## 6.5 Special Permissions (SUID, SGID, Sticky)

| Simbol | Nama | Efek |
|---|---|---|
| `s` (user) | SUID | Jalankan executable sebagai owner |
| `s` (group) | SGID | Jalankan sebagai group / direktori warisi group |
| `t` (others) | Sticky bit | Hanya owner bisa hapus file di direktori |

**Set:**

```bash
$ chmod u+s file      # SUID
$ chmod g+s file      # SGID
$ chmod +t dir/       # Sticky bit
$ chmod 4755 file     # SUID (numeric)
$ chmod 2755 file     # SGID (numeric)
$ chmod 1777 dir/     # Sticky (numeric)
```

**Contoh:** `/usr/bin/passwd` punya SUID (`-rwsr-xr-x`) — user biasa bisa ubah password.

---

## 6.6 File Attributes (chattr & lsattr)

**Konsep:** Atribut khusus — bahkan root tidak bisa bypass.

| Atribut | Simbol | Efek |
|---|---|---|
| Immutable | `i` | Tidak bisa diubah/dihapus — bahkan root |
| Append-only | `a` | Hanya bisa ditambah |

| Perintah | Fungsi |
|---|---|
| `lsattr file` | Lihat atribut |
| `chattr +i file` | Set immutable |
| `chattr -i file` | Hapus immutable |
| `chattr +a file` | Set append-only |
| `chattr -a file` | Hapus append-only |

> [!] Jangan lupa `chattr -i` sebelum menghapus file immutable.

---

# BAGIAN 7 — REFERENSI CEPAT & TROUBLESHOOTING

## 7.1 Perbedaan Penting

| Konsep | Penjelasan |
|---|---|
| `>` vs `>>` | Timpa vs tambah |
| `rmdir` vs `rm -r` | Kosong vs berisi |
| `soft` vs `hard` limit | Aktif vs batas |
| `remove` vs `purge` | Sisa config vs bersih |
| `df` vs `du` | Filesystem vs direktori |
| `usermod -G` vs `-aG` | Overwrite vs append |
| Hard link vs symlink | Inode vs path |
| `=` vs `+` vs `-` (chmod) | Reset vs tambah vs hapus |
| `r` vs `rX` (ACL) | Read-only vs read + execute kondisional |
| `ip` vs `ifconfig` | Modern vs legacy |
| `reload` vs `restart` (service) | Baca ulang config vs matikan-nyalakan |
| HTTP vs HTTPS | Port 80 vs 443 |
| Self-signed vs CA-signed | Warning di browser vs tanpa warning |
| Firewalld vs UFW | Zone-based vs rule-based |
| phpMyAdmin vs CLI | GUI web vs terminal MySQL |
| Apache vs Nginx | Process-based vs event-driven |
| Runtime vs Permanent (firewalld) | Sementara vs persistent |
| BIOS vs UEFI | Legacy vs modern firmware |
| Runlevel vs Target | Nama lama vs systemd |
| `start` vs `enable` | Jalankan sekarang vs auto-start |
| `Files differ` vs `Only in` | Isi beda vs file unik |
| `diff` vs `comm` vs `cmp` | Baris vs sorted vs byte |

---

## 7.2 Danger Zone — Perintah Berbahaya

| Perintah | Bahaya |
|---|---|
| [X] `rm -rf /` | Hapus sistem |
| [X] `rm -rf ~/` | Hapus home |
| [X] `rm -r symlink/` | Hapus isi target |
| [X] Edit `/etc/fstab` tanpa test | Tidak boot |
| [X] Edit `/etc/passwd` langsung | Lock dari sistem |
| [X] `mkfs` di partisi ter-mount | Hapus data |
| [X] `chmod -R 777 /` | Rusak permission |
| [X] `dd if=/dev/zero of=/dev/sda` | Wipe disk |
| [X] Salah set default gateway | Putus koneksi network |
| [X] Edit vhost Apache tanpa `configtest` | Apache error |
| [X] Bagikan private key SSL (`.key`) | Siapa pun bisa impersonate server |
| [X] Lupa `-nodes` di openssl | Apache minta passphrase tiap restart |
| [X] Buka phpMyAdmin ke publik tanpa firewall | Target utama attacker |
| [X] `GRANT ALL PRIVILEGES ON *.* TO 'user'@'%'` | Akses penuh dari mana saja |
| [X] Edit config Nginx tanpa `nginx -t` | Nginx mati saat reload |
| [X] Edit `grub.cfg` langsung | Perubahan hilang saat update-grub |
| [X] Reboot server produksi tanpa alasan | Downtime tidak perlu |
| [X] Password dengan `!` pakai double quote | Bash error: event not found |

---

## 7.3 Alur Troubleshooting Umum

| Masalah | Langkah Awal |
|---|---|
| Tidak boot | Cek fstab, rescue mode |
| Disk penuh | `df -h` → `du -sh /* \| sort -h` |
| Lupa password root | Rescue mode → `passwd` |
| Fork error | `ulimit -u` |
| SSH gagal | Cek `~/.ssh` permission |
| Load tinggi | `top`, bandingkan `nproc` |
| Permission denied | `ls -l`, mungkin butuh `sudo` |
| ACL tidak berlaku | Cek `mask` dengan `getfacl` |
| Network tidak jalan | `ping 8.8.8.8` → `ip a` → `ip route` |
| DNS tidak resolve | Cek `/etc/resolv.conf`, uji dengan `dig` |
| Apache tidak jalan | `systemctl status apache2`, `apache2ctl configtest` |
| HTTPS warning | Normal untuk self-signed; produksi pakai Let's Encrypt |
| SSL handshake failed | Cek `mod_ssl` aktif, path cert benar |
| phpMyAdmin "Not Found" | Cek `a2query -s`, `Listen 8088` di ports.conf |
| phpMyAdmin login gagal | Cek user MySQL, `SELECT user, host, plugin FROM mysql.user` |
| Firewalld block akses | `firewall-cmd --list-all --zone=public` |
| Firewalld memblok SSH | `firewall-cmd --add-service=ssh --permanent` |
| Nginx tidak jalan | `nginx -t`, cek `journalctl -u nginx` |
| `dpkg was interrupted` | `sudo dpkg --configure -a` |
| Package konflik (MySQL/MariaDB) | Purge semua, hapus repo, install ulang |
| Systemd service gagal | `systemctl status <svc>`, `journalctl -u <svc>` |
| GRUB tidak muncul | Cek `/etc/default/grub`, `update-grub` |
| Bash `!` error | Pakai kutip tunggal: `echo 'user:pass!@#'` |

---

## 7.4 Best Practice

1. Pakai `sudo`, jangan login root langsung
2. `ls -lah` sebelum hapus
3. `rm -ri` daripada `rm -r`
4. Test fstab dengan `mount -a`
5. Jangan edit file sistem langsung
6. Backup sebelum eksperimen
7. Baca `man`
8. Test di VM sebelum produksi
9. Jangan `chmod 777`
10. Catat apa yang dilakukan
11. YAML pakai **spasi**, bukan Tab
12. `apache2ctl configtest` sebelum reload Apache
13. **Self-signed = testing**, produksi pakai **Let's Encrypt**
14. Private key SSL **jangan pernah** dibagikan, permission `600`
15. **phpMyAdmin di production publik = risiko** — batasi via firewall
16. **Firewalld: `--permanent` + `--reload`** untuk commit perubahan
17. **`nginx -t`** sebelum reload Nginx
18. **Pilih MySQL atau MariaDB**, jangan keduanya
19. **Jangan edit `grub.cfg`** — edit `/etc/default/grub`
20. **`systemctl daemon-reload`** setelah edit unit file
21. **Reboot ≠ fix masalah** — investigasi dulu
22. **Backup unit file** sebelum ubah service
23. **Ubah satu hal, test** — jangan ubah banyak sekaligus
24. **Baca error log** dulu sebelum bertindak
25. **Password dengan `!` → pakai kutip tunggal**
26. **`chpasswd` hapus lock & force change** — set password dulu, baru policy
27. **`find -size` pakai pembulatan** — untuk range inklusif, pakai byte (`c`)
28. **Baca output `diff` mentah dulu** — bisa `Files differ` atau `Only in`

---

## 7.5 Glosarium

| Istilah | Arti |
|---|---|
| **ACL** | Access Control List — permission granular |
| **Apache** | Web server populer di Linux |
| **a2ensite / a2dissite** | Enable/disable virtual host |
| **a2enmod / a2dismod** | Enable/disable module Apache |
| **APT** | Package manager Debian/Ubuntu |
| **Bash** | Shell default Linux |
| **BIOS** | Firmware legacy — pakai MBR |
| **CA** | Certificate Authority — penerbit certificate |
| **Certificate** | File yang membuktikan identitas server |
| **chattr** | Change attribute |
| **chmod** | Change mode — ubah permission |
| **chown** | Change owner |
| **chgrp** | Change group |
| **chpasswd** | Set password non-interaktif |
| **CIDR** | Classless Inter-Domain Routing |
| **comm** | Bandingkan file sorted |
| **cmp** | Bandingkan byte-by-byte |
| **Cron** | Scheduler tugas berkala |
| **CSR** | Certificate Signing Request |
| **Daemon** | Proses background |
| **daemon-reload** | Baca ulang unit file systemd |
| **df** | Disk Free |
| **diff** | Bandingkan file baris per baris |
| **dig** | DNS lookup tool |
| **DNS** | Domain Name System |
| **DocumentRoot** | Direktori file web |
| **du** | Disk Usage |
| **enable** | Auto-start service saat boot |
| **ext4** | Filesystem default |
| **find** | Cari file berdasarkan kriteria |
| **Firewalld** | Firewall modern dengan konsep zone |
| **FQDN** | Fully Qualified Domain Name |
| **fstab** | File System Table |
| **GECOS** | Comment di `/etc/passwd` |
| **GID** | Group ID |
| **grep** | Global Regular Expression Print |
| **GRUB** | Bootloader default Linux |
| **Hard link** | Nama tambahan ke inode sama |
| **History expansion** | Ekspansi `!` di bash — sumber error |
| **Hostname** | Nama perangkat di jaringan |
| **HTTPS** | HTTP + SSL/TLS (port 443) |
| **ifconfig** | Legacy network tool (deprecated) |
| **Inode** | Metadata file (bukan nama) |
| **ip** | Modern network tool |
| **IPv4 / IPv6** | Versi IP address |
| **journalctl** | Tool lihat log systemd |
| **Kernel** | Inti OS |
| **LAMP** | Linux + Apache + MySQL + PHP |
| **Let's Encrypt** | CA gratis, otomatis (Certbot) |
| **Load average** | Antrean proses |
| **LVM** | Logical Volume Manager |
| **MBR** | Master Boot Record — skema partisi lama |
| **mod_ssl** | Modul Apache untuk SSL |
| **MySQL** | Database server populer |
| **MariaDB** | Fork MySQL, kompatibel penuh |
| **mtr** | Ping + traceroute real-time |
| **Netplan** | Konfigurasi network Ubuntu (YAML) |
| **NetworkManager** | Service network modern |
| **Nginx** | Web server ringan, event-driven |
| **OpenSSL** | Tool untuk generate/manage certificate |
| **OOM** | Out of Memory |
| **PAM** | Pluggable Auth Modules |
| **Permission** | Hak akses |
| **PHP** | Bahasa pemrograman web |
| **phpMyAdmin** | GUI web untuk MySQL/MariaDB |
| **ping** | Uji konektivitas |
| **Pipe** | `\|` — sambung output |
| **PNIDN** | Predictable Network Interface Device Names |
| **POST** | Power-On Self-Test |
| **Private IP** | IP internal (10/8, 172.16/12, 192.168/16) |
| **Private key** | File rahasia SSL (`.key`, 600) |
| **PV/VG/LV** | Physical/Volume/Logical (LVM) |
| **rbash** | Restricted bash |
| **reload / restart** | Baca ulang vs matikan-nyalakan service |
| **Rescue mode** | Environment minimal untuk perbaikan darurat |
| **Root** | Superuser |
| **Routing** | Pemilihan jalur paket |
| **Runlevel** | Level operasi SysVinit (0–6) |
| **sed** | Stream editor — substitusi string |
| **Self-signed** | Certificate yang ditandatangani sendiri |
| **SELinux/AppArmor** | Mandatory access control |
| **SGID** | Set Group ID |
| **SIGTERM/SIGKILL** | Sinyal 15 / 9 |
| **SSH** | Secure Shell |
| **SSL/TLS** | Protokol enkripsi untuk HTTPS |
| **SUID** | Set User ID |
| **Sticky bit** | Proteksi hapus di direktori |
| **sudo** | Superuser Do |
| **Swap** | Memori cadangan |
| **Symlink** | Shortcut |
| **systemctl** | Perintah kontrol systemd |
| **systemd** | Init + service manager modern |
| **Target** | Pengganti runlevel di systemd |
| **traceroute** | Lihat jalur paket |
| **Troubleshooting** | Proses diagnosis & resolusi masalah |
| **UEFI** | Firmware modern — pakai GPT |
| **UFW** | Uncomplicated Firewall (Ubuntu) |
| **UID** | User ID |
| **ulimit** | User limit |
| **Umask** | Filter permission default |
| **unit file** | File definisi service systemd |
| **Vim** | Editor powerful |
| **VirtualHost** | Konfigurasi domain di Apache |
| **wc** | Word count |
| **www-data** | User default Apache/Nginx |
| **Zombie** | Proses selesai belum di-reap |
| **Zone (firewalld)** | Kumpulan aturan firewall |

---

## 7.6 Kalau Panic

| Situasi | Langkah |
|---|---|
| Tidak boot | Rescue mode dari GRUB |
| Lupa password root | Rescue → `passwd` |
| Disk penuh | `df -h` → `du -sh /* \| sort -h` |
| Fork error | `ulimit -u` (naikkan sampai `ulimit -Hu`) |
| Command hang | `CTRL + C` |
| Vim stuck | `Esc` → `:q!` |
| Nano stuck | `CTRL + X` → `N` |
| Terminal kacau | `reset` atau `stty sane` |
| Lupa perintah | `man <cmd>` atau `--help` |
| Network down | `ping 8.8.8.8` → cek `ip a` → `ip route` |
| Apache error | `apache2ctl configtest` → cek `/var/log/apache2/error.log` |
| Nginx error | `nginx -t` → cek `/var/log/nginx/error.log` |
| SSL error | Cek `a2enmod ssl`, path cert di vhost, permission `.key` |
| phpMyAdmin error | Cek `ss -tlnp \| grep :8088`, `a2query -s`, `firewall-cmd --list-all` |
| Firewalld block | `firewall-cmd --add-port=<port>/tcp --permanent && firewall-cmd --reload` |
| `dpkg` interrupted | `sudo dpkg --configure -a` |
| Service stuck | Kill proses, restart VM kalau perlu |
| Systemd service gagal | `journalctl -u <svc> -n 50 --no-pager` |
| GRUB error | Boot ke Live USB → `grub-install` + `update-grub` |
| Bash `!` error | Pakai kutip tunggal |

> **Tips:** Kalau panik — jangan ketik apapun dulu. Tarik napas, baca error, baru cari solusi.

---

## 7.7 Troubleshooting Apache

**Alur investigasi:**

```
1. systemctl status apache2         → pesan error
2. journalctl -u apache2 -n 50      → log systemd
3. tail /var/log/apache2/error.log  → error spesifik
4. apache2ctl configtest            → cek syntax
5. ss -tlnp | grep :80              → cek port
6. ls -la /var/www/html/            → cek file & perm
7. grep -A 5 "<Directory /var/www/>" apache2.conf  → cek Require
8. Fix → restart → curl verify
```

**Penyebab umum gagal start:**

| Kategori | % | Contoh |
|---|---|---|
| Config error | 50% | Syntax salah, `;` hilang, direktif salah tempat |
| Permission | 30% | www-data tidak bisa baca |
| Port conflict | 20% | Nginx / proses lain |

**403 Forbidden — cek:**

- `Require all denied` di `<Directory /var/www/>` → ubah ke `granted`
- `index.html` tidak ada
- Permission: file `644`, dir `755`
- Owner: `www-data:www-data`

**Perintah penting:**

| Perintah | Fungsi |
|---|---|
| `systemctl status apache2` | Cek status + error |
| `journalctl -u apache2 -n 50` | Log systemd |
| `apache2ctl configtest` | Validasi syntax |
| `apache2ctl -S` | Ringkasan vhost |
| `ss -tlnp \| grep :80` | Cek port |
| `curl -v http://localhost` | Debug HTTP |
| `sed -n 'X,Yp' apache2.conf` | Lihat baris config |
| `grep -rn "pattern" /etc/apache2/` | Cari direktif |

**Tips:**
- **Jangan reinstall** — 99% bisa di-fix dengan config.
- Error bisa **bertingkat** — fix satu, cek lagi.
- Error code: 403 = forbidden, 404 = not found, 500 = server error.
- `SSLEngine` hanya boleh di dalam `<VirtualHost *:443>`, bukan global config.

---

## 7.8 Basic Troubleshooting (Teori)

**Resep 6 langkah:**

```
1. Characterize  → apa yang rusak, kapan mulai
2. Reproduce     → bisa direproduksi?
3. Easy first    → cek service, kabel, restart
4. Eliminate     → satu per satu
5. One change    → ubah satu hal, test
6. Check logs    → /var/log/, journalctl
```

**Level troubleshooting:**
- **Beginner** → ikut panduan
- **Experienced** → paham "kenapa"
- **Wizard** → debug kernel-level

**Networking check (berurutan):**

| # | Cek | Perintah |
|---|---|---|
| 1 | IP | `ip a` |
| 2 | Driver | `lsmod`, `ls /sys/class/net/` |
| 3 | Connectivity | `ping -c3 8.8.8.8` |
| 4 | Routing | `ip route` |
| 5 | DNS | `dig`, `cat /etc/resolv.conf` |

**Boot failure:**

| Tahap | Gejala | Solusi |
|---|---|---|
| BIOS/UEFI | Tidak POST | Cek BIOS setup |
| GRUB | Tidak muncul menu | Live USB → `grub-install` |
| Kernel | Kernel panic | GRUB `e`, boot kernel lama |
| Root FS | Gagal mount / | Rescue mode, fix fstab |
| systemd | Service error | Single-user, `journalctl -xb` |

**Rescue Mode:**
- Akses: GRUB → Advanced options → (recovery mode) → root
- `mount -o remount,rw /` dulu sebelum edit
- `fsck /dev/sdX` untuk repair FS
- `journalctl -xb` untuk log

---

# BAGIAN 8 — NETWORKING & WEB SERVER

## 8.0 IP Address — Konsep Dasar

**IPv4 vs IPv6:**

| Aspek | IPv4 | IPv6 |
|---|---|---|
| Panjang | 32-bit | 128-bit |
| Format | `192.168.1.1` | `2001:db8::1` |
| Jumlah | ~4.3 miliar | ~3.4×10³⁸ |

**Private IP:**

| Range | Blok |
|---|---|
| `10.0.0.0 – 10.255.255.255` | `10/8` |
| `172.16.0.0 – 172.31.255.255` | `172.16/12` |
| `192.168.0.0 – 192.168.255.255` | `192.168/16` |

**Reserved:**

| Alamat | Fungsi |
|---|---|
| `127.0.0.1` | Loopback (IPv4) |
| `::1` | Loopback (IPv6) |
| `0.0.0.0` | Unknown / DHCP |
| `255.255.255.255` | Broadcast |
| `fe80::/10` | Link-local (IPv6) |

**CIDR cheat sheet:**

| CIDR | Netmask | Host |
|---|---|---|
| `/8` | `255.0.0.0` | 16 juta |
| `/16` | `255.255.0.0` | 65.534 |
| `/24` | `255.255.255.0` | 254 |
| `/30` | `255.255.255.252` | 2 |

**Hostname:**

| Istilah | Contoh |
|---|---|
| Hostname | `academy` |
| Domain | `adinusa.id` |
| FQDN | `academy.adinusa.id` |

---

## 8.1 Hostname

**Konsep:** Nama sistem di jaringan. Ada 3 tipe: static (permanen), transient (sementara), pretty (deskriptif).

| Perintah | Fungsi |
|---|---|
| `hostname` | Lihat/set hostname (transient) |
| `hostnamectl` | Lihat info hostname lengkap |
| `hostnamectl set-hostname <nama>` | Set hostname permanent |
| `hostname -f` | FQDN |
| `cat /etc/hostname` | Hostname permanent |

**Tips:**
- `hostname` = sementara (hilang setelah reboot).
- `hostnamectl set-hostname` = permanent.
- Prompt shell tidak langsung berubah — logout-login ulang.
- Hostname valid: huruf, angka, `-`, `.` saja.

---

## 8.2 Network Device & `ip` Command

**Penamaan modern (PNIDN):**

| Skema | Contoh |
|---|---|
| Onboard index | `eno1` |
| PCI slot | `ens1` |
| PCI bus/slot | `enp0s3`, `enp2s0` |
| MAC address | `enx7837d1ea46da` |
| Legacy | `eth0`, `wlan0` |

**Kenapa PNIDN?** Nama lama (`eth0`, `eth1`) tidak deterministik — bisa tukar antar reboot. PNIDN pakai atribut hardware → stabil.

**Perintah `ip`:**

| Perintah | Fungsi |
|---|---|
| `ip a` | Lihat semua interface & IP |
| `ip addr add <ip>/<prefix> dev <if>` | Assign IP |
| `ip link set <if> up/down` | Nyalakan/matikan |
| `ip route` | Lihat routing table |
| `ip route add default via <gw>` | Tambah default gateway |
| `ip monitor` | Pantau perubahan real-time |

**`ip` vs `ifconfig`:**

| Tugas | `ip` | `ifconfig` |
|---|---|---|
| Lihat | `ip a` | `ifconfig` |
| Assign IP | `ip addr add ...` | `ifconfig ens3 10.5.5.10` |
| Up | `ip link set up` | `ifconfig up` |
| Set MTU | `ip link set mtu 1480` | `ifconfig mtu 1480` |

> `ifconfig` deprecated — pakai `ip`. Kalau perlu `ifconfig`, install `net-tools`.

**Routing table (contoh):**
```
default via 10.5.5.1 dev ens3 proto static
10.5.5.0/24 dev ens3 proto kernel scope link src 10.5.5.10
```

- `default via ...` → default route, untuk traffic umum.
- `10.5.5.0/24 dev ens3` → subnet langsung terhubung.

---

## 8.3 Name Resolution & Diagnostics

**Name resolution** = hostname → IP.

**Urutan:**
1. `/etc/hosts` (static) → dicek **dulu**.
2. DNS server (`/etc/resolv.conf`) → kalau tidak ada di hosts.

**File penting:**

| File | Fungsi |
|---|---|
| `/etc/hosts` | Static mapping (localhost, dll) |
| `/etc/resolv.conf` | DNS server (`nameserver 8.8.8.8`) |

**Tools DNS:**

| Tool | Output |
|---|---|
| `dig adinusa.id` | Detail, modern |
| `host adinusa.id` | Ringkas |
| `nslookup adinusa.id` | Lama, deprecated |

**Diagnostics:**

| Tool | Fungsi |
|---|---|
| `ping -c3 host` | Uji konektivitas (RTT, packet loss) |
| `traceroute host` | Lihat jalur paket (hop per hop) |
| `mtr host` | Ping + traceroute real-time |
| `dig domain` | Query DNS |

**Alur troubleshooting network:**
```
1. ping 8.8.8.8        → internet jalan?
2. ping google.com     → DNS jalan?
3. ip a                → interface punya IP?
4. ip route            → default gateway ada?
5. dig google.com      → DNS resolve?
6. traceroute          → di mana masalahnya?
```

---

## 8.4 Static Configuration (Netplan)

**Konsep:** Ubuntu modern pakai **Netplan** untuk konfigurasi network. File YAML di `/etc/netplan/`.

**File contoh:**
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s9:
      addresses:
      - 172.17.10.10/24
```

**Perintah:**

| Perintah | Fungsi |
|---|---|
| `sudo vim /etc/netplan/*.yaml` | Edit konfigurasi |
| `sudo netplan apply` | Terapkan |
| `sudo netplan try` | Coba + rollback otomatis kalau error |
| `ip link show` | Lihat interface |
| `ip link set <if> up` | Aktifkan interface |
| `ip addr show <if>` | Lihat IP |
| `ifconfig` | Legacy (butuh `net-tools`) |

**Struktur YAML:**

```yaml
network:
  version: 2
  ethernets:
    <interface>:
      dhcp4: true|false
      addresses: [IP/prefix]
      gateway4: IP
      nameservers:
        addresses: [DNS]
```

**Tips:**
- YAML **wajib pakai spasi**, bukan Tab.
- `netplan try` aman untuk server remote — rollback kalau error.
- Setelah apply, kadang perlu `ip link set up` manual.
- DHCP vs Static: server produksi → static, laptop → DHCP.

---

## 8.5 Apache Virtual Host

**Konsep:** Apache = web server populer. Virtual host = satu server, banyak domain.

**Struktur direktori:**
```
/etc/apache2/
├── sites-available/   ← config vhost (draft)
├── sites-enabled/     ← symlink config aktif
└── apache2.conf       ← config utama
/var/www/<domain>/     ← file web
```

| Perintah | Fungsi |
|---|---|
| `sudo apt install apache2` | Install Apache |
| `sudo systemctl status apache2` | Cek status |
| `sudo systemctl reload apache2` | Reload config |
| `sudo systemctl restart apache2` | Restart service |
| `sudo a2ensite <site>.conf` | Aktifkan site |
| `sudo a2dissite <site>.conf` | Nonaktifkan |
| `sudo a2query -s` | Site aktif |
| `sudo apache2ctl configtest` | Cek syntax config |
| `sudo apache2ctl -S` | Ringkasan vhost |
| `curl http://<domain>` | Verifikasi web |

**Contoh config vhost:**

```apache
<VirtualHost *:80>
        ServerName www.<domain>
        ServerAlias <domain>
        DocumentRoot /var/www/<domain>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

**Alur konfigurasi:**
```
1. Install apache2
2. Buat document root + index.html
3. Buat config vhost di sites-available/
4. Enable dengan a2ensite (buat symlink)
5. Reload/restart Apache
6. Tambah domain ke /etc/hosts (testing lokal)
7. Verifikasi dengan curl
```

**File penting:**

| File | Isi |
|---|---|
| `/etc/apache2/sites-available/<site>.conf` | Config vhost (draft) |
| `/etc/apache2/sites-enabled/<site>.conf` | Symlink config aktif |
| `/var/www/<domain>/index.html` | Homepage |
| `/etc/hosts` | DNS lokal |
| `/var/log/apache2/error.log` | Log error |
| `/var/log/apache2/access.log` | Log akses |

**Tips:**
- `/etc/hosts` untuk testing local tanpa DNS.
- `reload` lebih halus dari `restart`.
- `chown -R www-data:www-data /var/www/<domain>` untuk permission.
- Selalu `apache2ctl configtest` sebelum reload.
- Halaman default Apache ada di `/var/www/html/` — bisa dinonaktifkan dengan `a2dissite 000-default.conf`.

---

## 8.6 Apache HTTPS (Self-Signed)

**Konsep:** HTTPS = HTTP + SSL/TLS. Data dienkripsi. Untuk **testing**, pakai **self-signed certificate** (ditandatangani sendiri, browser akan warning). Untuk produksi, pakai **Let's Encrypt**.

**Perbedaan:**

| Jenis Cert | Cocok untuk |
|---|---|
| **Self-signed** | Testing, dev, internal |
| **Let's Encrypt** | Produksi (gratis, otomatis) |
| **Commercial CA** | Produksi enterprise (berbayar) |

**Port:** HTTP = 80, HTTPS = **443**.

**Alur setup:**

```bash
# 1. Install Apache + OpenSSL
sudo apt update
sudo apt install apache2 openssl ssl-cert -y

# 2. Enable modul SSL
sudo a2enmod ssl

# 3. Buat direktori cert
sudo mkdir /etc/apache2/ssl

# 4. Generate self-signed certificate (non-interaktif)
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/apache2/ssl/USERNAME-adinusa.id.key \
  -out /etc/apache2/ssl/USERNAME-adinusa.id.crt \
  -subj "/C=ID/ST=Jawa Barat/L=Bogor/O=Personal/CN=USERNAME-adinusa.id"

# 5. Buat vhost HTTPS
sudo vim /etc/apache2/sites-available/USERNAME-adinusa.id-ssl.conf

# 6. Enable + reload
sudo a2ensite USERNAME-adinusa.id-ssl.conf
sudo systemctl reload apache2
sudo systemctl restart apache2

# 7. Update /etc/hosts
sudo vim /etc/hosts
# Tambah: 127.0.0.1 USERNAME-adinusa.id www.USERNAME-adinusa.id

# 8. Verifikasi
curl -vk https://USERNAME-adinusa.id
```

**Penjelasan opsi `openssl req`:**

| Opsi | Fungsi |
|---|---|
| `req` | Mulai proses CSR |
| `-x509` | Output self-signed (bukan CSR) |
| `-nodes` | Private key **tanpa** passphrase |
| `-days 365` | Valid 365 hari |
| `-newkey rsa:2048` | Generate key RSA 2048-bit |
| `-keyout` | Path private key |
| `-out` | Path certificate |
| `-subj` | Subject non-interaktif |

**Contoh config vhost HTTPS:**

```apache
<IfModule mod_ssl.c>
  <VirtualHost *:443>
    ServerName USERNAME-adinusa.id
    ServerAlias www.USERNAME-adinusa.id
    DocumentRoot /var/www/USERNAME-adinusa.id

    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/USERNAME-adinusa.id.crt
    SSLCertificateKeyFile /etc/apache2/ssl/USERNAME-adinusa.id.key

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
</IfModule>
```

**File penting:**

| File | Perm | Isi |
|---|---|---|
| `/etc/apache2/ssl/<domain>.crt` | 644 | Certificate publik |
| `/etc/apache2/ssl/<domain>.key` | 600 | Private key (rahasia) |
| `/etc/apache2/sites-available/<domain>-ssl.conf` | 644 | Config vhost HTTPS |
| `/etc/apache2/ports.conf` | 644 | `Listen 443` |

**Cek certificate:**

```bash
# Info lengkap
sudo openssl x509 -in /etc/apache2/ssl/<domain>.crt -noout -text

# Tanggal expiry
sudo openssl x509 -in /etc/apache2/ssl/<domain>.crt -noout -dates

# Subject
sudo openssl x509 -in /etc/apache2/ssl/<domain>.crt -noout -subject
```

**Tips:**
- **`mod_ssl`** harus aktif (`a2enmod ssl`).
- **Private key `.key`** permission `600`, owner root — **jangan dibagikan**.
- **`-nodes`** = key tanpa passphrase (praktis server, tapi hati-hati).
- **`-subj`** = isi subject non-interaktif (`/CN=domain`).
- **Self-signed = testing**. Produksi pakai **Let's Encrypt** + Certbot.
- `curl -k` untuk ignore warning certificate.
- `curl -vk` untuk ignore + lihat detail SSL handshake.
- **`reload` vs `restart`:** reload lebih halus, restart kalau perlu.

**Redirect HTTP → HTTPS (opsional):**

Tambahkan di vhost HTTP (`<domain>.conf`):

```apache
<VirtualHost *:80>
    ServerName <domain>
    Redirect permanent / https://<domain>/
</VirtualHost>
```

---

## 8.7 phpMyAdmin + Firewalld

**Konsep:** phpMyAdmin = GUI web untuk MySQL/MariaDB. Firewalld = firewall dengan konsep zone.

**Stack LAMP:**

| Komponen | Paket |
|---|---|
| Web server | `apache2` |
| Database | `mysql-server` |
| PHP | `php libapache2-mod-php php-mysql` |
| Ekstensi PHP | `php-{mbstring,zip,gd,curl}` |
| phpMyAdmin | `phpmyadmin` |
| Firewall | `firewalld` |

**Alur install lengkap:**

```bash
# 1. Update system
sudo apt update -y && sudo apt upgrade -y

# 2. Install Apache
sudo apt install apache2 -y
sudo systemctl enable --now apache2

# 3. Install MySQL
sudo apt install mysql-server -y
sudo systemctl enable --now mysql

# 4. Install PHP + ekstensi
sudo apt install php libapache2-mod-php php-mysql -y
sudo apt install php-{mbstring,zip,gd,curl} -y

# 5. Install phpMyAdmin
#    - Pilih: apache2
#    - dbconfig-common: Yes
#    - Isi password MySQL untuk phpmyadmin
sudo apt install phpmyadmin -y

# 6. Enable mbstring
sudo phpenmod mbstring
sudo systemctl restart apache2

# 7. Buat user MySQL (kedua host — lihat tips)
sudo mysql
```

Di prompt `mysql>`:

```sql
CREATE USER 'USERNAME'@'localhost' IDENTIFIED WITH mysql_native_password BY 'adinusa';
CREATE USER 'USERNAME'@'%' IDENTIFIED WITH mysql_native_password BY 'adinusa';
GRANT ALL PRIVILEGES ON *.* TO 'USERNAME'@'localhost' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'USERNAME'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

**Vhost phpMyAdmin di port 8088:**

```bash
sudo nano /etc/apache2/sites-available/phpmyadmin.conf
```

Isi:

```apache
<VirtualHost *:8088>
    ServerAdmin admin@mydomain.local
    ServerName phpmyadmin.mydomain.local
    DocumentRoot /usr/share/phpmyadmin

    <Directory /usr/share/phpmyadmin>
        Options SymLinksIfOwnerMatch
        DirectoryIndex index.php

        <IfModule mod_php.c>
            php_admin_value upload_tmp_dir /var/lib/phpmyadmin/tmp
            php_admin_value open_basedir /usr/share/phpmyadmin/:/usr/share/doc/phpmyadmin/:/etc/phpmyadmin/:/var/lib/phpmyadmin/:/usr/share/php/:/usr/share/javascript/
        </IfModule>
    </Directory>

    <Directory /usr/share/phpmyadmin/templates>
        Require all denied
    </Directory>
    <Directory /usr/share/phpmyadmin/libraries>
        Require all denied
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/phpmyadmin_error.log
    CustomLog ${APACHE_LOG_DIR}/phpmyadmin_access.log combined
</VirtualHost>
```

**Tambahkan `Listen 8088` di `/etc/apache2/ports.conf`:**

```apache
Listen 80
Listen 8088

<IfModule ssl_module>
    Listen 443
</IfModule>
```

**Enable site & reload:**

```bash
sudo a2ensite phpmyadmin
sudo systemctl reload apache2
```

**Install & konfigurasi firewalld:**

```bash
# Install
sudo apt install firewalld -y

# Assign interface enp0s8 ke zone
sudo firewall-cmd --add-interface=enp0s8 --permanent

# Buka port 8088 di zone public
sudo firewall-cmd --add-port=8088/tcp --zone=public --permanent

# Reload
sudo firewall-cmd --reload

# Verifikasi
sudo firewall-cmd --list-all --zone=public
```

**Akses phpMyAdmin:**

```
http://<IP-VM>:8088
```

**Perintah firewalld penting:**

| Perintah | Fungsi |
|---|---|
| `firewall-cmd --state` | Cek status firewalld |
| `firewall-cmd --list-all --zone=public` | Lihat konfigurasi zone |
| `firewall-cmd --add-interface=<if> --permanent` | Assign interface ke zone |
| `firewall-cmd --add-port=<port>/tcp --zone=public --permanent` | Buka port |
| `firewall-cmd --add-service=<service> --permanent` | Buka service |
| `firewall-cmd --remove-port=<port>/tcp --permanent` | Tutup port |
| `firewall-cmd --reload` | Reload konfigurasi |

**Tips:**
- **phpMyAdmin di production = risiko** — batasi via firewall.
- **Port custom (8088)** untuk isolasi dari web utama.
- **`--permanent` + `--reload`** untuk commit perubahan firewalld.
- MySQL root di Ubuntu pakai **auth_socket** → `sudo mysql`.
- User `@'%'` **≠** user `@'localhost'` — buat keduanya untuk phpMyAdmin.
- MySQL 8.0 pakai **`caching_sha2_password`** — tidak kompatibel dengan phpMyAdmin lama. Ubah ke **`mysql_native_password`**.
- `GRANT ALL PRIVILEGES ON *.*` hanya untuk lab — production batasi per database.

---

## 8.8 Nginx Custom Port & Document Root

**Konsep:** Nginx = web server ringan, event-driven. Alternatif Apache untuk static content & reverse proxy.

**Apache vs Nginx:**

| Aspek | Apache | Nginx |
|---|---|---|
| Model | Process-based | Event-driven |
| Static | Bagus | Sangat bagus |
| Memory | Lebih besar | Lebih kecil |
| Config | vhost + `.htaccess` | Server block |

**Alur install & config:**

```bash
# 1. Install
sudo apt install nginx -y
sudo systemctl enable --now nginx

# 2. Buat document root
sudo mkdir -p /opt/lab162
sudo nano /opt/lab162/index.html
# Isi: Hello Adinusa, I am USERNAME

# 3. Edit config
sudo nano /etc/nginx/sites-available/default
```

Config:

```nginx
server {
    listen 8080 default_server;
    listen [::]:8080 default_server;

    root /opt/lab162;
    index index.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
# 4. Test & reload
sudo nginx -t
sudo systemctl reload nginx

# 5. Firewalld
sudo firewall-cmd --add-port=8080/tcp --zone=public --permanent
sudo firewall-cmd --reload

# 6. Verifikasi
curl http://localhost:8080
```

**File penting:**

| File | Isi |
|---|---|
| `/etc/nginx/nginx.conf` | Config utama |
| `/etc/nginx/sites-available/default` | Config site |
| `/etc/nginx/sites-enabled/default` | Symlink aktif |
| `/opt/lab162/index.html` | Homepage |
| `/var/log/nginx/error.log` | Log error |

**Tips:**
- **`nginx -t`** wajib sebelum reload.
- **Permission `www-data:www-data`** untuk file web.
- **Port non-standard** mengurangi attack surface.
- **Firewalld: `--permanent` + `--reload`** untuk commit.
- Nginx cocok untuk **reverse proxy** (Node.js, Python, Go).

---

## 8.9 Firewalld & Zone (Teori)

**Konsep:** Firewalld = firewall dinamis berbasis **zone**. Runtime vs Permanent config.

**Zone** (dari ketat ke longgar):
`drop` → `block` → `public` → `external` → `dmz` → `work` → `home` → `internal` → `trusted`

**Perintah penting:**

| Perintah | Fungsi |
|---|---|
| `firewall-cmd --state` | Cek status |
| `firewall-cmd --get-active-zones` | Zone aktif |
| `firewall-cmd --get-default-zone` | Zone default |
| `firewall-cmd --get-zones` | Semua zone |
| `firewall-cmd --zone=public --list-all` | Detail zone |
| `firewall-cmd --zone=public --add-service=http --permanent` | Tambah service |
| `firewall-cmd --zone=public --add-port=8080/tcp --permanent` | Buka port |
| `firewall-cmd --reload` | Apply permanent |
| `firewall-cmd --runtime-to-permanent` | Convert runtime ke permanent |

**Runtime vs Permanent:**
- Runtime → instant, hilang setelah reboot
- Permanent → butuh `--reload` untuk apply

**HTTPS:**
- Port **443**
- Service predefined `https` di firewalld
- SSL obsolete → pakai **TLS 1.2/1.3**

**Tips:**
- Jangan jalankan `iptables` + `firewalld` bersamaan.
- Edit config di `/etc/firewalld/`, bukan `/usr/lib/firewalld/`.
- Pakai zone **paling ketat** yang masih fungsional.

---

# BAGIAN 9 — PANDUAN UPDATE

## 9.1 Cara Nambah Section

**Setiap selesai 1 lab baru, cukup 3 langkah:**

1. **Tambah section** di Bagian yang sesuai. Nomor bebas, misal `8.10`, `8.11` — tidak perlu renumber yang lama.
2. **Update tabel Progress** di atas.
3. **Tambah baris** di Changelog.

**Format section minimal:**

```markdown
## X.Y Nama Topik

**Konsep:** 1 kalimat penjelasan.

| Perintah | Fungsi |
|---|---|
| `cmd` | Deskripsi |

**Tips:**
- Poin praktis

> [!] Catatan penting
```

**Kalau butuh Bagian baru:** Kalau topik benar-benar beda (Docker, Kubernetes, dll), buat Bagian 10, 11, dst. Tidak perlu reorganisasi.

---

## 9.2 Changelog

| Tanggal | Update |
|---|---|
| 2026-10-12 | Tambah 5.10 User Management Advanced (Challenge 5) |
| 2026-10-12 | Tambah 2.5 Diff comparing files (Challenge 4) |
| 2026-10-12 | Tambah 4.7 Disk + Auto-Mount (Challenge 3) |
| 2026-10-12 | Tambah 1.7 Find command (Challenge 2) |
| 2026-10-12 | Tambah 2.4 Grep text processing (Challenge 1) |
| 2026-10-12 | Fix encoding — emoji & karakter khusus |
| 2026-10-12 | Tambah 7.8 Basic Troubleshooting teori (Lab 19.0) |
| 2026-10-12 | Tambah 7.7 Troubleshooting Apache (Lab 19.1) |
| 2026-10-12 | Tambah 5.9 System Init teori (Lab 18.0) |
| 2026-10-12 | Tambah 5.8 systemd Service (Lab 18.1) |
| 2026-10-12 | Tambah 5.7 System Startup & GRUB teori (Lab 17.0) |
| 2026-10-11 | Tambah 8.9 Firewalld & Zone teori (Lab 16.0) |
| 2026-10-11 | Tambah 8.8 Nginx Custom Port (Lab 16.3) |
| 2026-10-11 | Tambah 8.7 phpMyAdmin + Firewalld (Lab 16.2) |
| 2026-10-11 | Tambah 8.6 Apache HTTPS Self-Signed (Lab 16.1) |
| 2026-10-10 | Tambah 8.5 Apache Virtual Host (Lab 15.2) |
| 2026-10-10 | Tambah 8.4 Static Configuration (Lab 15.1) |
| 2026-10-10 | Tambah 8.2 Network Device & 8.3 Name Resolution (Lab 15.0) |
| 2026-10-10 | Tambah 8.1 Hostname (Lab 14.1) |
| 2026-10-10 | Tambah 8.0 IP Address teori (Lab 14.0) |
| 2026-10-09 | Tambah 6.6 File Attributes (Lab 13.4) |
| 2026-10-09 | Tambah quiz permissions (Lab 13.5) |
| 2026-10-09 | Tambah quiz permissions & ownership (Lab 13.6) |
| 2026-10-08 | Tambah 6.3 ACL (Lab 13.3) |
| 2026-10-08 | Tambah 6.2 chmod Octal (Lab 13.2) |
| 2026-10-08 | Tambah 6.1 chmod Symbolic (Lab 13.1) |
| 2026-10-07 | Tambah Bagian 5 User & Security |
| 2026-10-06 | Rombak struktur — encoding fix, TOC ringkas |

**Cara pakai:**
- Setiap update, tambahkan **satu baris** di paling atas.
- Format: `YYYY-MM-DD | Deskripsi singkat`.

---

## Catatan Pribadi & Disclaimer

> **Catatan pribadi:** Cheatsheet ini adalah rangkuman belajar pribadi dari kursus Linux System Administration di Adinusa. Ditulis ulang dengan bahasa sendiri sebagai sarana *self-reminder* dan portofolio.
>
> **Disclaimer:** Materi asli dari modul Adinusa **tidak didistribusikan** di repositori ini. Semua catatan di sini adalah hasil pemahaman pribadi, bukan salinan modul. Jika ada kesalahan, itu murni dari pemahaman saya sendiri — bukan dari materi Adinusa.
>
> **Untuk pembaca:** Kalau kamu menemukan cheatsheet ini bermanfaat, silakan belajar darinya. Tapi untuk pemahaman yang utuh, tetap ikuti kursus aslinya di Adinusa.

---

## Referensi Lab

- **1.x:** Lab 3.1 – 3.4 (File, Directory, Editor, Pipe)
- **1.7:** Challenge 2 (Finding Files)
- **2.4:** Challenge 1 (Working with File Content)
- **2.5:** Challenge 4 (Comparing Files)
- **3.1:** Lab 4.1 – 4.2 (Resource Limits)
- **3.2:** Lab 5.1 (Process Management)
- **3.3:** Lab 7.1 (Monitoring)
- **4.1:** Lab 6.1 (APT)
- **4.2:** Lab 6.2 (External Repository)
- **4.3:** Lab 9.1 (Swap)
- **4.4:** Lab 9.2 (df/du)
- **4.5:** Lab 9.3 (Partition)
- **4.6:** Lab 10.1 (LVM)
- **4.7:** Challenge 3 (Working with Partition)
- **5.1:** Lab 8.1 – 8.3 (Inode, Hard Link, Symlink)
- **5.2:** Lab 12.1, 12.5 (User & Group)
- **5.3:** Lab 12.3 (Password Aging)
- **5.4:** Lab 12.2 (Restricted User)
- **5.5:** Lab 12.4 (SSH Key Auth)
- **5.6:** Lab 11.1 (Kernel & Boot)
- **5.7:** Lab 17.0 (System Startup & GRUB Teori)
- **5.8:** Lab 18.1 (systemd Service)
- **5.9:** Lab 18.0 (System Init Teori)
- **5.10:** Challenge 5 (Manage User Accounts)
- **6.1:** Lab 13.1 (Chmod Symbolic)
- **6.2:** Lab 13.2 (Chmod Octal)
- **6.3:** Lab 13.3 (ACL)
- **6.4:** Lab 13.6 (Chown & Chgrp)
- **6.5:** Lab 13.6 (SUID, SGID, Sticky)
- **6.6:** Lab 13.4 (File Attributes)
- **7.7:** Lab 19.1 (Troubleshooting Apache)
- **7.8:** Lab 19.0 (Basic Troubleshooting Teori)
- **8.0:** Lab 14.0 (IP Address Teori)
- **8.1:** Lab 14.1 (Hostname)
- **8.2:** Lab 15.0 (Network Device & ip)
- **8.3:** Lab 15.0 (Name Resolution & Diagnostics)
- **8.4:** Lab 15.1 (Static Configuration / Netplan)
- **8.5:** Lab 15.2 (Apache Virtual Host)
- **8.6:** Lab 16.1 (Apache HTTPS Self-Signed)
- **8.7:** Lab 16.2 (phpMyAdmin + Firewalld)
- **8.8:** Lab 16.3 (Nginx Custom Port)
- **8.9:** Lab 16.0 (Firewalld & Zone Teori)
