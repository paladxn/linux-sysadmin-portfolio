# Lab 8.3 — Hard Link & Soft Link (Praktik Gabungan)

**Course:** Linux System Administration (Adinusa)
**Topic:** Perbedaan hard link & soft link, termasuk link ke direktori
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membedakan **hard link** dan **soft link** secara praktis
- Membuat link antar file **dan direktori**
- Memverifikasi bagaimana perubahan di satu file memengaruhi file yang di-link
- Memahami mengapa hard link **tidak bisa** dibuat untuk direktori, sementara soft link bisa

---

## 📘 Konsep Dasar

Sebelum praktik, beberapa poin penting:

| Aspek | Hard Link | Soft Link (Symbolic) |
|---|---|---|
| Dibuat dengan | `ln source target` | `ln -s source target` |
| Menunjuk ke | Inode | Path |
| Inode number | Sama dengan source | Berbeda |
| Lintas filesystem | ❌ Tidak bisa | ✅ Bisa |
| Link ke direktori | ❌ Tidak bisa | ✅ Bisa |
| Kalau target dihapus | Data tetap ada | Link rusak (dangling) |

**Filosofi:**
> Hard link = "nama kedua untuk file yang sama".
> Soft link = "penunjuk yang berisi path ke file/direktori lain".

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-008-3
```

---

## 🧪 Challenge (Practice Task)

### Soal

1. Di direktori `~/lab83/src`, buat file `notes.txt` dengan isi `My first notes`.
2. Buat **hard link** dari `notes.txt` di direktori `~/lab83/bck` dengan nama `backupnotes.txt`.
3. Tulis username kamu ke dalam file `~/lab83/src/sourcefile.txt`.
4. Buat **soft link** bernama `linktosource.txt` di `~/lab83/bck` yang menunjuk ke `~/lab83/src/sourcefile.txt`.
5. Di dalam direktori `lab83`, buat soft link lain bernama `tmplink` yang menunjuk ke `/tmp`.

### ✅ Solusi

```bash
# 0. Masuk ke direktori lab83 (asumsi sudah ada dari start lab)
$ cd ~/lab83

# 1. Buat file notes.txt
$ echo "My first notes" > ~/lab83/src/notes.txt

# 2. Buat hard link di folder bck
$ ln ~/lab83/src/notes.txt ~/lab83/bck/backupnotes.txt

# 3. Tulis username ke sourcefile.txt
$ whoami > ~/lab83/src/sourcefile.txt
# atau langsung tulis manual:
# echo "arijfarid69" > ~/lab83/src/sourcefile.txt

# 4. Buat soft link ke sourcefile.txt
$ ln -s ~/lab83/src/sourcefile.txt ~/lab83/bck/linktosource.txt

# 5. Buat soft link ke /tmp
$ ln -s /tmp ~/lab83/tmplink
```

### 🔍 Verifikasi

**A. Cek struktur direktori:**

```bash
$ ls -lR ~/lab83
```

Output yang diharapkan:

```
/home/student/lab83:
total 0
drwxr-xr-x 2 student student  ... bck
drwxr-xr-x 2 student student  ... src
lrwxrwxrwx 1 student student    4 ... tmplink -> /tmp

/home/student/lab83/bck:
total 0
-rw-r--r-- 2 student student ... backupnotes.txt
lrwxrwxrwx 1 student student ... linktosource.txt -> /home/student/lab83/src/sourcefile.txt

