# Lab 6.1 — Package Management dengan APT

**Course:** Linux System Administration (Adinusa)
**Topic:** Instalasi, update, dan hapus paket menggunakan APT
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Melakukan **update** dan **upgrade** paket menggunakan APT
- Mencari paket dan menampilkan detail informasi paket
- Menginstal dan menghapus paket
- Memverifikasi paket yang sudah terinstal beserta versinya

---

## 📘 Guided Example

### 1. Update package index

```bash
$ sudo apt update
```

**Penjelasan:** Memperbarui daftar paket yang tersedia dari repository. Ini **wajib** dilakukan sebelum instalasi agar mendapatkan versi terbaru.

### 2. Upgrade installed packages

```bash
$ sudo apt upgrade -y
```

**Penjelasan:** Meng-upgrade semua paket yang sudah terinstal ke versi terbaru. Opsi `-y` otomatis menjawab "yes" untuk konfirmasi.

### 3. Search for a package

```bash
$ apt search tree
```

Contoh output:

```
tree/noble 2.1.1-2ubuntu3 amd64
  displays an indented directory tree, in color
```

Menampilkan paket yang cocok dengan kata kunci beserta deskripsi singkat.

### 4. Show detailed package info

```bash
$ apt show tree
```

Contoh output (sebagian):

```
Package: tree
Version: 2.1.1-2ubuntu3
Priority: optional
Section: universe/utils
Origin: Ubuntu
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Original-Maintainer: Florian Ernst <florian@debian.org>
Bugs: https://bugs.launchpad.net/ubuntu/+filebug
Installed-Size: 111 kB
Depends: libc6 (>= 2.38)
```

Menampilkan informasi detail: versi, dependensi, deskripsi, ukuran, dan lainnya.

### 5. Install a package

```bash
$ sudo apt install tree -y
```

Menginstal paket `tree`. Opsi `-y` untuk auto-konfirmasi.

### 6. Verify installation

```bash
$ tree /etc
```

Menampilkan struktur direktori `/etc` dalam bentuk pohon (tree).

### 7. Lihat daftar paket terinstal

```bash
$ apt list --installed
```

Contoh output (baris untuk `tree`):

```
tree/noble,now 2.1.1-2ubuntu3 amd64 [installed]
```

---

## 🧪 Practice Task

### Soal

1. Cari paket `htop` menggunakan `apt search`.
2. Tampilkan informasi detail paket `htop` dengan `apt show`.
3. Instal `htop`.
4. Verifikasi bahwa `htop` sudah terinstal dengan `apt list --installed | grep htop`.
5. Jalankan `htop` sebentar (tekan `q` untuk keluar).
6. Hapus paket `htop` beserta konfigurasinya menggunakan `apt purge htop -y`.
7. Verifikasi bahwa `htop` sudah tidak ada di daftar paket terinstal.

### ✅ Solusi

```bash
# 1. Cari paket htop
$ apt search htop

# 2. Lihat detail paket htop
$ apt show htop

# 3. Instal htop
$ sudo apt install htop -y

# 4. Verifikasi instalasi
$ apt list --installed | grep htop

# 5. Jalankan htop (tekan q untuk keluar)
$ htop

# 6. Hapus htop beserta konfigurasinya
$ sudo apt purge htop -y

# 7. Verifikasi sudah terhapus
$ apt list --installed | grep htop
# (tidak ada output)
```

### 🔍 Hasil Akhir yang Diharapkan

Setelah langkah 7, perintah `apt list --installed | grep htop` tidak menghasilkan output apa pun, menandakan `htop` sudah terhapus.

---

## 🐛 Troubleshooting & Kesalahan

- **Lupa `sudo`:** Saat pertama kali mencoba `apt install tree` tanpa `sudo`, muncul error *"Permission denied"*. Ternyata instalasi paket butuh akses root. Solusinya: selalu gunakan `sudo` untuk `apt install`, `apt upgrade`, `apt purge`, dll.

