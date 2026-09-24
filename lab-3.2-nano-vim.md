# Lab 3.2 — Nano & Vim Text Editor

**Course:** Linux System Administration (Adinusa)
**Topic:** Text editor dasar (Nano & Vim)
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membuka, mengedit, menyimpan, dan menutup file menggunakan **Nano**
- Memahami konsep **mode** pada **Vim** untuk editing yang lebih efisien
- Mengetahui perbedaan Nano dan Vim agar bisa memilih editor sesuai kebutuhan

---

## 📘 Guided Example

### 1. Membuat file baru dengan Nano

```bash
$ nano notes.txt
```

Perintah ini membuka editor Nano. Ketik teks, misalnya:

```
Learning Linux is fun.
```

### 2. Perintah dasar Nano

| Aksi | Shortcut |
|---|---|
| Save | `CTRL + O` lalu `Enter` |
| Exit | `CTRL + X` |
| Search text | `CTRL + W`, ketik kata, `Enter` |
| Set mark (mulai seleksi) | `CTRL + ^` (`CTRL + Shift + 6`) |
| Cut (mark & cut) | `CTRL + K` |
| Paste | `CTRL + U` |

### 3. Save & Exit di Nano

```
CTRL + O → Enter → CTRL + X
```

### 4. Membuka file yang sama dengan Vim

```bash
$ vim notes.txt
```

Vim secara default mulai di **normal mode**.

**Catatan tentang `vi` vs `vim`:**

- `vi` adalah editor UNIX original.
- `vim` (*Vi IMproved*) adalah versi enhancement dengan lebih banyak fitur.
- Di banyak distro modern, menjalankan `vi` sebenarnya menjalankan `vim`.
- Keduanya bisa dipakai bergantian di sebagian besar sistem Linux.

### 5. Perintah dasar Vim

| Aksi | Perintah |
|---|---|
| Masuk insert mode | `i` |
| Keluar insert mode | `Esc` |
| Save & exit | `:wq` |
| Exit tanpa save | `:q!` |
| Search | `/kata` lalu `Enter` (`n` next, `N` previous) |
| Copy (yank) satu baris | `yy` |
| Paste di bawah | `p` |
| Paste di atas | `P` |
| Cut (delete) satu baris | `dd` |

### 6. Verifikasi perubahan

```bash
$ cat notes.txt
```

Menampilkan isi file untuk memastikan editan sudah tersimpan.

---

## 🧪 Practice Task

### Soal

1. Di dalam direktori `~/lab32`, buat file `lab.txt` menggunakan **nano**, tulis minimal 3 baris:

   ```
   Linux is powerful
   Learning nano is simple
   Text editors are useful
   ```

   Simpan dan keluar.

2. Buka `~/lab32/lab.txt` menggunakan **vim**, tambahkan baris baru di akhir:

   ```
   This line was added using vim
   ```

   Simpan file.

3. Masih di dalam vim:
   - Cari kata `Linux`.
   - Copy baris `Linux is powerful` dengan `yy`, lalu paste setelah baris terakhir dengan `p`.
   - Simpan dan keluar.

4. Tampilkan isi file dengan:

   ```bash
   $ cat ~/lab32/lab.txt
   ```

   File harus berisi minimal **5 baris** dengan modifikasi dari nano dan vim.

### ✅ Solusi

```bash
# 1. Buat file dengan nano
$ nano ~/lab32/lab.txt
# Ketik tiga baris teks, lalu CTRL+O → Enter → CTRL+X

# 2. Buka dengan vim
$ vim ~/lab32/lab.txt

# Di dalam vim:
#   - Tekan G untuk ke baris terakhir
#   - Tekan o untuk buka baris baru di bawah, masuk insert mode
#   - Ketik: This line was added using vim
#   - Tekan Esc untuk kembali ke normal mode

# 3. Search & copy-paste
#   - Ketik: /Linux lalu Enter  → cursor ke baris "Linux is powerful"
#   - Tekan yy                  → copy baris
#   - Tekan G                   → ke baris terakhir
#   - Tekan p                   → paste di bawah baris terakhir
#   - Ketik: :wq lalu Enter     → save & exit

# 4. Verifikasi
$ cat ~/lab32/lab.txt
```

### 🔍 Hasil Akhir yang Diharapkan

```
Linux is powerful
Learning nano is simple
Text editors are useful
This line was added using vim
Linux is powerful
```

---

## 💡 Catatan & Insight Pribadi

- Nano jauh lebih ramah untuk pemula karena semua shortcut ditampilkan di bagian bawah layar.
- Vim lebih cepat dan powerful, tapi butuh waktu untuk menghafal mode dan perintahnya.
- Konsep **mode** di Vim adalah hal yang paling sering bikin bingung pemula:
  - **Normal mode** → untuk navigasi & perintah (`yy`, `dd`, `p`, dll)
  - **Insert mode** → untuk mengetik teks (tekan `i`)
  - **Command mode** → untuk perintah seperti `:wq`, `:q!`
- Tips: kalau bingung di Vim, tekan `Esc` dulu untuk kembali ke normal mode, baru ketik perintah.
- Vim punya `vimtutor` — tutorial interaktif bawaan yang sangat berguna untuk latihan.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `nano file` | Membuka file dengan Nano |
| `vim file` | Membuka file dengan Vim |
| `CTRL + O` | Save (Nano) |
| `CTRL + X` | Exit (Nano) |
| `i` | Insert mode (Vim) |
| `Esc` | Keluar dari insert mode (Vim) |
| `:wq` | Save & exit (Vim) |
| `:q!` | Exit tanpa save (Vim) |
| `yy` / `dd` / `p` | Copy / cut / paste baris (Vim) |
| `/kata` | Search (Vim) |
| `cat` | Menampilkan isi file |

---

## 📌 Kesimpulan

Lab ini memberikan gambaran praktis tentang dua editor teks paling umum di Linux. Nano cocok untuk edit cepat dan konfigurasi sederhana, sedangkan Vim lebih powerful untuk pekerjaan jangka panjang dan editing kompleks. Menguasai keduanya adalah nilai tambah besar untuk seorang sysadmin.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.