/home/student/lab83/src:
total 0
-rw-r--r-- 2 student student ... notes.txt
-rw-r--r-- 1 student student ... sourcefile.txt
```

**Perhatikan:**
- `backupnotes.txt` **tidak punya tanda `->`** → ini hard link.
- `linktosource.txt` **punya tanda `->`** → ini soft link.
- `tmplink` juga punya tanda `->` dan menunjuk ke `/tmp`.

**B. Verifikasi hard link punya inode yang sama:**

```bash
$ ls -li ~/lab83/src/notes.txt ~/lab83/bck/backupnotes.txt
```

Output yang diharapkan:

```
264540 -rw-r--r-- 2 student student 15 ... /home/student/lab83/src/notes.txt
264540 -rw-r--r-- 2 student student 15 ... /home/student/lab83/bck/backupnotes.txt
```

**Inode number sama** (`264540`) dan **link count = 2** → hard link terbukti.

**C. Verifikasi soft link punya inode berbeda:**

```bash
$ ls -li ~/lab83/src/sourcefile.txt ~/lab83/bck/linktosource.txt
```

Output yang diharapkan:

```
264541 -rw-r--r-- 1 student student 11 ... sourcefile.txt
264542 lrwxrwxrwx 1 student student 42 ... linktosource.txt -> /home/student/lab83/src/sourcefile.txt
```

**Inode berbeda** (`264541` vs `264542`) → soft link terbukti.

**D. Verifikasi perubahan di satu file memengaruhi hard link:**

```bash
$ echo "Baris tambahan" >> ~/lab83/src/notes.txt
$ cat ~/lab83/bck/backupnotes.txt
# Output:
# My first notes
# Baris tambahan
```

Perubahan langsung terlihat karena keduanya berbagi inode.

**E. Verifikasi soft link ke direktori:**

```bash
$ ls ~/lab83/tmplink | head
```

Output: daftar file di `/tmp` — karena `tmplink` menunjuk ke sana.

**F. Verifikasi symlink ke file:**

```bash
$ cat ~/lab83/bck/linktosource.txt
# Output: arijfarid69 (username kamu)
```

**G. Cek grading:**

```bash
$ nusactl grade linlab-008-3
```

---

## 🐛 Troubleshooting & Kesalahan

- **Hard link ke direktori gagal:** Saat mencoba `ln ~/lab83/src ~/lab83/srclink`, muncul error *"hard link not allowed for directory"*. Ini **memang dilarang** di Linux untuk mencegah loop di struktur direktori. Kalau butuh link ke direktori, gunakan **soft link** (`ln -s`).

- **Hard link lintas filesystem gagal:** Saat mencoba `ln ~/lab83/src/notes.txt /tmp/notes-link.txt`, muncul error *"Invalid cross-device link"*. Karena `/home` dan `/tmp` berada di filesystem yang berbeda, hard link tidak bisa dibuat. Solusinya: soft link.

- **Lupa `-s` saat buat soft link:** Awalnya saya buat `ln sourcefile.txt linktosource.txt` (tanpa `-s`), dan yang terbuat adalah **hard link**, bukan soft link. Ciri-cirinya: tidak ada tanda `->` di output `ls -l`, dan inode-nya sama. Selalu gunakan `-s` untuk soft link.

- **Path relatif vs absolut di symlink:** Saat membuat `linktosource.txt`, saya memakai path absolut (`/home/student/lab83/src/sourcefile.txt`) agar symlink tetap valid meskipun dipindah. Kalau pakai path relatif, symlink bisa rusak saat direktori-nya direlokasi.

- **`cat` ke symlink yang menunjuk direktori:** `cat tmplink` akan error karena `tmplink` menunjuk direktori, bukan file. Gunakan `ls tmplink` untuk melihat isinya.

- **Grading gagal karena urutan salah:** Kalau file dihapus atau symlink dibuat dengan nama berbeda, grading bisa gagal. Pastikan:
  - Nama file persis: `notes.txt`, `backupnotes.txt`, `sourcefile.txt`, `linktosource.txt`, `tmplink`.
  - Lokasi file persis: `~/lab83/src/` dan `~/lab83/bck/`.
  - `tmplink` ada di `~/lab83/`, bukan di dalam `src` atau `bck`.

- **Folder `bck` atau `src` belum ada:** Kalau `mkdir` tidak dijalankan (mungkin terlewat saat start lab), buat dulu:
  ```bash
  $ mkdir -p ~/lab83/src ~/lab83/bck
  ```

---

## 💡 Catatan & Insight Pribadi

- **Kenapa hard link tidak bisa untuk direktori?** Karena direktori punya struktur khusus (`.`, `..`) yang bisa menyebabkan **loop tak terbatas** kalau di-hard link. Bayangkan jika `dir` punya hard link di subdirektori-nya sendiri — traversal direktori bisa terjebak selamanya.

- **Symlink ke direktori sangat umum di Linux:**
  - `/usr/bin/python3` sering symlink ke `python3.12`.
  - `/etc/nginx/sites-enabled/` berisi symlink ke `sites-available/`.
  - `systemctl enable service` membuat symlink di `/etc/systemd/system/multi-user.target.wants/`.

- **Cara cek apakah sebuah file adalah symlink:**
  ```bash
  $ test -L file && echo "symlink"
  $ file file
  $ readlink file    # tampilkan target
  ```

- **Cara menghapus symlink tanpa menghapus targetnya:**
  ```bash
  $ rm symlink-name    # aman, hanya menghapus symlink
  ```
  **JANGAN** pakai trailing slash saat `rm` symlink ke direktori:
  ```bash
  $ rm -r symlink-to-dir/   # ❌ ini bisa menghapus isi direktori target!
  ```

- **Cara hapus hard link:**
  ```bash
  $ rm salah-satu-nama
  ```
  Data tetap ada selama masih ada nama lain yang menunjuk inode-nya. Link count berkurang 1.

- **`find` untuk mencari symlink rusak (dangling):**
  ```bash
  $ find ~/lab83 -xtype l
  ```

- **`cp` vs `cp -P` vs `cp -L`:**
  - `cp link target` → copy **isi** yang ditunjuk symlink (mengikuti).
  - `cp -P link target` → copy **symlink-nya** (bukan isinya).
  - `cp -L link target` → paksa follow symlink.

- **Kasus nyata:** Di server produksi, symlink sering dipakai untuk **deployment**:
  - Aplikasi versi baru ditaruh di `/var/www/app-v2/`.
  - Symlink `/var/www/current` → `/var/www/app-v2/`.
  - Kalau ada masalah, tinggal ganti symlink ke `app-v1/` — rollback instan tanpa pindah file.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `ln source target` | Buat hard link |
| `ln -s source target` | Buat soft link (file atau direktori) |
| `ls -l` | Lihat tipe file (`l` = symlink, `-` = file biasa) |
| `ls -li` | Lihat inode + link count |
| `readlink link` | Tampilkan path target symlink |
| `file file` | Identifikasi tipe file |
| `test -L file` | Cek apakah file adalah symlink |
| `find . -xtype l` | Cari symlink rusak |
| `rm link` | Hapus symlink (target aman) |

---

## 📌 Kesimpulan

Lab ini menggabungkan konsep hard link (Lab 8.1) dan soft link (Lab 8.2) dalam praktik yang lebih kompleks. Yang paling berharga dari lab ini adalah pemahaman **kapan memakai hard link dan kapan memakai soft link**:

- **Hard link** → untuk file dalam filesystem yang sama, ketika ingin data tetap ada meski salah satu nama dihapus.
- **Soft link** → untuk direktori, lintas filesystem, atau ketika ingin target bisa dengan mudah diganti.

Kemampuan membuat symlink ke direktori (`tmplink -> /tmp`) adalah dasar dari banyak pola konfigurasi server, termasuk manajemen versi aplikasi dan site configuration di web server.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.