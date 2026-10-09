# Lab 13.4 — Managing File Attributes (`chattr` & `lsattr`)

**Course:** Linux System Administration (Adinusa)
**Topic:** File attributes — immutable & append-only
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Melihat atribut file dengan `lsattr`
- Mengatur **immutable** (`+i`) dan **append-only** (`+a`) dengan `chattr`
- Menguji perilaku proteksi file
- Memverifikasi perubahan atribut

---

## 📘 Konsep Dasar

| Atribut | Simbol | Efek |
|---|---|---|
| Immutable | `i` | File tidak bisa diubah, di-rename, atau dihapus — bahkan oleh root |
| Append-only | `a` | File hanya bisa ditambah isinya, tidak bisa di-overwrite atau dihapus |
| Extent | `e` | Default di ext4 — menandakan file pakai extents |

**Perintah:**

| Perintah | Fungsi |
|---|---|
| `lsattr` | Lihat atribut file |
| `chattr +i file` | Set immutable |
| `chattr -i file` | Hapus immutable |
| `chattr +a file` | Set append-only |
| `chattr -a file` | Hapus append-only |
| `chattr = file` | Reset semua atribut |

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-013-4
```

---

## 📘 Guided Example

### 1. Buat direktori dan file uji

```bash
$ mkdir -p /tmp/attr_lab
$ cd /tmp/attr_lab
$ sudo touch config.txt log.txt
```

### 2. Lihat atribut awal

```bash
$ lsattr
--------------e------- ./config.txt
--------------e------- ./log.txt
```

Hanya atribut `e` (extent) yang aktif — normal di ext4.

### 3. Set immutable pada `config.txt`

```bash
$ sudo chattr +i config.txt
$ lsattr config.txt
----i---------e------- config.txt
```

### 4. Uji proteksi immutable

```bash
$ echo "test" | sudo tee config.txt
tee: config.txt: Operation not permitted

$ sudo rm config.txt
rm: cannot remove 'config.txt': Operation not permitted
```

Kedua perintah gagal — file tidak bisa diubah maupun dihapus.

### 5. Hapus immutable dan verifikasi

```bash
$ sudo chattr -i config.txt
$ echo "test from immutable" | sudo tee config.txt
test from immutable
```

Sekarang file bisa diedit.

### 6. Set append-only pada `log.txt`

```bash
$ sudo chattr +a log.txt
$ echo "Log entry 1" | sudo tee -a log.txt
Log entry 1
```

### 7. Uji proteksi append-only

```bash
$ echo "overwrite" | sudo tee log.txt
tee: log.txt: Operation not permitted

$ sudo rm log.txt
rm: cannot remove 'log.txt': Operation not permitted
```

Hanya append yang berhasil. Overwrite dan hapus gagal.

### 8. Verifikasi atribut akhir

```bash
$ lsattr /tmp/attr_lab
--------------e------- /tmp/attr_lab/config.txt
-----a--------e------- /tmp/attr_lab/log.txt
```

`config.txt` kembali normal, `log.txt` punya atribut `a` (append-only).

---

## 🧪 Practice Task

### Soal

1. Buat file `secure.txt` di `/tmp/attr_lab`.
2. Set immutable, uji ubah & hapus — pastikan gagal.
3. Hapus immutable.
4. Set append-only.
5. Tambahkan dua baris teks.
6. Uji overwrite & hapus — pastikan gagal.
7. Verifikasi atribut akhir.

### ✅ Solusi

```bash
cd /tmp/attr_lab
sudo touch secure.txt

# Immutable
sudo chattr +i secure.txt
echo "coba" | sudo tee secure.txt   # gagal
sudo rm secure.txt                  # gagal

# Hapus immutable
sudo chattr -i secure.txt

# Append-only
sudo chattr +a secure.txt
echo "baris 1" | sudo tee -a secure.txt
echo "baris 2" | sudo tee -a secure.txt

# Uji proteksi
echo "overwrite" | sudo tee secure.txt   # gagal
sudo rm secure.txt                        # gagal

