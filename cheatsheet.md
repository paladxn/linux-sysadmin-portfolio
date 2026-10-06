# 🐧 Linux SysAdmin Cheatsheet

Referensi cepat perintah Linux yang dipelajari dari **Lab 3.1 – 3.4**
(Kursus Linux System Administration — Adinusa).

> 📌 Cheatsheet ini akan terus diperbarui seiring bertambahnya lab.

---

## 📂 1. Navigasi Direktori

| Perintah | Fungsi |
|---|---|
| `pwd` | Menampilkan direktori kerja saat ini (*print working directory*) |
| `cd folder` | Masuk ke folder tertentu |
| `cd ..` | Naik satu level ke direktori parent |
| `cd ~` | Kembali ke home directory |
| `cd -` | Kembali ke direktori sebelumnya |

---

## 📁 2. Manajemen File & Direktori

| Perintah | Fungsi |
|---|---|
| `mkdir nama` | Membuat direktori baru |
| `mkdir -p a/b/c` | Membuat direktori beserta parent-nya dalam satu perintah |
| `touch file.txt` | Membuat file kosong (atau update timestamp) |
| `rm file.txt` | Menghapus file |
| `rmdir folder` | Menghapus direktori **kosong** saja |
| `rm -r folder` | Menghapus direktori beserta isinya (rekursif) |
| `rm -ri folder` | Hapus rekursif dengan konfirmasi interaktif ✅ lebih aman |

---

## 📄 3. Melihat Isi File & Direktori

| Perintah | Fungsi |
|---|---|
| `ls` | Menampilkan isi direktori |
| `ls -a` | Tampilkan semua file, termasuk hidden (`.`) |
| `ls -l` | Format panjang (permission, owner, size, tanggal) |
| `ls -lah` | Detail + hidden + ukuran human-readable |
| `ls *.txt` | Menampilkan semua file dengan ekstensi `.txt` |
| `ls file*` | Menampilkan file yang diawali `file` |
| `cat file` | Tampilkan seluruh isi file |
| `less file` | Tampilkan isi file halaman per halaman (bisa di-scroll) |
| `ls --help` | Menampilkan bantuan perintah `ls` |

---

## 🧮 4. Membuat & Menulis Isi File

| Perintah | Fungsi |
|---|---|
| `echo "teks" > file` | Tulis teks ke file (**overwrite**) |
| `echo "teks" >> file` | Tambahkan teks ke akhir file (**append**) |
| `cat file1 file2 > file3` | Gabungkan dua file menjadi satu |
| `nano file` | Buka file dengan Nano (editor ramah pemula) |
| `vim file` | Buka file dengan Vim (editor powerful) |

---

## 🔗 5. Pipe, Filter & Hitung

| Perintah | Fungsi |
|---|---|
| `\|` (pipe) | Kirim output satu perintah ke perintah berikutnya |
| `ls -l \| less` | Lihat daftar file dengan scroll |
| `cat file \| grep kata` | Filter baris yang mengandung `kata` |
| `ls \| grep -c pola` | Hitung jumlah file yang cocok dengan pola |
| `ls \| wc -l` | Hitung jumlah item/baris |
| `wc -l` | Hitung baris |
| `wc -w` | Hitung kata |
| `wc -c` | Hitung byte |

---

## 🧠 6. Perintah Nano (Shortcut)

| Shortcut | Fungsi |
|---|---|
| `CTRL + O` lalu `Enter` | Simpan file |
| `CTRL + X` | Keluar dari Nano |
| `CTRL + W` | Cari teks |
| `CTRL + ^` (`CTRL + Shift + 6`) | Set mark (mulai seleksi) |
| `CTRL + K` | Cut (setelah mark) |
| `CTRL + U` | Paste |

---

## 🧠 7. Perintah Vim

### Mode

| Mode | Cara Masuk | Fungsi |
|---|---|---|
| Normal | `Esc` | Navigasi & perintah |
| Insert | `i` | Mengetik teks |
| Command | `:` | Perintah save/quit/search |

### Navigasi & Editing

| Perintah | Fungsi |
|---|---|
| `i` | Masuk insert mode |
| `Esc` | Keluar dari insert mode |
| `:wq` | Save & exit |
| `:q!` | Exit tanpa save |
| `/kata` | Cari kata (`n` next, `N` previous) |
| `yy` | Copy (yank) satu baris |
| `p` | Paste di bawah |
| `P` | Paste di atas |
| `dd` | Cut (delete) satu baris |
| `G` | Ke baris terakhir |
| `gg` | Ke baris pertama |
| `o` | Buka baris baru di bawah (insert mode) |

