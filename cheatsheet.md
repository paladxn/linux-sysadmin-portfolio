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

## 📌 Disclaimer

> ⚠️ Cheatsheet ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa.
> Materi asli tidak didistribusikan di repositori ini.

---

