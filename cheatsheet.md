# 🐧 Linux SysAdmin Cheatsheet

**Referensi cepat untuk belajar Linux System Administration**
Kumpulan perintah dari Lab 3.1 – 12.5 (kursus Adinusa) + pengalaman praktik.

> 📖 **Cara pakai cheatsheet ini:**
> - Bagian **1–2** = fondasi. Wajib dikuasai dulu.
> - Bagian **3–4** = sistem & storage. Dipakai sehari-hari.
> - Bagian **5** = user & security. Untuk produksi.
> - Bagian **6** = referensi cepat.

> ⚠️ **Penanda bahaya:**
> - ✅ = aman
> - ⚠️ = hati-hati (bisa mengubah data)
> - 🚨 = berbahaya (bisa menghapus sistem / mengunci akses)

---

## 📑 Daftar Isi

**Bagian 1 — Fondasi**
- [1.1 Navigasi Direktori](#11-navigasi-direktori)
- [1.2 Manajemen File & Direktori](#12-manajemen-file--direktori)
- [1.3 Melihat Isi File & Direktori](#13-melihat-isi-file--direktori)
- [1.4 Membuat & Menulis File](#14-membuat--menulis-file)
- [1.5 Nano — Editor Ramah Pemula](#15-nano--editor-ramah-pemula)
- [1.6 Vim — Editor Powerful](#16-vim--editor-powerful)

**Bagian 2 — Manipulasi Data**
- [2.1 Pipe, Filter & Hitung](#21-pipe-filter--hitung)
- [2.2 Opsi Penting yang Wajib Dihafal](#22-opsi-penting-yang-wajib-dihafal)
- [2.3 Kombinasi Pola yang Sering Dipakai](#23-kombinasi-pola-yang-sering-dipakai)

**Bagian 3 — Sistem & Proses**
- [3.1 Resource Limits (`ulimit`)](#31-resource-limits-ulimit)
- [3.2 Process Management](#32-process-management)
- [3.3 Monitoring CPU & Load](#33-monitoring-cpu--load)

**Bagian 4 — Package & Storage**
- [4.1 Package Management (APT)](#41-package-management-apt)
- [4.2 External Repository & Version Pinning](#42-external-repository--version-pinning)
- [4.3 Swap File](#43-swap-file)
- [4.4 Disk Usage (`df` & `du`)](#44-disk-usage-df--du)
- [4.5 Partition Management](#45-partition-management)
- [4.6 LVM (Logical Volume Manager)](#46-lvm-logical-volume-manager)

**Bagian 5 — User & Security**
- [5.1 Inode & Links](#51-inode--links)
- [5.2 User & Group Management](#52-user--group-management)
- [5.3 Password Aging (`chage`)](#53-password-aging-chage)
- [5.4 Restricted User & Access Control](#54-restricted-user--access-control)
- [5.5 SSH & SSH Key Authentication](#55-ssh--ssh-key-authentication)
- [5.6 Kernel & Boot](#56-kernel--boot)

**Bagian 6 — Referensi Cepat**
- [6.1 Perbedaan Penting](#61-perbedaan-penting)
- [6.2 Danger Zone — Perintah Berbahaya](#62-danger-zone--perintah-berbahaya)
- [6.3 Alur Troubleshooting Umum](#63-alur-troubleshooting-umum)
- [6.4 Best Practice untuk Pemula](#64-best-practice-untuk-pemula)
- [📖 Glosarium](#-glosarium)
- [🚀 Kalau Baru Mulai](#-kalau-baru-mulai)
- [⚠️ Command yang Sering Typo](#️-command-yang-sering-typo)
- [🆘 Kalau Panic](#-kalau-panic)

---

# 📚 BAGIAN 1 — FONDASI

## 1.1 Navigasi Direktori

**Kapan dipakai:** Setiap saat. Ini GPS-nya Linux.

| Perintah | Fungsi | Analogi |
|---|---|---|
| `pwd` | Tampilkan posisi saat ini | "Saya di mana?" |
| `cd folder` | Masuk ke folder | "Jalan ke folder" |
| `cd ..` | Naik satu level | "Kembali ke parent" |
| `cd ~` | Ke home directory | "Pulang ke rumah" |
| `cd -` | Ke direktori sebelumnya | "Balik ke tempat tadi" |

```bash
$ pwd
/home/student
$ cd lab5
$ pwd
/home/student/lab5
```

---

## 1.2 Manajemen File & Direktori

| Perintah | Fungsi | Catatan |
|---|---|---|
| `mkdir nama` | Buat direktori | — |
| `mkdir -p a/b/c` | Buat direktori + parent sekaligus | ✅ hemat waktu |
| `touch file.txt` | Buat file kosong | Juga update timestamp |
| `rm file.txt` | Hapus file | ⚠️ langsung hapus |
| `rmdir folder` | Hapus direktori **kosong** | ✅ aman |
| `rm -r folder` | Hapus direktori + isinya | 🚨 tidak ada undo |
| `rm -ri folder` | Hapus dengan konfirmasi | ✅ **selalu pakai ini** |

> 💡 **Tips senior:** Biasakan `rm -ri` daripada `rm -r`. Konfirmasi per file memang menyebalkan, tapi menyelamatkan dari bencana.

---

## 1.3 Melihat Isi File & Direktori

| Perintah | Fungsi | Kapan dipakai |
|---|---|---|
| `ls` | List isi direktori | Cepat, overview |
| `ls -a` | Tampilkan hidden file | Cek `.bashrc`, `.ssh` |
| `ls -l` | Format panjang | Lihat permission, owner, size |
| `ls -lah` | Detail + hidden + human-readable | **Paling sering dipakai** |
| `ls -li` | Detail + inode number | Debug hard link |
| `cat file` | Tampilkan seluruh isi | File pendek |
| `less file` | Tampilkan per halaman | File panjang (log) |
| `ls --help` | Bantuan | Selalu cek kalau ragu |

> 💡 **Tips:** Kuasai `ls -lah` — ini perintah yang paling sering dipakai sysadmin.

---

## 1.4 Membuat & Menulis File

| Perintah | Fungsi |
|---|---|
| `echo "teks" > file` | Tulis ke file (**overwrite** isi lama) |
| `echo "teks" >> file` | Tambah ke akhir (**append**) |
| `cat file1 file2 > file3` | Gabungkan dua file |
| `nano file` | Edit dengan Nano (ramah pemula) |
| `vim file` | Edit dengan Vim (powerful) |

> ⚠️ **Bedakan `>` dan `>>`:**
> - `>` → **timpa** isi lama. Kalau salah, data hilang.
> - `>>` → **tambah** di akhir. Lebih aman.

---

## 1.5 Nano — Editor Ramah Pemula

**Kapan dipakai:** Edit cepat, config file. Semua shortcut tampil di bawah layar.

| Shortcut | Fungsi |
|---|---|
| `CTRL + O` lalu `Enter` | Simpan |
| `CTRL + X` | Keluar |
| `CTRL + W` | Cari teks |
| `CTRL + ^` (`CTRL + Shift + 6`) | Set mark (mulai seleksi) |
| `CTRL + K` | Cut |
| `CTRL + U` | Paste |

---

## 1.6 Vim — Editor Powerful

**Kapan dipakai:** Edit file besar, butuh cepat, atau di server minimal.

**Konsep kunci:** Vim punya **mode** — ini yang bikin pemula bingung.

| Mode | Cara Masuk | Fungsi |
|---|---|---|
| Normal | `Esc` | Navigasi & perintah |
| Insert | `i` | Mengetik teks |
| Command | `:` | Save/quit/search |

**Perintah penting:**

| Perintah | Fungsi |
|---|---|
| `i` | Masuk insert mode |
| `Esc` | Keluar dari insert mode |
| `:wq` | Save & exit |
| `:q!` | Exit tanpa save |
| `/kata` | Cari (`n` next, `N` previous) |
| `yy` / `dd` / `p` | Copy / cut / paste baris |
| `G` / `gg` | Ke baris terakhir / pertama |
| `o` | Buka baris baru di bawah |

> 💡 **Tips pemula:** Kalau bingung di Vim, tekan `Esc` dulu. Itu "tombol panik" untuk kembali ke normal mode.

---

# 📚 BAGIAN 2 — MANIPULASI DATA

## 2.1 Pipe, Filter & Hitung

**Konsep:** Pipe (`|`) menghubungkan output satu perintah ke input perintah lain. Ini filosofi Unix: *"do one thing, do it well"*.

| Perintah | Fungsi |
|---|---|
| `ls -l \| less` | Lihat daftar dengan scroll |
| `cat file \| grep kata` | Filter baris yang mengandung `kata` |
| `ls \| grep -c pola` | Hitung file yang cocok dengan pola |
| `ls \| wc -l` | Hitung jumlah item |
| `wc -l` / `-w` / `-c` | Hitung baris / kata / byte |

**Contoh nyata:**

```bash
# Cari semua file .log yang mengandung "ERROR"
$ cat /var/log/*.log | grep ERROR

# Hitung berapa user dengan shell bash
$ cat /etc/passwd | grep "/bin/bash" | wc -l
```

> 💡 **Analogi:** Pipe itu seperti pipa air — output dari satu keran langsung masuk ke keran berikutnya.

---

## 2.2 Opsi Penting yang Wajib Dihafal

| Opsi | Arti | Berlaku di |
|---|---|---|
| `-a` | All (termasuk hidden) | `ls` |
| `-l` | Long format | `ls` |
| `-h` | Human-readable (K, M, G) | `ls`, `df`, `du` |
| `-r` | Recursive | `cp`, `rm` |
| `-i` | Interactive (konfirmasi) | `rm`, `cp` |
| `-p` | Buat parent directory | `mkdir` |
| `-c` | Count | `grep` |
| `-v` | Invert match | `grep` |
| `-n` | Numeric / no-newline | `sort`, `echo` |

---

## 2.3 Kombinasi Pola yang Sering Dipakai

```bash
# Hitung file dengan pola tertentu
$ ls | grep -c client

# Lihat isi file panjang dengan scroll
$ cat /etc/passwd | less

# Gabungkan dua file
$ cat notes.txt data.txt > combined.txt

# Tulis hasil ke file dengan template
$ echo "client = $(ls | grep -c client)" > answer.txt

# Buang pesan error (redirect stderr ke /dev/null)
$ ls pola* 2>/dev/null

# Cari 20 file/direktori terbesar di home
$ du -ah ~ | sort -h | tail -20
```

---

# 📚 BAGIAN 3 — SISTEM & PROSES

## 3.1 Resource Limits (`ulimit`)

**Konsep:** Linux membatasi resource per user — jumlah file terbuka, proses, memory. Berguna untuk mencegah satu user menghabiskan resource sistem.

| Perintah | Fungsi |
|---|---|
| `ulimit -a` | Tampilkan semua limit |
| `ulimit -n` | Limit open files |
| `ulimit -u` | Limit proses per user |
| `ulimit -c` | Limit core file size |
| `ulimit -f` | Limit ukuran file |

**Membuat limit permanen di `/etc/security/limits.conf`:**

```
<domain>   <type>   <item>   <value>
```

| Field | Nilai |
|---|---|
| `<domain>` | Username, `@groupname`, atau `*` |
| `<type>` | `soft` (aktif) atau `hard` (batas atas) |
| `<item>` | `nproc`, `nofile`, `core`, dll |

**Contoh:**

```
@student   hard   nproc   100
student    soft   nofile  2000
student    hard   nofile  3000
```

**Verifikasi:**

```bash
$ ulimit -n     # soft limit
$ ulimit -Hn    # hard limit
$ ulimit -u     # max processes
```

> ⚠️ **Penting:** Perubahan di `limits.conf` hanya berlaku setelah **logout & login ulang**.

> 💡 **Tips:** Kalau `ulimit -u` menunjukkan angka kecil (misal 20) dan kamu tidak bisa fork proses baru — itu sebabnya. Cek `ulimit -u` dulu saat ada error *"fork: Resource temporarily unavailable"*.

---

## 3.2 Process Management

| Perintah | Fungsi |
|---|---|
| `command &` | Jalankan di background |
| `ps aux \| grep nama` | Cari proses |
| `pgrep -af pola` | Dapatkan PID |
| `kill PID` | Kirim `SIGTERM` (sopan) |
| `kill -9 PID` | Paksa (`SIGKILL`) |
| `killall nama` | Hentikan semua dengan nama sama |
| `pkill -f pola` | Hentikan berdasarkan pola |
| `jobs` | Daftar job di shell |
| `fg` / `bg` | Pindah job ke foreground/background |

**Sinyal penting:**

| Sinyal | Nomor | Fungsi |
|---|---|---|
| `SIGTERM` | 15 | Berhenti sopan (default `kill`) |
| `SIGKILL` | 9 | Paksa berhenti (tidak bisa ditolak) |
| `SIGHUP` | 1 | Reload konfigurasi |
| `SIGINT` | 2 | Interrupt (`CTRL + C`) |

> 💡 **Tips:** Selalu coba `kill PID` dulu. Baru `kill -9` kalau proses tidak merespons.

**Trik hindari baris `grep` muncul di output:**

```bash
$ ps aux | grep [k]illing     # bracket trick
$ pgrep -af killing           # atau pakai pgrep
```

---

## 3.3 Monitoring CPU & Load

| Perintah | Fungsi |
|---|---|
| `lscpu` | Info CPU |
| `nproc` | Jumlah logical CPU |
| `top` | Monitor real-time |
| `top -bn1` | Mode batch 1 iterasi |

**Tombol interaktif di `top`:**

| Tombol | Fungsi |
|---|---|
| `l` `t` `m` | Toggle header (load, tasks, memory) |
| `P` / `M` | Sort by %CPU / %MEM |
| `1` | Tampilkan per-CPU |
| `k` | Kill proses |
| `q` | Keluar |

**Load average:**

- Format: `load average: 1min, 5min, 15min`
- **Aturan emas:** Load ≈ jumlah CPU → CPU 100% penuh
- Contoh: 1 CPU dengan load 1.0 = penuh. 4 CPU dengan load 4.0 = penuh.

> 💡 **Analogi:** Load average itu seperti panjang antrean di kasir. Kalau antreannya lebih panjang dari jumlah kasir, ada yang menunggu.

---

# 📚 BAGIAN 4 — PACKAGE & STORAGE

## 4.1 Package Management (APT)

**Konsep:** APT = cara install/uninstall software di Debian/Ubuntu.

| Perintah | Fungsi |
|---|---|
| `sudo apt update` | Refresh daftar paket (**wajib sebelum install**) |
| `sudo apt upgrade -y` | Upgrade paket terinstal |
| `apt search <kata>` | Cari paket |
| `apt show <paket>` | Detail paket |
| `sudo apt install <paket> -y` | Instal |
| `apt list --installed` | Daftar paket terinstal |
| `sudo apt remove <paket>` | Hapus (config tersisa) |
| `sudo apt purge <paket>` | Hapus + config |
| `sudo apt autoremove` | Bersihkan dependensi tak terpakai |

> ⚠️ **Bedakan `remove` dan `purge`:** `remove` menyisakan config. `purge` menghapus semua jejak.

---

## 4.2 External Repository & Version Pinning

**Kapan dipakai:** Install versi paket tertentu yang tidak ada di repo default.

| Perintah | Fungsi |
|---|---|
| `curl -LsS <url> -o script` | Download script |
| `sudo tee /etc/apt/sources.list.d/<name>.list` | Buat repo manual |
| `apt-cache policy <paket>` | Cek versi tersedia |
| `sudo apt install <paket>=<versi>` | Install versi spesifik |
| `apt-mark hold <paket>` | Kunci versi |
| `lsb_release -cs` | Cek codename distro |

**Format file repo di `/etc/apt/sources.list.d/`:**

```
deb [arch=amd64,arm64] https://repo.example.com/ubuntu noble main
```

> ⚠️ **Hati-hati:** Repo eksternal bisa bentrok dengan repo default. Selalu cek `apt-cache policy` setelah menambah repo.

---

## 4.3 Swap File

**Konsep:** Swap = "memori cadangan" di disk saat RAM penuh. Bukan pengganti RAM, tapi penyelamat dari OOM (Out of Memory).

| Perintah | Fungsi |
|---|---|
| `fallocate -l 2G /swapfile` | Buat file swap 2 GB |
| `chmod 600 /swapfile` | Set permission ketat |
| `mkswap /swapfile` | Format sebagai swap |
| `swapon /swapfile` | Aktifkan |
| `swapoff /swapfile` | Nonaktifkan |
| `swapon --show` | Daftar swap aktif |
| `free -h` | Total memori + swap |

**Format `/etc/fstab` untuk swap:**

```
/swapfile swap swap defaults 0 0
```

**Ukuran swap yang disarankan:**

| RAM | Swap |
|---|---|
| ≤ 2 GB | 2× RAM |
| 2–8 GB | 1× RAM |
| > 8 GB | 0.5× RAM, min 4 GB |

---

## 4.4 Disk Usage (`df` & `du`)

**Perbedaan kunci:**

| Perintah | Menjawab pertanyaan | Sumber data |
|---|---|---|
| `df` | "Berapa sisa ruang disk?" | Superblock (cepat) |
| `du` | "Folder mana yang besar?" | Setiap file (lambat) |

| Perintah | Fungsi |
|---|---|
| `df -h` | Penggunaan filesystem |
| `df -Th` | + tipe filesystem |
| `df -i` | Penggunaan inode |
| `du -sh .` | Total ukuran direktori ini |
| `du -h dir/*` | Ukuran tiap file |
| `du --max-depth=1` | Ukuran per subdirektori (1 level) |
| `du -ah \| sort -h \| tail -20` | Top 20 file/direktori terbesar |

> 💡 **Tips:** Server tiba-tiba penuh? Jalankan `df -h` untuk cari filesystem yang penuh, lalu `du -sh /* \| sort -h` untuk cari pelakunya.

---

## 4.5 Partition Management

| Perintah | Fungsi |
|---|---|
| `lsblk` / `lsblk -f` | Lihat disk & partisi |
| `sudo parted /dev/sdX` | Tool partisi GPT/MBR |
| `(parted) mklabel gpt` | Buat table GPT |
| `(parted) mkpart` | Buat partisi |
| `(parted) set N lvm on` | Set flag LVM |
| `sudo fdisk /dev/sdX` | Tool partisi interaktif |
| `(fdisk) n / d / w / q` | New / delete / write / quit |
| `mkfs -t ext4 -L LABEL /dev/sdX1` | Format + label |
| `blkid /dev/sdX1` | Lihat label + UUID |
| `mount LABEL=NAME /mnt/point` | Mount dengan label |
| `mount -a` | Mount semua dari fstab |
| `partprobe /dev/sdX` | Reload partition table |

**Format `/etc/fstab` yang aman:**

```
LABEL=NAMA  /mnt/point  ext4  defaults,nofail  0  2
```

- `LABEL=` → identifikasi device yang konsisten (nama `/dev/sdX` bisa berubah)
- `nofail` → sistem tetap boot meski partisi tidak ada

> 🚨 **PENTING:** Sebelum reboot setelah edit `/etc/fstab`, **selalu test** dengan `mount -a`. Kalau fstab rusak, sistem bisa tidak boot.

---

## 4.6 LVM (Logical Volume Manager)

**Konsep:** LVM = "partisi virtual" yang bisa di-resize tanpa reboot.

**Alur:**

```
partisi → pvcreate → vgcreate → lvcreate → mkfs → mount → fstab
```

| Perintah | Fungsi |
|---|---|
| `pvcreate /dev/sdb1` | Buat Physical Volume |
| `vgcreate <vg> /dev/sdb1 /dev/sdb2` | Buat Volume Group |
| `lvcreate -n <lv> -L 400M <vg>` | Buat Logical Volume |
| `mkfs -t ext4 /dev/<vg>/<lv>` | Format LV |
| `pvs` / `vgs` / `lvs` | Info ringkas |
| `pvdisplay` / `vgdisplay` / `lvdisplay` | Info detail |
| `vgextend <vg> /dev/sdc1` | Tambah PV ke VG |
| `lvextend -L +200M /dev/<vg>/<lv>` | Perbesar LV |
| `resize2fs /dev/<vg>/<lv>` | Resize filesystem ext4 |
| `lvreduce -L 200M /dev/<vg>/<lv>` | Perkecil LV |
| `lvremove` / `vgremove` / `pvremove` | Hapus (urutan: LV → VG → PV) |
| `lvcreate -s -n snap -L 100M /dev/<vg>/<lv>` | Snapshot |

**Aturan penting:**

- Flag `lvm on` wajib di partisi (`parted set N lvm on`)
- `udevadm settle` sebelum `pvcreate`
- `systemctl daemon-reload` setelah edit fstab
- **Resize:**
  - Perbesar: `lvextend` → `resize2fs`
  - Perkecil: `resize2fs` → `lvreduce` (filesystem dulu!)

---

# 📚 BAGIAN 5 — USER & SECURITY

## 5.1 Inode & Links

**Konsep inode:** Inode = "KTP" file. Setiap file punya inode unik yang menyimpan metadata (permission, owner, size, pointer ke data). **Nama file hanya label** yang menunjuk ke inode.

| Perintah | Fungsi |
|---|---|
| `ls -li file` | Lihat inode + link count |
| `ln source target` | Buat hard link |
| `ln -s source target` | Buat symbolic link |
| `readlink link` | Lihat target symlink |
| `file link` | Identifikasi tipe file |
| `test -L link` | Cek apakah symlink |
| `stat file` | Info lengkap |
| `df -h /path` | Filesystem dari path |

**Perbedaan Hard vs Symbolic Link:**

| Aspek | Hard Link | Symbolic Link |
|---|---|---|
| Menunjuk ke | Inode | Path |
| Inode number | Sama | Berbeda |
| Lintas filesystem | ❌ | ✅ |
| Link ke direktori | ❌ | ✅ |
| Target dihapus | Data tetap ada | Link rusak (dangling) |
| Tanda di `ls -l` | File biasa | `l` + `->` |

**Symlink ke direktori:**

```bash
$ ln -s /tmp ~/tmplink       # symlink ke /tmp
$ rm ~/tmplink                # ✅ aman
```

> 🚨 **JANGAN** pakai trailing slash saat hapus symlink direktori:
> ```bash
> $ rm -r ~/tmplink/    # ❌ ini hapus isi /tmp!
> ```

---

## 5.2 User & Group Management

| Perintah | Fungsi |
|---|---|
| `useradd <user>` | Buat user |
| `useradd -m -s /bin/bash <user>` | Buat + home + shell |
| `userdel -r <user>` | Hapus + home |
| `usermod -c "comment" <user>` | Set comment |
| `usermod -aG <group> <user>` | Tambah ke supplementary group |
| `usermod -L` / `-U` | Lock / unlock user |
| `passwd <user>` | Set password |
| `passwd -S <user>` | Cek status password |
| `groupadd -g <gid> <group>` | Buat group |
| `id <user>` / `groups <user>` | Info group |
| `su - <user>` | Switch user |
| `sudo -l -U <user>` | Cek sudo privilege |
| `visudo -c` | Validasi sudoers |

**File penting:**

| File | Permission | Isi |
|---|---|---|
| `/etc/passwd` | 644 | Data user |
| `/etc/shadow` | 400 | Hash password & aging |
| `/etc/group` | 644 | Data group |
| `/etc/gshadow` | 400 | Password group |
| `/etc/skel/` | — | Template home user baru |

**Format sudoers:**

```
%group  ALL=(ALL)  NOPASSWD:ALL
user    ALL=(root) NOPASSWD:/usr/bin/systemctl restart nginx
```

**Aturan emas:**

- **Selalu `usermod -aG`** (append), jangan `-G` — nanti group lama hilang!
- User harus **logout & login ulang** setelah diubah group-nya.
- Permission file di `/etc/sudoers.d/` harus `0440`, tanpa titik di nama.
- **Hindari `NOPASSWD:ALL`** di production — batasi perintah spesifik.

> 🚨 **JANGAN pernah edit `/etc/passwd`, `/etc/shadow`, atau `/etc/group` secara langsung.** Gunakan `useradd`, `usermod`, `passwd`, `chage`. Salah edit bisa mengunci kamu dari sistem.

---

## 5.3 Password Aging (`chage`)

**Konsep:** Password aging memaksa user ganti password secara berkala.

| Perintah | Fungsi |
|---|---|
| `chage -l <user>` | Lihat info aging |
| `chage -M 30 -m 7 -W 5 <user>` | Set max/min/warning |
| `chage -d 0 <user>` | Paksa ganti saat login berikutnya |
| `chage -E YYYY-MM-DD <user>` | Set account expiration |
| `chage -I N <user>` | Set inactive days setelah expired |

**Default system-wide di `/etc/login.defs`:**

```
PASS_MAX_DAYS   90
PASS_MIN_DAYS   1
PASS_WARN_AGE   7
```

> 💡 Hanya berlaku untuk **user baru**. User lama harus diubah manual dengan `chage`.

**Verifikasi sebelum grading:**

```bash
$ sudo chage -l <user>       # cek aging
$ sudo passwd -S <user>      # cek status (harus P)
$ sudo grep <user> /etc/shadow  # field ke-3 harus 0 kalau force change
```

---

## 5.4 Restricted User & Access Control

**Kapan dipakai:** Guest account, akun demo, akun untuk otomasi terbatas.

**Langkah membuat restricted user:**

```bash
# 1. Buat user dengan shell normal dulu
useradd -m -s /bin/bash <username>
passwd <username>

# 2. Ganti ke restricted shell
usermod -s /bin/rbash <username>

# 3. Buat direktori command yang diizinkan
mkdir -p /home/<username>/bin
chown <username>:<username> /home/<username>/bin
chmod 755 /home/<username>/bin

# 4. Batasi PATH
echo 'export PATH=$HOME/bin' >> /home/<username>/.bash_profile

# 5. Symlink command yang diizinkan
ln -s /bin/ls /home/<username>/bin/ls
ln -s /bin/cat /home/<username>/bin/cat
ln -s /usr/bin/sudo /home/<username>/bin/sudo

# 6. Buat group & tambahkan user
groupadd -f limited<username>
usermod -aG limited<username> <username>

# 7. Sudoers terbatas
visudo -f /etc/sudoers.d/limited<username>
# Isi: %limited<username> ALL=(ALL) NOPASSWD: /bin/ls, /bin/cat
chmod 440 /etc/sudoers.d/limited<username>

# 8. Batasi login SSH
echo "-:limited<username>:ALL EXCEPT LOCAL" >> /etc/security/access.conf
```

**Batasan `rbash`:**

- ❌ `cd` ke direktori lain
- ❌ Set/modifikasi env var (PATH, SHELL, ENV)
- ❌ Command dengan `/` di path
- ❌ Redirect input/output

> ⚠️ **Penting:** `rbash` **BUKAN security tool**. Mudah di-bypass oleh user berpengalaman. Untuk keamanan sejati, pakai SELinux, AppArmor, atau container.

---

## 5.5 SSH & SSH Key Authentication

**Konsep:** SSH = cara aman akses server remote. Key-based auth = login tanpa password, lebih aman dari password.

| Perintah | Fungsi |
|---|---|
| `ssh user@host` | Login remote |
| `ssh user@host "command"` | Jalankan perintah remote |
| `scp file user@host:/path` | Copy file |
| `scp -r dir user@host:/path` | Copy direktori |
| `ssh-keygen -t rsa -b 4096 -N ""` | Generate key pair |
| `ssh-copy-id -i ~/.ssh/id_rsa.pub -p PORT user@host` | Salin public key |
| `ssh -p PORT -o BatchMode=yes user@host "cmd"` | Verifikasi passwordless |
| `ssh-keyscan -p PORT host >> ~/.ssh/known_hosts` | Ambil host key |
| `ssh-keygen -R "[host]:PORT"` | Hapus host key lama |

**File SSH & permission:**

| File | Permission | Fungsi |
|---|---|---|
| `~/.ssh` | `700` | Direktori SSH |
| `~/.ssh/id_rsa` | `600` | Private key |
| `~/.ssh/id_rsa.pub` | `644` | Public key |
| `~/.ssh/authorized_keys` | `600` | Key yang diizinkan |
| `~/.ssh/known_hosts` | `644` | Host key server |

> 💡 **Tips:** Private key **tidak pernah** dikirim ke server. Hanya public key yang disalin. Server pakai public key untuk "menantang" — hanya private key yang bisa jawab.

---

## 5.6 Kernel & Boot

| Perintah / Konsep | Fungsi |
|---|---|
| `cat /proc/cmdline` | Lihat boot parameters aktif |
| `man bootparam` | Dokumentasi parameter |
| `lsmod` | Daftar kernel module |
| `modprobe <module>` | Load/unload module |
| `modinfo <module>` | Info module |
| `mount -o remount,rw /` | Remount root read-write |

**Kernel Boot Parameters Umum:**

| Parameter | Fungsi |
|---|---|
| `root=UUID=...` | Filesystem root |
| `ro` | Mount root read-only dulu |
| `quiet` | Kurangi output boot |
| `nomodeset` | Troubleshooting GPU |
| `noapic` | Nonaktifkan APIC |
| `crashkernel=` | Reserve memori crash dump |
| `resume=UUID=...` | Resume dari hibernasi |

**Rescue Mode (Ubuntu):**

- Akses: GRUB → Advanced options → **(recovery mode)**
- Opsi: `resume`, `clean`, `dpkg`, `fsck`, `grub`, `network`, `root`
- Dari root shell:
  ```bash
  mount -o remount,rw /
  passwd <user>
  update-grub
  grub-install /dev/sda
  ```

---

# 📚 BAGIAN 6 — REFERENSI CEPAT

## 6.1 Perbedaan Penting

| Konsep | Penjelasan |
|---|---|
| `>` vs `>>` | `>` timpa isi, `>>` tambah di akhir |
| `rmdir` vs `rm -r` | `rmdir` hanya direktori kosong, `rm -r` termasuk isi |
| `vi` vs `vim` | `vi` original, `vim` enhanced (banyak distro me-link `vi` → `vim`) |
| Nano vs Vim | Nano ramah pemula, Vim powerful (butuh hafal mode) |
| `soft` vs `hard` limit | `soft` aktif, `hard` batas atas |
| `remove` vs `purge` | `remove` sisa config, `purge` bersih total |
| `df` vs `du` | `df` = filesystem, `du` = direktori |
| `usermod -G` vs `-aG` | `-G` overwrite, `-aG` append |
| Hard link vs symlink | Hard = inode sama, symlink = path |

---

## 6.2 Danger Zone — Perintah Berbahaya

| Perintah | Bahaya |
|---|---|
| 🚨 `rm -rf /` | Hapus seluruh sistem |
| 🚨 `rm -rf ~/` | Hapus seluruh home |
| 🚨 `rm -r symlink/` | Hapus isi target symlink |
| 🚨 Edit `/etc/fstab` tanpa test | Sistem tidak boot |
| 🚨 Edit `/etc/passwd` langsung | Lock dari sistem |
| 🚨 `mkfs` di partisi ter-mount | Hapus data |
| 🚨 `chmod -R 777 /` | Rusak permission sistem |
| 🚨 `dd if=/dev/zero of=/dev/sda` | Wipe disk |

---

## 6.3 Alur Troubleshooting Umum

| Masalah | Langkah Awal |
|---|---|
| Sistem tidak boot | Cek `/etc/fstab` dengan `mount -a`, atau masuk rescue mode |
| Disk penuh | `df -h` → `du -sh /* \| sort -h` |
| Lupa password root | Rescue mode → `passwd` |
| `fork: Resource temporarily unavailable` | Cek `ulimit -u`, kurangi proses, atau naikkan limit |
| SSH tidak bisa login | Cek permission `~/.ssh`, `authorized_keys`, `PermitRootLogin` |
| Load average tinggi | `top` → cek proses, bandingkan dengan `nproc` |
| File hilang | Cek hard link dengan `ls -li`, atau cek backup |

---

## 6.4 Best Practice untuk Pemula

1. **Selalu pakai `sudo`** untuk perintah yang butuh root. Jangan login sebagai root langsung.
2. **Biasakan `ls -lah`** sebelum menghapus — pastikan yang dihapus benar.
3. **Pakai `rm -ri`** daripada `rm -r` — konfirmasi menyelamatkan.
4. **Test `/etc/fstab` dengan `mount -a`** sebelum reboot.
5. **Jangan edit file sistem langsung** — pakai tools resmi.
6. **Backup dulu** sebelum eksperimen berbahaya.
7. **Baca `man` page** — `man ls`, `man chage`, dll.
8. **Catat** apa yang kamu lakukan — untuk dirimu sendiri nanti.
9. **Test di VM** sebelum produksi.
10. **Konsisten** dengan konvensi — jangan bikin standar sendiri.

---

## 📖 Glosarium

| Istilah | Arti Singkat |
|---|---|
| **APT** | Advanced Package Tool — package manager Debian/Ubuntu |
| **Bash** | Bourne Again Shell — shell default di banyak Linux |
| **chroot** | Mengubah root directory untuk isolasi |
| **Cron** | Scheduler untuk menjalankan tugas berkala |
| **Daemon** | Proses yang berjalan di background |
| **df** | Disk Free — lihat penggunaan filesystem |
| **du** | Disk Usage — lihat ukuran file/direktori |
| **ext4** | Filesystem default di banyak distro Linux |
| **fstab** | File System Table — konfigurasi mount otomatis |
| **GECOS** | Field comment di `/etc/passwd` (nama lengkap, dll) |
| **GID** | Group ID — nomor unik untuk group |
| **GRUB** | Bootloader yang umum dipakai Linux |
| **Hard link** | Nama tambahan yang menunjuk inode yang sama |
| **Inode** | "KTP" file — metadata, bukan nama |
| **Kernel** | Inti sistem operasi yang mengelola hardware |
| **Load average** | Panjang antrean proses yang menunggu CPU |
| **LVM** | Logical Volume Manager — partisi virtual |
| **MBR** | Master Boot Record — skema partisi lama |
| **GPT** | GUID Partition Table — skema partisi modern |
| **OOM** | Out of Memory — sistem kehabisan RAM |
| **PAM** | Pluggable Authentication Modules |
| **Parted** | Tool untuk partisi disk (GPT/MBR) |
| **Pipe** | `\|` — menghubungkan output ke input |
| **PV** | Physical Volume (LVM) |
| **VG** | Volume Group (LVM) |
| **LV** | Logical Volume (LVM) |
| **rbash** | Restricted bash — shell terbatas |
| **Root** | Superuser dengan akses tak terbatas |
| **SELinux** | Security-Enhanced Linux — MAC |
| **AppArmor** | Alternatif SELinux, lebih mudah |
| **SIGTERM** | Sinyal 15 — berhenti sopan |
| **SIGKILL** | Sinyal 9 — paksa berhenti |
| **SSH** | Secure Shell — akses remote aman |
| **sudo** | Superuser Do — jalankan sebagai root |
| **Swap** | Memori cadangan di disk |
| **Symlink** | "Shortcut" ke file/folder lain |
| **Systemd** | Init system modern di banyak distro |
| **UID** | User ID — nomor unik untuk user |
| **ulimit** | User limit — batas resource |
| **Vim** | Vi IMproved — editor powerful |
| **Zombie** | Proses yang sudah selesai tapi belum di-reap parent-nya |

---

## 🚀 Kalau Baru Mulai

**Urutan belajar yang disarankan:**

1. **Bagian 1** — kuasai navigasi & file. Ini fondasi.
2. **Bagian 2** — praktik pipe & filter di terminal. Ini inti Linux.
3. **Bagian 3** — pelajari proses & resource saat sudah nyaman.
4. **Bagian 4** — storage & package untuk server.
5. **Bagian 5** — security & user management untuk produksi.
6. **Bagian 6** — referensi cepat saat lupa.

**Tips belajar:**
- Praktik langsung di terminal — jangan cuma baca.
- Bikin VM sendiri (VirtualBox/VMware) untuk eksperimen bebas.
- Ulangi perintah sampai hafal tanpa lihat cheatsheet.
- Fokus ke **satu topik per minggu** — jangan loncat-loncat.

---

## ⚠️ Command yang Sering Typo

Berdasarkan pengalaman pribadi dan teman-teman:

| Typo Umum | Seharusnya |
|---|---|
| `chage -l user>` (ada `>`) | `chage -l user` |
| `du_each.tx` | `du_each.txt` |
| `usermod -G` (tanpa `-a`) | `usermod -aG` |
| `df_all.txt>` | `df_all.txt` |
| `wm` (bukan `rm`) | `rm` |
| `ifconfig` (deprecated) | `ip a` |
| `apt-get install` tanpa `sudo` | `sudo apt install` |
| `rm -rf /` (ada spasi) | `rm -rf /path` |
| `chmod 777` (semua file) | `chmod 644` atau `755` |

> 💡 **Tips:** Kalau muncul prompt `>`, berarti bash menunggu input tambahan. Tekan **CTRL + C** untuk membatalkan. Biasanya karena ada tanda kutip atau `>` yang belum ditutup.

---

## 🆘 Kalau Panic

**Situasi darurat dan langkah pertamanya:**

| Situasi | Langkah Pertama |
|---|---|
| Sistem tidak boot | Masuk rescue mode dari GRUB |
| Lupa password root | Rescue mode → `passwd` |
| Disk penuh total | `df -h` → `du -sh /* \| sort -h` → hapus file besar |
| Fork error | `ulimit -u` — cek limit, kurangi proses |
| SSH tidak bisa login | Cek `~/.ssh` permission, `sshd_config` |
| Command hang | `CTRL + C` (interrupt) |
| Vim stuck | `Esc` → `:q!` (keluar tanpa save) |
| Nano stuck | `CTRL + X` → `N` (discard) |
| Terminal kacau | `reset` atau `stty sane` |
| Lupa perintah | `man <perintah>` atau `--help` |

> 💡 **Tips:** Kalau benar-benar panik dan tidak tahu harus apa — **jangan ketik apapun dulu**. Tarik napas, baca pesan error, baru cari solusi. Panik biasanya bikin kesalahan makin parah.

---

## 📌 Disclaimer

> ⚠️ Cheatsheet ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa.
> Materi asli tidak didistribusikan di repositori ini.

---

**Referensi Lab:**

- **Bagian 1–2:** Lab 3.1 – 3.4 (File, Directory, Editor, Pipe)
- **Bagian 3.1:** Lab 4.1 – 4.2 (Resource Limits)
- **Bagian 3.2:** Lab 5.1 (Process Management)
- **Bagian 3.3:** Lab 7.1 (Monitoring)
- **Bagian 4.1:** Lab 6.1 (APT)
- **Bagian 4.2:** Lab 6.2 (External Repository)
- **Bagian 4.3:** Lab 9.1 (Swap)
- **Bagian 4.4:** Lab 9.2 (df/du)
- **Bagian 4.5:** Lab 9.3 (Partition)
- **Bagian 4.6:** Lab 10.1 (LVM)
- **Bagian 5.1:** Lab 8.1 – 8.3 (Inode, Hard Link, Symlink)
- **Bagian 5.2:** Lab 12.1, 12.5 (User & Group)
- **Bagian 5.3:** Lab 12.3 (Password Aging)
- **Bagian 5.4:** Lab 12.2 (Restricted User)
- **Bagian 5.5:** Lab 12.4 (SSH Key Auth)
- **Bagian 5.6:** Lab 11.1 (Kernel & Boot)