- **Belum `apt update`:** Saya sempat langsung `apt install` tanpa `apt update` terlebih dahulu, dan paket yang dicari tidak ditemukan. Setelah menjalankan `sudo apt update`, paket baru muncul. Pelajaran: **selalu update index** sebelum instalasi.

- **`apt upgrade` vs `apt full-upgrade`:** Saya bingung kenapa `apt upgrade` kadang tidak meng-upgrade semua paket. Ternyata `apt upgrade` tidak menghapus paket yang konflik, sedangkan `apt full-upgrade` bisa menghapus paket jika diperlukan untuk menyelesaikan dependensi. Untuk pemakaian sehari-hari, `apt upgrade` sudah cukup.

- **Menghapus konfigurasi:** Saat menghapus paket dengan `apt remove`, file konfigurasi masih tersisa. Untuk menghapusnya juga, gunakan `apt purge`. Ini penting saat ingin benar-benar bersih.

- **Paket tidak ditemukan:** Jika `apt search` tidak menemukan paket, pastikan repository universe sudah aktif. Di Ubuntu, beberapa paket seperti `tree` ada di repository `universe`. Cek dengan `sudo add-apt-repository universe` jika perlu.

---

## 💡 Catatan & Insight Pribadi

- **Perbedaan `apt update` dan `apt upgrade`:**
  - `apt update` → memperbarui **daftar** paket dari repository (tidak menginstal apa pun).
  - `apt upgrade` → meng-upgrade paket yang sudah terinstal ke versi terbaru.

- **`apt` vs `apt-get`:** `apt` adalah versi modern yang lebih ramah pengguna (ada progress bar, warna). `apt-get` masih dipakai untuk scripting karena outputnya lebih stabil. Untuk interaktif, `apt` lebih nyaman.

- **Perintah `apt` yang sering dipakai:**

  | Perintah | Fungsi |
  |---|---|
  | `apt update` | Update daftar paket |
  | `apt upgrade` | Upgrade paket terinstal |
  | `apt install <paket>` | Instal paket |
  | `apt remove <paket>` | Hapus paket (konfigurasi tersisa) |
  | `apt purge <paket>` | Hapus paket + konfigurasi |
  | `apt search <kata>` | Cari paket |
  | `apt show <paket>` | Detail paket |
  | `apt list --installed` | Daftar paket terinstal |
  | `apt autoremove` | Hapus dependensi yang tidak terpakai |

- **`apt autoremove`:** Setelah menghapus paket, sering ada dependensi yang ikut terinstal tapi tidak diperlukan lagi. `apt autoremove` membersihkannya.

- **Kasus nyata:** Di server produksi, `apt update && apt upgrade -y` sering dijalankan secara berkala (misal via cron) untuk menjaga keamanan sistem. Tapi harus hati-hati karena upgrade bisa mengubah versi kernel atau library yang berpotensi memecahkan aplikasi. Biasanya upgrade dilakukan di jadwal maintenance.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `sudo apt update` | Update daftar paket dari repository |
| `sudo apt upgrade -y` | Upgrade paket terinstal |
| `apt search <kata>` | Cari paket |
| `apt show <paket>` | Tampilkan detail paket |
| `sudo apt install <paket> -y` | Instal paket |
| `apt list --installed` | Daftar paket terinstal |
| `sudo apt purge <paket> -y` | Hapus paket + konfigurasi |
| `sudo apt autoremove` | Hapus dependensi tak terpakai |

---

## 📌 Kesimpulan

Lab ini memperkenalkan **package management** menggunakan APT, yang merupakan keterampilan dasar wajib bagi sysadmin Debian/Ubuntu. Menguasai `apt` memungkinkan kita menginstal, meng-update, dan menghapus perangkat lunak dengan aman. Selain itu, pemahaman tentang `apt update` vs `apt upgrade`, serta `remove` vs `purge`, sangat penting untuk menjaga kebersihan dan keamanan sistem.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.