---

## 🧩 8. Kombinasi Pola yang Sering Dipakai

```bash
# Hitung jumlah file di direktori
$ ls | wc -l

# Hitung file dengan pola tertentu
$ ls | grep -c client

# Lihat isi file panjang dengan scroll
$ cat /etc/passwd | less

# Filter baris dari file
$ cat file.txt | grep Second

# Gabungkan dua file jadi satu
$ cat notes.txt data.txt > combined.txt

# Tulis hasil ke file dengan pola template
$ echo "client = $(ls | grep -c client)" > lab34-answer.txt

# Buang pesan error
$ ls pola* 2>/dev/null

# Lihat detail semua file termasuk hidden
$ ls -lah
```

---

## 🧭 9. Opsi Penting yang Perlu Dihafal

| Opsi | Arti |
|---|---|
| `-a` | All (tampilkan hidden files) |
| `-l` | Long format |
| `-h` | Human-readable (K, M, G) |
| `-r` | Recursive (rekursif) |
| `-i` | Interactive (konfirmasi tiap aksi) |
| `-p` | Buat parent directory (pada `mkdir`) |
| `-c` | Count (pada `grep`) |
| `-v` | Invert match (pada `grep`) |

---

## 🧠 10. Perbedaan Penting

| Konsep | Penjelasan |
|---|---|
| `>` vs `>>` | `>` menimpa isi, `>>` menambahkan di akhir |
| `rmdir` vs `rm -r` | `rmdir` hanya untuk direktori kosong, `rm -r` untuk direktori berisi |
| `vi` vs `vim` | `vi` = original, `vim` = versi enhanced (banyak distro me-link `vi` ke `vim`) |
| Nano vs Vim | Nano ramah pemula, Vim powerful & cepat (butuh hafal mode) |

---

## ⚙️ 11. Resource Limits (`ulimit`)

| Perintah | Fungsi |
|---|---|
| `ulimit -a` | Tampilkan semua limit |
| `ulimit -n` | Lihat/set limit open files |
| `ulimit -u` | Lihat/set limit proses per user |
| `ulimit -c` | Lihat/set limit core file size |
| `ulimit -f` | Lihat/set limit ukuran file |
| `sudo nano /etc/security/limits.conf` | Edit limit permanen |

---

### Persistent Limits (`/etc/security/limits.conf`)

```
<domain>   <type>   <item>   <value>
```

| Field | Nilai |
|---|---|
| `<domain>` | Username, `@groupname`, atau `*` |
| `<type>` | `soft`, `hard`, atau `-` |
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
## 🎯 12. Process Management

| Perintah | Fungsi |
|---|---|
| `command &` | Jalankan proses di background |
| `ps aux \| grep nama` | Cari proses berdasarkan nama |
| `pgrep -f pola` | Dapatkan PID berdasarkan pola |
| `kill PID` | Kirim `SIGTERM` ke proses |
| `kill -9 PID` | Paksa hentikan (`SIGKILL`) |
| `killall nama` | Hentikan semua proses dengan nama sama |
| `pkill -f pola` | Hentikan proses berdasarkan pola |
| `jobs` | Daftar job di shell |
| `fg` / `bg` | Pindah job ke foreground / background |

## 📦 13. Package Management (APT)

| Perintah | Fungsi |
|---|---|
| `sudo apt update` | Update daftar paket |
| `sudo apt upgrade -y` | Upgrade paket terinstal |
| `apt search <kata>` | Cari paket |
| `apt show <paket>` | Detail paket |
| `sudo apt install <paket> -y` | Instal paket |
| `apt list --installed` | Daftar paket terinstal |
| `sudo apt remove <paket>` | Hapus paket |
| `sudo apt purge <paket>` | Hapus paket + konfigurasi |
| `sudo apt autoremove` | Hapus dependensi tak terpakai |

## 📦 14. External Repository & Version Pinning

| Perintah | Fungsi |
|---|---|
| `curl -LsS <url> -o script` | Download script setup |
| `sudo ./mariadb_repo_setup --mariadb-server-version="mariadb-10.11"` | Setup repository MariaDB |
| `sudo tee /etc/apt/sources.list.d/mariadb.list` | Buat file repository manual |
| `apt-cache policy <paket>` | Cek versi yang tersedia |
| `sudo apt install <paket>=<versi>` | Instal versi spesifik |
| `apt-mark hold <paket>` | Kunci versi paket |
| `lsb_release -cs` | Cek codename distro |
| `sudo apt update` | Update index setelah tambah repository |

