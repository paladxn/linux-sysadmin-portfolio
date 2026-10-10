# Linux SysAdmin Cheatsheet

**Catatan belajar Linux System Administration**
Dari kursus Adinusa (Lab 3.x – 15.x) + praktik pribadi.

> **Cara pakai:** Bagian 1–2 = fondasi. Bagian 3–4 = sistem & storage. Bagian 5–6 = security. Bagian 7 = referensi cepat. Bagian 8 = networking.

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

**Terakhir update:** 2026-10-10 (Lab 15.2 Apache Custom Domain)

---

## Daftar Isi

**Fondasi**
- [1.1 Navigasi Direktori](#11-navigasi-direktori)
- [1.2 Manajemen File & Direktori](#12-manajemen-file--direktori)
- [1.3 Melihat Isi File](#13-melihat-isi-file)
- [1.4 Membuat & Menulis File](#14-membuat--menulis-file)
- [1.5 Nano](#15-nano--editor-ramah-pemula)
- [1.6 Vim](#16-vim--editor-powerful)

**Data & Teks**
- [2.1 Pipe, Filter & Hitung](#21-pipe-filter--hitung)
- [2.2 Opsi Penting](#22-opsi-penting-yang-wajib-dihafal)
- [2.3 Kombinasi Pola Umum](#23-kombinasi-pola-yang-sering-dipakai)

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

**User & Security**
- [5.1 Inode & Links](#51-inode--links)
- [5.2 User & Group](#52-user--group-management)
- [5.3 Password Aging](#53-password-aging-chage)
- [5.4 Restricted User](#54-restricted-user--access-control)
- [5.5 SSH](#55-ssh--ssh-key-authentication)
- [5.6 Kernel & Boot](#56-kernel--boot)

**Permissions**
- [6.1 chmod Symbolic](#61-chmod--symbolic-mode)
- [6.2 chmod Octal](#62-chmod--numeric-mode-octal)
- [6.3 ACL](#63-acl-access-control-list)
- [6.4 chown & chgrp](#64-chown--chgrp)
- [6.5 Special Permissions](#65-special-permissions-suid-sgid-sticky)
- [6.6 File Attributes](#66-file-attributes-chattr--lsattr)

**Referensi Cepat**
- [7.1 Perbedaan Penting](#71-perbedaan-penting)
- [7.2 Danger Zone](#72-danger-zone--perintah-berbahaya)
- [7.3 Troubleshooting](#73-alur-troubleshooting-umum)
- [7.4 Best Practice](#74-best-practice)
- [7.5 Glosarium](#75-glosarium)
- [7.6 Kalau Panic](#76-kalau-panic)

**Networking**
- [8.0 IP Address](#80-ip-address--konsep-dasar)
- [8.1 Hostname](#81-hostname)
- [8.2 Network Device & `ip` Command](#82-network-device--ip-command)
- [8.3 Name Resolution & Diagnostics](#83-name-resolution--diagnostics)
- [8.4 Static Configuration (Netplan)](#84-static-configuration-netplan)
- [8.5 Apache Virtual Host](#85-apache-virtual-host)

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

# BAGIAN 7 — REFERENSI CEPAT

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

---

## 7.5 Glosarium

| Istilah | Arti |
|---|---|
| **ACL** | Access Control List — permission granular |
| **Apache** | Web server populer di Linux |
| **a2ensite / a2dissite** | Enable/disable virtual host |
| **APT** | Package manager Debian/Ubuntu |
| **Bash** | Shell default Linux |
| **chattr** | Change attribute |
| **chmod** | Change mode — ubah permission |
| **chown** | Change owner |
| **chgrp** | Change group |
| **CIDR** | Classless Inter-Domain Routing |
| **Cron** | Scheduler tugas berkala |
| **Daemon** | Proses background |
| **df** | Disk Free |
| **dig** | DNS lookup tool |
| **DNS** | Domain Name System |
| **DocumentRoot** | Direktori file web di Apache |
| **du** | Disk Usage |
| **ext4** | Filesystem default |
| **FQDN** | Fully Qualified Domain Name |
| **fstab** | File System Table |
| **GECOS** | Comment di `/etc/passwd` |
| **GID** | Group ID |
| **GRUB** | Bootloader |
| **Hard link** | Nama tambahan ke inode sama |
| **Hostname** | Nama perangkat di jaringan |
| **ifconfig** | Legacy network tool (deprecated) |
| **Inode** | Metadata file (bukan nama) |
| **ip** | Modern network tool |
| **IPv4 / IPv6** | Versi IP address |
| **Kernel** | Inti OS |
| **Load average** | Antrean proses |
| **LVM** | Logical Volume Manager |
| **mtr** | Ping + traceroute real-time |
| **Netplan** | Konfigurasi network Ubuntu (YAML) |
| **NetworkManager** | Service network modern |
| **OOM** | Out of Memory |
| **PAM** | Pluggable Auth Modules |
| **Permission** | Hak akses |
| **ping** | Uji konektivitas |
| **Pipe** | `\|` — sambung output |
| **PNIDN** | Predictable Network Interface Device Names |
| **Private IP** | IP internal (10/8, 172.16/12, 192.168/16) |
| **PV/VG/LV** | Physical/Volume/Logical (LVM) |
| **rbash** | Restricted bash |
| **reload / restart** | Baca ulang vs matikan-nyalakan service |
| **Root** | Superuser |
| **Routing** | Pemilihan jalur paket |
| **SELinux/AppArmor** | Mandatory access control |
| **SGID** | Set Group ID |
| **SIGTERM/SIGKILL** | Sinyal 15 / 9 |
| **SSH** | Secure Shell |
| **SUID** | Set User ID |
| **Sticky bit** | Proteksi hapus di direktori |
| **sudo** | Superuser Do |
| **Swap** | Memori cadangan |
| **Symlink** | Shortcut |
| **Systemd** | Init system |
| **traceroute** | Lihat jalur paket |
| **UID** | User ID |
| **ulimit** | User limit |
| **Umask** | Filter permission default |
| **Vim** | Editor powerful |
| **VirtualHost** | Konfigurasi domain di Apache |
| **www-data** | User default Apache |
| **Zombie** | Proses selesai belum di-reap |

---

## 7.6 Kalau Panic

| Situasi | Langkah |
|---|---|
| Tidak boot | Rescue mode dari GRUB |
| Lupa password root | Rescue → `passwd` |
| Disk penuh | `df -h` → `du -sh /* \| sort -h` |
| Fork error | `ulimit -u` |
| Command hang | `CTRL + C` |
| Vim stuck | `Esc` → `:q!` |
| Nano stuck | `CTRL + X` → `N` |
| Terminal kacau | `reset` atau `stty sane` |
| Lupa perintah | `man <cmd>` atau `--help` |
| Network down | `ping 8.8.8.8` → cek `ip a` → `ip route` |
| Apache error | `apache2ctl configtest` → cek `/var/log/apache2/error.log` |

> **Tips:** Kalau panik — jangan ketik apapun dulu. Tarik napas, baca error, baru cari solusi.

---

# BAGIAN 8 — NETWORKING

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

# BAGIAN 9 — PANDUAN UPDATE

## 9.1 Cara Nambah Section

**Setiap selesai 1 lab baru, cukup 3 langkah:**

1. **Tambah section** di Bagian yang sesuai. Nomor bebas, misal `8.6`, `8.7` — tidak perlu renumber yang lama.
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
- **2.x:** Lab 3.3, 3.4 (Pipe, Wildcard)
- **3.1:** Lab 4.1 – 4.2 (Resource Limits)
- **3.2:** Lab 5.1 (Process Management)
- **3.3:** Lab 7.1 (Monitoring)
- **4.1:** Lab 6.1 (APT)
- **4.2:** Lab 6.2 (External Repository)
- **4.3:** Lab 9.1 (Swap)
- **4.4:** Lab 9.2 (df/du)
- **4.5:** Lab 9.3 (Partition)
- **4.6:** Lab 10.1 (LVM)
- **5.1:** Lab 8.1 – 8.3 (Inode, Hard Link, Symlink)
- **5.2:** Lab 12.1, 12.5 (User & Group)
- **5.3:** Lab 12.3 (Password Aging)
- **5.4:** Lab 12.2 (Restricted User)
- **5.5:** Lab 12.4 (SSH Key Auth)
- **5.6:** Lab 11.1 (Kernel & Boot)
- **6.1:** Lab 13.1 (Chmod Symbolic)
- **6.2:** Lab 13.2 (Chmod Octal)
- **6.3:** Lab 13.3 (ACL)
- **6.4:** Lab 13.6 (Chown & Chgrp)
- **6.5:** Lab 13.6 (SUID, SGID, Sticky)
- **6.6:** Lab 13.4 (File Attributes)
- **8.0:** Lab 14.0 (IP Address Teori)
- **8.1:** Lab 14.1 (Hostname)
- **8.2:** Lab 15.0 (Network Device & ip)
- **8.3:** Lab 15.0 (Name Resolution & Diagnostics)
- **8.4:** Lab 15.1 (Static Configuration / Netplan)
- **8.5:** Lab 15.2 (Apache Virtual Host)
