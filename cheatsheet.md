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

## 📌 Disclaimer

> ⚠️ Cheatsheet ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa.
> Materi asli tidak didistribusikan di repositori ini.

---

**Referensi Lab:**

- [Lab 3.1 — File & Directory Operations](lab-3.1-file-directory.md)
- [Lab 3.2 — Nano & Vim Text Editor](lab-3.2-nano-vim.md)
- [Lab 3.3 — Pipe, Wildcard & Redirection](lab-3.3-pipe-wildcard-redirection.md)
- [Lab 3.4 — `ls` Command & Pipe Combination](lab-3.4-ls-command.md)