## 📊 15. Monitoring CPU & Load

| Perintah | Fungsi |
|---|---|
| `lscpu` | Info CPU (jumlah, arsitektur) |
| `nproc` | Jumlah logical CPU |
| `top` | Monitor proses real-time |
| `top -bn1` | Mode batch 1 iterasi |
| `top` interaktif: `l` `t` `m` `P` `M` `1` `k` `q` | Toggle header, sort, per-CPU, kill, quit |
| Load average | Rata-rata proses menunggu CPU (1, 5, 15 menit) |
| Aturan load | Load = jumlah CPU → CPU 100% penuh |

## 🔗 16. Inode & Hard Links

| Perintah | Fungsi |
|---|---|
| `ls -li file` | Lihat inode number + link count |
| `ln source target` | Buat hard link |
| `ln -s source target` | Buat symbolic link |
| `stat file` | Info lengkap (inode, link count, device) |
| `df -h /path` | Lihat filesystem dari path |

## 🔗 16. Inode & Links

| Perintah | Fungsi |
|---|---|
| `ls -li file` | Lihat inode number + link count |
| `ln source target` | Buat hard link |
| `ln -s source target` | Buat symbolic link |
| `readlink link` | Lihat path target symlink |
| `file link` | Identifikasi tipe file |
| `test -L link` | Cek apakah file adalah symlink |
| `stat file` | Info lengkap (inode, link count, device) |
| `df -h /path` | Lihat filesystem dari path |

### Symlink ke Direktori

```bash
$ ln -s /tmp ~/tmplink       # symlink ke direktori /tmp
$ ls ~/tmplink                # lihat isi /tmp
$ readlink ~/tmplink          # → /tmp
```

**Peringatan:** Saat menghapus symlink ke direktori, jangan pakai trailing slash:
```bash
$ rm ~/tmplink                # ✅ aman (hapus symlink saja)
$ rm -r ~/tmplink/            # ❌ bahaya (hapus isi /tmp!)
```

**Perbedaan Hard Link vs Symbolic Link:**

| Aspek | Hard Link | Symbolic Link |
|---|---|---|
| Menunjuk ke | Inode | Path |
| Inode number | Sama | Berbeda |
| Lintas filesystem | ❌ | ✅ |
| Link ke direktori | ❌ | ✅ |
| Target dihapus | Data tetap ada | Link rusak (dangling) |
| Tanda di `ls -l` | File biasa | `l` + `->` |

**Aturan:**
- Hard link → inode sama, hanya bisa dalam filesystem yang sama, tidak bisa ke direktori.
- Symbolic link → inode berbeda, bisa lintas filesystem.
- Data hilang hanya saat link count = 0.

## 💾 17. Swap & Storage

| Perintah | Fungsi |
|---|---|
| `fallocate -l 2G /swapfile` | Buat file swap 2 GB |
| `dd if=/dev/zero of=/swapfile bs=1M count=2048` | Alternatif buat file |
| `chmod 600 /swapfile` | Set permission ketat |
| `mkswap /swapfile` | Format sebagai swap |
| `swapon /swapfile` | Aktifkan swap |
| `swapoff /swapfile` | Nonaktifkan swap |
| `swapon --show` | Daftar swap aktif |
| `free -h` | Total memori + swap |
| `cat /proc/swaps` | Swap dari kernel |
| `blkid /swapfile` | Cek UUID swap |

**Format `/etc/fstab` untuk swap:**
```
/swapfile swap swap defaults 0 0
```

**Ukuran swap yang disarankan:**
- RAM ≤ 2 GB → 2× RAM
- RAM 2–8 GB → 1× RAM
- RAM > 8 GB → 0.5× RAM, min 4 GB

### 📊 Disk Usage (`df` & `du`)

| Perintah | Fungsi |
|---|---|
| `df -h` | Penggunaan filesystem (human-readable) |
| `df -Th` | Penggunaan + tipe filesystem |
| `df -i` | Penggunaan inode |
| `du -sh .` | Total ukuran direktori saat ini |
| `du -h dir/*` | Ukuran tiap file di direktori |
| `du --max-depth=1` | Ukuran per subdirektori (1 level) |
| `du -ah \| sort -h \| tail -20` | 20 file/direktori terbesar |
| `df -h > file.txt` | Simpan output ke file |