# Verifikasi
lsattr secure.txt
cat secure.txt
```

### 🔍 Hasil yang Diharapkan

`lsattr secure.txt`:
```
-----a--------e------- secure.txt
```

`cat secure.txt`:
```
baris 1
baris 2
```

---

## 🐛 Troubleshooting & Kesalahan

- **Langkah terlewat:** Saya sempat lupa menulis ulang `config.txt` setelah `chattr -i`, dan lupa set `+a` pada `log.txt`. Akibatnya grading gagal di case 2, 3, 4. **Pelajaran:** ikuti urutan instruksi dengan teliti, dan verifikasi setiap langkah sebelum lanjut.

- **`chattr: Operation not supported`:** Filesystem tidak mendukung atribut (misal FAT32, exFAT). Pastikan pakai **ext4**, **xfs**, atau **btrfs**.

- **`lsattr: Operation not supported`:** Sama — filesystem tidak mendukung.

- **Atribut `e` (extent):** Ini default di ext4. Tidak perlu diubah. Fokus ke `i` dan `a`.

- **Ingin lihat semua atribut termasuk hidden:** `lsattr -a`.

- **Ingin reset semua atribut:** `chattr = file` (sama dengan menghapus semua atribut).

- **Immutable pada direktori:** Bisa, tapi hati-hati — direktori jadi tidak bisa diubah isinya.

- **Root tetap tidak bisa:** Immutable dan append-only **berlaku bahkan untuk root**. Ini yang membuatnya kuat.

- **`rm` gagal tapi file masih ada:** Itu benar — append-only mencegah penghapusan.

- **`tee` tanpa `-a` tetap gagal:** Karena mencoba overwrite. Gunakan `tee -a` untuk append.

- **Typo `>` di akhir perintah:** Sering terjadi saat copy-paste. Kalau muncul prompt `>`, tekan **CTRL + C**.

---

## 💡 Catatan & Insight Pribadi

### Kapan Pakai `chattr`?

- **Immutable (`+i`):** File konfigurasi kritis yang tidak boleh diubah tanpa sengaja — misal `/etc/hosts`, `/etc/fstab`, `/etc/passwd`. Juga untuk file log yang harus utuh.
- **Append-only (`+a`):** File log yang hanya boleh ditambah, tidak boleh dihapus/di-overwrite. Umum di server produksi untuk audit trail.

### Perbedaan `chmod` dan `chattr`

| Aspek | `chmod` | `chattr` |
|---|---|---|
| Mengatur | Permission (siapa boleh apa) | Atribut (sifat file) |
| Berlaku untuk | User/group/others | File itu sendiri |
| Root bisa bypass? | Ya | **Tidak** (untuk `i` dan `a`) |
| Contoh | `644`, `755` | `+i`, `+a` |

### Bahaya `chattr +i`

Kalau salah set `+i` pada file dan lupa cara melepasnya, file itu **tidak bisa dihapus bahkan oleh root**. Selalu ingat `chattr -i`.

### Kasus Nyata

- **Server produksi:** File konfigurasi penting di-set `+i` agar tidak ada yang sengaja/tidak sengaja mengubah.
- **Log server:** File log di-set `+a` agar tidak bisa dihapus attacker (untuk audit).
- **Backup:** File backup di-set `+i` agar tidak termodifikasi.

### Pelajaran Kunci

1. `lsattr` untuk lihat atribut.
2. `chattr +i` = proteksi total (bahkan root tidak bisa).
3. `chattr +a` = hanya bisa append.
4. Urutan langkah penting — jangan lewatkan verifikasi.
5. Selalu `chattr -i` sebelum menghapus file immutable.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `lsattr` | Lihat atribut |
| `lsattr -a` | Termasuk hidden |
| `chattr +i file` | Set immutable |
| `chattr -i file` | Hapus immutable |
| `chattr +a file` | Set append-only |
| `chattr -a file` | Hapus append-only |
| `chattr = file` | Reset semua atribut |

---

## 📌 Kesimpulan

Lab ini memperkenalkan **file attributes** — lapisan proteksi di atas permission biasa. Dengan `chattr`, kita bisa membuat file yang bahkan root tidak bisa ubah atau hapus. Sangat berguna untuk file konfigurasi kritis dan log yang harus utuh. Namun, hati-hati — kalau salah set, bisa merepotkan. Selalu ingat cara melepasnya dengan `chattr -i` atau `chattr -a`.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