**Perbedaan:**
- `df` → filesystem usage (dari superblock, cepat)
- `du` → file/directory usage (baca setiap file, lambat)

### 🗂️ Partition Management

| Perintah | Fungsi |
|---|---|
| `lsblk` / `lsblk -f` | Lihat disk & partisi (opsional: filesystem) |
| `sudo parted /dev/sdX` | Tool partisi (GPT/MBR) |
| `(parted) mklabel gpt` | Buat table GPT |
| `(parted) mkpart` | Buat partisi |
| `sudo fdisk /dev/sdX` | Tool partisi interaktif |
| `(fdisk) n / d / w / q` | New / delete / write / quit |
| `mkfs -t ext4 -L LABEL /dev/sdX1` | Format + label |
| `blkid /dev/sdX1` | Lihat label + UUID |
| `mount LABEL=NAME /mnt/point` | Mount dengan label |
| `mount -a` | Mount semua dari fstab |
| `partprobe /dev/sdX` | Reload partition table |

**Format `/etc/fstab`:**
```
LABEL=NAMA  /mnt/point  ext4  defaults,nofail  0  2
```
- `LABEL=` → identifikasi device yang konsisten
- `nofail` → sistem tetap boot meski partisi tidak ada

## 🗄️ 18. LVM (Logical Volume Manager)

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
| `pvs` / `pvdisplay` | Info PV (ringkas / detail) |
| `vgs` / `vgdisplay` | Info VG |
| `lvs` / `lvdisplay` | Info LV |
| `vgextend <vg> /dev/sdc1` | Tambah PV ke VG |
| `lvextend -L +200M /dev/<vg>/<lv>` | Perbesar LV |
| `resize2fs /dev/<vg>/<lv>` | Resize filesystem ext4 |
| `lvreduce -L 200M /dev/<vg>/<lv>` | Perkecil LV |
| `lvremove` / `vgremove` / `pvremove` | Hapus (urutan: LV → VG → PV) |
| `lvcreate -s -n snap -L 100M /dev/<vg>/<lv>` | Snapshot LV |

**Catatan:**
- Flag `lvm on` wajib di partisi (via `parted set N lvm on`).
- `udevadm settle` sebelum `pvcreate` untuk memastikan device siap.
- `systemctl daemon-reload` setelah edit fstab.
- Resize: perbesar = `lvextend` → `resize2fs`; perkecil = `resize2fs` → `lvreduce`.

### 🔐 Password Aging (`chage`)

| Perintah | Fungsi |
|---|---|
| `chage -l <user>` | Lihat info aging |
| `chage -m N -M N <user>` | Min/max hari password |
| `chage -E YYYY-MM-DD <user>` | Expire akun |
| `chage -d 0 <user>` | Force ganti password saat login |

### 🔑 SSH (Secure Shell)

| Perintah | Fungsi |
|---|---|
| `ssh user@host` | Login remote |
| `ssh user@host "command"` | Jalankan perintah remote |
| `scp file user@host:/path` | Copy file |
| `scp -r dir user@host:/path` | Copy direktori |
| `ssh-keygen` | Generate key pair |
| `ssh-copy-id user@host` | Copy public key ke server |

**File SSH:**
- `~/.ssh/id_rsa` — private key (600)
- `~/.ssh/id_rsa.pub` — public key (644)
- `~/.ssh/authorized_keys` — public key yang diizinkan (600)
- `/etc/ssh/sshd_config` — config SSH server

### 📂 File Penting User/Group

| File | Permission | Isi |
|---|---|---|
| `/etc/passwd` | 644 | Data user |
| `/etc/shadow` | 400 | Hash password |
| `/etc/group` | 644 | Data group |
| `/etc/gshadow` | 400 | Password group |
| `/etc/skel/` | — | Template home user baru |

**Aturan emas:** Jangan pernah edit file-file di atas secara langsung — gunakan tools resmi (`useradd`, `usermod`, `chage`, dll).

## 👤 19. User & Group Management

| Perintah | Fungsi |
|---|---|
| `useradd <user>` | Buat user |
| `useradd -m -s /bin/bash <user>` | Buat user + home + shell |
| `userdel -r <user>` | Hapus user + home |
| `usermod -c "comment" <user>` | Set comment |
| `usermod -aG <group> <user>` | Tambah ke supplementary group (**wajib -a**) |
| `usermod -L` / `-U` | Lock / unlock user |
| `passwd <user>` | Set password |
| `passwd -S <user>` | Cek status password |
| `chage -d 0 <user>` | Force ganti password saat login |
| `groupadd -g <gid> <group>` | Buat group dengan GID |
| `id <user>` / `groups <user>` | Info user & group |
| `su - <user>` | Switch user |
| `sudo -l -U <user>` | Lihat sudo privilege user |
| `visudo -c` | Validasi sudoers |

**File penting:**
- `/etc/passwd` — data user
- `/etc/shadow` — password
- `/etc/group` — data group
- `/etc/sudoers.d/` — sudoers modular

**Format sudoers:**
```
%group  ALL=(ALL)  NOPASSWD:ALL
user    ALL=(root) NOPASSWD:/usr/bin/systemctl restart nginx
```

**Aturan penting:**
- Selalu `usermod -aG` (append), jangan `-G`.
- User harus logout & login ulang setelah diubah group-nya.
- Permission file di `/etc/sudoers.d/` harus `0440`.
- Hindari `NOPASSWD:ALL` di production.

## 🔒 20. Restricted User & Access Control

**Langkah membuat restricted user:**

```bash
# 1. Buat user
useradd -m -s /bin/bash nusa
passwd nusa

# 2. Aktifkan restricted shell
usermod -s /bin/rbash nusa

# 3. Buat direktori command
mkdir -p /home/nusa/bin
chown nusa:nusa /home/nusa/bin
chmod 755 /home/nusa/bin

# 4. Batasi PATH
echo 'export PATH=$HOME/bin' >> /home/nusa/.bash_profile

# 5. Symlink command yang diizinkan
ln -s /bin/ls /home/nusa/bin/ls
ln -s /bin/cat /home/nusa/bin/cat
ln -s /usr/bin/sudo /home/nusa/bin/sudo

# 6. Buat group & tambahkan user
groupadd -f limitednusa
usermod -aG limitednusa nusa

# 7. Sudoers terbatas
visudo -f /etc/sudoers.d/limitednusa
# Isi: %limitednusa ALL=(ALL) NOPASSWD: /bin/ls, /bin/cat
chmod 440 /etc/sudoers.d/limitednusa

# 8. Batasi login SSH
echo "-:limitednusa:ALL EXCEPT LOCAL" >> /etc/security/access.conf
```

**Batasan `rbash`:**
- ❌ `cd` ke direktori lain
- ❌ Set/modifikasi env var (PATH, SHELL, ENV)
- ❌ Command dengan `/` di path
- ❌ Redirect input/output (`>`, `<`, `>>`)

**Catatan:**
- `rbash` **bukan** security tool — mudah di-bypass.
- Selalu test dengan `su - user` setelah konfigurasi.
- File di `/etc/sudoers.d/` harus permission `0440` dan tanpa titik di nama.

### ⏳ Password Aging (`chage`)

| Perintah | Fungsi |
|---|---|
| `chage -l user` | Lihat info aging |
| `chage -M 30 -m 7 -W 5 user` | Set max/min/warning days |
| `chage -d 0 user` | Paksa ganti password saat login |
| `chage -E YYYY-MM-DD user` | Set account expiration |
| `chage -I N user` | Set inactive days setelah expired |

**Default policy system-wide:**
- Edit `/etc/login.defs`:
  ```
  PASS_MAX_DAYS   90
  PASS_MIN_DAYS   1
  PASS_WARN_AGE   7
  ```
- Hanya berlaku untuk user baru.

  ### 🔑 SSH Key Authentication

| Perintah | Fungsi |
|---|---|
| `ssh-keygen -t rsa -b 4096 -N ""` | Generate key pair tanpa passphrase |
| `ssh-copy-id -i ~/.ssh/id_rsa.pub -p PORT user@host` | Salin public key ke server |
| `ssh -p PORT user@host "command"` | Jalankan perintah remote |
| `ssh -i ~/.ssh/id_rsa -p PORT user@host` | Gunakan key spesifik |
| `ssh -o BatchMode=yes ...` | Verifikasi passwordless |
| `ssh-keyscan -p PORT host >> ~/.ssh/known_hosts` | Ambil host key |
| `ssh-keygen -R "[host]:PORT"` | Hapus host key lama |

**Permission file SSH:**
- `~/.ssh` → `700`
- `~/.ssh/id_rsa` → `600`
- `~/.ssh/id_rsa.pub` → `644`
- `~/.ssh/authorized_keys` → `600`

## 📌 Disclaimer

> ⚠️ Cheatsheet ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa.
> Materi asli tidak didistribusikan di repositori ini.

---

