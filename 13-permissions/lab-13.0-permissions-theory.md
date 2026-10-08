# Lab 13.0 — Permission, Ownership, Umask & ACL (Teori)

**Course:** Linux System Administration (Adinusa)
**Topic:** Teori permission, ownership, umask, dan ACL
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab teori ini, saya mampu:

- Memahami konsep **owner**, **group**, dan **permission** di Linux
- Membaca output `ls -l` dengan benar
- Menjelaskan bagaimana kernel memeriksa permission
- Memahami **special permissions** (SUID, SGID, sticky bit)
- Menggunakan **`chmod`**, **`chown`**, **`chgrp`**, dan **`umask`**
- Memahami konsep **ACL** untuk permission granular

---

## 📘 Bagian 1 — Konsep Dasar Permission

Setiap file dan direktori di Linux punya:

| Atribut | Arti |
|---|---|
| **Owner** | User yang memiliki file |
| **Group** | Group yang punya akses |
| **Permission** | Hak akses: read, write, execute |

Permission ini yang menentukan **siapa boleh apa** — fondasi keamanan Linux.

### Melihat Permission

```bash
$ ls -l a_file
-rw-rw-r-- 1 coop aproject 1601 Mar 9 15:04 a_file
```

### Breakdown Output `ls -l`

```
-rw-rw-r-- 1 coop aproject 1601 Mar 9 15:04 a_file
│└┬┘└┬┘└┬┘ │  │     │       │    │          │
│ │  │  │  │  │     │       │    │          └── nama file
│ │  │  │  │  │     │       │    └───────────── timestamp
│ │  │  │  │  │     │       └────────────────── ukuran (byte)
│ │  │  │  │  │     └────────────────────────── group
│ │  │  │  │  └──────────────────────────────── owner
│ │  │  │  └─────────────────────────────────── jumlah hard link
│ │  │  └────────────────────────────────────── others: r--
│ │  └───────────────────────────────────────── group: rw-
│ └──────────────────────────────────────────── user: rw-
└────────────────────────────────────────────── tipe file (-)
```

**Tipe file (karakter pertama):**

| Simbol | Tipe |
|---|---|
| `-` | File biasa |
| `d` | Direktori |
| `l` | Symbolic link |
| `c` | Character device |
| `b` | Block device |
| `s` | Socket |
| `p` | Named pipe (FIFO) |

---

## 📘 Bagian 2 — Tiga Kelompok Permission

| Kelompok | Simbol | Arti |
|---|---|---|
| **Owner** | `u` | Pemilik file |
| **Group** | `g` | Group pemilik file |
| **Others** | `o` | Semua user lain |

### Tiga Jenis Permission

| Permission | Simbol | Efek pada File | Efek pada Direktori |
|---|---|---|---|
| Read | `r` | Baca isi | List isi |
| Write | `w` | Ubah isi | Buat/hapus file |
| Execute | `x` | Jalankan | Masuk (`cd`) |

---

## 📘 Bagian 3 — Octal Digits (Numeric Mode)

### Konversi Permission ke Angka

| Permission | Angka |
|---|---|
| Read | **4** |
| Write | **2** |
| Execute | **1** |

### Kombinasi Umum

| Angka | Binary | Symbolic | Arti |
|---|---|---|---|
| 0 | 000 | `---` | No permission |
| 1 | 001 | `--x` | Execute only |
| 2 | 010 | `-w-` | Write only |
| 3 | 011 | `-wx` | Write + execute |
| 4 | 100 | `r--` | Read only |
| 5 | 101 | `r-x` | Read + execute |
| 6 | 110 | `rw-` | Read + write |
| 7 | 111 | `rwx` | Full access |

### Contoh Penerapan

```bash
# rw-r--r-- (owner rw, group r, others r)
$ chmod 644 file1

# rwxr-xr-x (owner rwx, group rx, others rx)
$ chmod 755 file1

# rwx------ (owner full, others none)
$ chmod 700 file1
```

---

## 📘 Bagian 4 — Bagaimana Kernel Memeriksa Permission

Ketika user mencoba akses file, kernel memeriksa permission **berurutan**:

1. **Owner permission** — kalau user adalah owner, pakai bit owner.
2. **Group permission** — kalau bukan owner tapi anggota group, pakai bit group.
3. **Others permission** — kalau bukan keduanya, pakai bit others.

**Penting:** Kernel hanya menerapkan **satu set permission** per akses. Jadi kalau user adalah owner, permission group-nya **tidak berlaku** meskipun lebih longgar.

**Contoh:**

```
-rw------- owner:user1 group:dev
```

- `user1` (owner) → bisa read + write.
- User lain di group `dev` → **tidak bisa apa-apa** (karena bukan owner, dan group permission `---`).
- User di luar group → juga tidak bisa.

---

## 📘 Bagian 5 — Special Permissions

Selain `rwx` standar, Linux mendukung **special permission**:

| Special | Simbol | Efek |
|---|---|---|
| **SUID** | `s` (di posisi user) | Executable dijalankan sebagai **owner**, bukan user yang menjalankannya |
| **SGID** | `s` (di posisi group) | Executable dijalankan sebagai **group**; atau direktori baru mewarisi group |
| **Sticky Bit** | `t` (di posisi others) | Hanya owner file yang bisa hapus file di direktori (umum di `/tmp`) |

**Contoh klasik:** `/usr/bin/passwd` punya SUID (`-rwsr-xr-x`), sehingga user biasa bisa ubah password meskipun file `/etc/shadow` dimiliki root.

Akan dipelajari di lab terpisah.

---

## 📘 Bagian 6 — `chmod` — Symbolic vs Octal

### Symbolic Mode

**Syntax:** `chmod [class][operator][permissions] file`

| Class | Arti |
|---|---|
| `u` | User (owner) |
| `g` | Group |
| `o` | Others |
| `a` | All (u+g+o) |

| Operator | Fungsi |
|---|---|
| `+` | Tambah |
| `-` | Hapus |
| `=` | Set (reset) |

**Contoh:**

```bash
$ ls -l a_file
-rw-rw-r-- 1 coop coop 1601 Mar 9 15:04 a_file

$ chmod uo+x,g-w a_file
$ ls -l a_file
-rwxr--r-x 1 coop coop 1601 Mar 9 15:04 a_file
```

**Breakdown:**
- `uo+x` → tambah execute untuk user & others.
- `g-w` → hapus write dari group.

### Octal Mode

```bash
$ chmod 755 file   # rwxr-xr-x
$ chmod 644 file   # rw-r--r--
$ chmod 700 file   # rwx------
```

### Kapan Pakai Apa?

| Aspek | Symbolic | Octal |
|---|---|---|
| Kejelasan | ✅ Eksplisit | ❌ Harus hafal |
| Kecepatan | ❌ Lebih panjang | ✅ Singkat |
| Granularitas | ✅ Ubah sebagian | ❌ Set semua |
| Umum dipakai | Tweak kecil | Set standar |

---

## 📘 Bagian 7 — `chown` & `chgrp`

Setiap file punya **owner** dan **group**. Kadang perlu diubah — misal saat transfer file antar user atau setup kolaborasi tim.

| Perintah | Fungsi | Siapa yang boleh |
|---|---|---|
| `chown` | Ubah owner (dan opsional group) | Hanya root |
| `chgrp` | Ubah group | Root, atau user anggota group tersebut |

### Contoh `chown`

```bash
# Ubah owner
$ sudo chown wally somefile

# Ubah owner + group
$ sudo chown wally:cleavers somefile

# Rekursif (direktori + isinya)
$ sudo chown -R wally:cleavers ./
$ sudo chown -R wally:wally subdir
```

**Catatan:** Pemisah owner dan group bisa `:` (modern) atau `.` (lama).

### Contoh `chgrp`

```bash
$ sudo chgrp cleavers somefile
```

### Kapan Pakai `chown` vs `chmod`?

| Kebutuhan | Pakai |
|---|---|
| Ubah **siapa** yang memiliki file | `chown` / `chgrp` |
| Ubah **apa** yang boleh dilakukan | `chmod` |

---

## 📘 Bagian 8 — Umask

### Konsep

Saat file/direktori baru dibuat, permission awalnya **tidak langsung** dipakai. Ada filter bernama **umask** yang menentukan permission mana yang **ditolak**.

**Default sebelum umask:**

| Tipe | Default |
|---|---|
| File | `0666` (rw-rw-rw-) |
| Direktori | `0777` (rwxrwxrwx) |

**Catatan:** File **tidak pernah** dibuat dengan execute secara default, bahkan tanpa umask.

### Cara Kerja Umask

Rumus:

```
Final = Default & ~umask
```

**Contoh dengan umask `0002`:**

**File:**
- Default: `0666`
- Umask: `0002`
- Invert umask: `0775`
- `0666 & 0775 = 0664` → `rw-rw-r--`

**Direktori:**
- Default: `0777`
- Umask: `0002`
- Invert umask: `0775`
- `0777 & 0775 = 0775` → `rwxrwxr-x`

### Cek Umask Saat Ini

```bash
$ umask
0002
```

### Umask Umum

| Umask | File | Direktori | Kapan dipakai |
|---|---|---|---|
| `022` | `644` | `755` | Default banyak distro (aman) |
| `002` | `664` | `775` | Kolaborasi group |
| `077` | `600` | `700` | Private ketat |
| `027` | `640` | `750` | Server produksi (group read-only) |

### Ubah Umask

```bash
$ umask 0022
```

Berlaku untuk sesi shell saat ini. Untuk permanent, tambahkan di `~/.bashrc` atau `/etc/profile`.

**Tips:** Di server produksi, umask `027` sering dipakai — file jadi `640`, direktori `750`. Ini mencegah "others" melihat apapun.

---

## 📘 Bagian 9 — Filesystem ACL

### Keterbatasan Permission Tradisional

Permission tradisional hanya punya **tiga slot**:
- Satu owner
- Satu group
- Others

**Masalah:** Bagaimana kalau butuh memberi akses ke **beberapa user spesifik** dengan permission berbeda-beda? Permission tradisional tidak bisa.

**Solusi:** **POSIX ACL** (Access Control List).

### Cara Kerja ACL

- ACL memperluas model permission standar.
- Harus didukung filesystem (ext4, xfs, btrfs support).
- Bisa diaktifkan saat mount dengan `-o acl`, tapi biasanya sudah default di distro modern.
- Setiap file/direktori bisa punya banyak **entry ACL** tambahan.

### Melihat ACL

```bash
$ getfacl /home/stephane/file1
```

Output:

```
# file: home/stephane/file1
# owner: stephane
# group: stephane
user::rw-
user:isabelle:r-x
group::r--
mask::r-x
other::---
```

### Mengelola ACL

```bash
# Beri akses read+execute ke isabelle
$ setfacl -m u:isabelle:rx /home/stephane/file1

# Hapus ACL isabelle
$ setfacl -x u:isabelle /home/stephane/file1

# ACL untuk group
$ setfacl -m g:projectteam:rw /project/file2
```

### Default ACL

Direktori bisa punya **default ACL** yang diwarisi file baru:

```bash
$ setfacl -m d:u:isabelle:rx somedir
```

Sekarang, semua file baru di `somedir/` otomatis memberi akses `r-x` ke `isabelle`.

### Preservation of ACL

| Perintah | Apakah ACL dipreservasi? |
|---|---|
| `mv` | ✅ Ya |
| `cp` | ❌ Tidak |
| `cp -p` | ✅ Ya |
| `rsync -A` | ✅ Ya |
| `tar --acls` | ✅ Ya |

### Keuntungan ACL

- **Tidak perlu permission terlalu longgar** (seperti 777).
- **Kolaborasi presisi** antara banyak user & group.
- **Integrasi** dengan model permission Unix tradisional.

### Keterbatasan ACL

- Tidak semua filesystem support.
- Bisa membingungkan kalau terlalu banyak entry.
- **Bukan security tool sejati** — untuk keamanan maksimal, pakai SELinux/AppArmor.
- Perlu maintenance manual (misal saat user dihapus).

---

## 🐛 Troubleshooting & Kesalahan

- **Permission tidak bekerja seperti yang diharapkan:** Cek urutan pemeriksaan kernel — owner dulu, lalu group, lalu others. Hanya **satu set** yang berlaku.

- **File baru permission-nya "aneh":** Cek `umask`. Umask menentukan permission default file baru.

- **`chown` gagal "Operation not permitted":** Hanya root yang bisa `chown`. Untuk `chgrp`, user harus anggota group tujuan.

- **`setfacl: command not found`:** Install paket `acl`:
  ```bash
  $ sudo apt install acl -y
  ```

- **ACL tidak bekerja meskipun sudah di-set:** Cek **mask**. Kalau mask tidak mengizinkan, ACL akan dibatasi. Lihat `mask::rw-` di output `getfacl`.

- **`ls -l` menampilkan `+`:** Tanda file punya ACL. Untuk lihat detailnya, gunakan `getfacl`.

- **File jadi tidak bisa diakses setelah `chmod`:** Mungkin kamu menghapus permission `x` dari direktori, atau `chmod o=` menghapus akses others. Cek dengan `ls -l`.

- **File baru tidak mewarisi group direktori:** Set **SGID** di direktori:
  ```bash
  $ chmod g+s direktori/
  ```

- **User tidak bisa masuk direktori meskipun punya `r`:** Butuh `x` di direktori untuk `cd`. `r` saja hanya bisa `ls`.

- **`chmod` pada symlink mengubah target:** Gunakan `chmod -h` di sistem yang mendukung, atau ubah permission target secara langsung.

- **ACL hilang setelah `cp`:** Gunakan `cp -p` (preserve) atau `rsync -A` untuk menjaga ACL.

---

## 💡 Catatan & Insight Pribadi

### Urutan Prioritas Permission

> **Owner > Group > Others.**
> Kalau user adalah owner, permission group-nya **diabaikan**.

Ini sering bikin bingung pemula: "Kenapa saya tidak bisa akses padahal saya anggota group?" — mungkin karena kamu adalah owner, dan permission owner-nya `---`.

### Umask dan Default Permission

Umask **bukan** permission — dia **filter**. File baru permission-nya `default & ~umask`. Jadi:
- Umask `022` → file `644`, direktori `755`.
- Umask `002` → file `664`, direktori `775`.
- Umask `077` → file `600`, direktori `700`.

**Tips:** Di server produksi, banyak yang pakai `027` — file `640`, direktori `750`. Ini mencegah "others" melihat data.

### Kapan Pakai `chmod`, `chown`, atau `chgrp`?

| Tujuan | Perintah |
|---|---|
| Ubah permission (siapa boleh apa) | `chmod` |
| Ubah owner file | `chown` |
| Ubah group file | `chgrp` |
| Ubah owner + group | `chown user:group` |

### Hierarki Keamanan File

1. **Permission tradisional** (rwx) — dasar.
2. **Special permission** (SUID, SGID, sticky) — tambahan.
3. **ACL** — granular per user/group.
4. **SELinux/AppArmor** — mandatory access control (level lanjut).
5. **Container/chroot** — isolasi penuh.

Semakin ke bawah, semakin kuat, tapi semakin kompleks.

### Best Practice

- **Jangan pakai `chmod 777`.** Itu sama dengan membuka pintu untuk siapa saja.
- **Gunakan `chmod 644` untuk file data**, `755` untuk script, `700` untuk direktori private.
- **Pakai ACL** kalau butuh granular. Jangan `chmod 777` hanya karena malas.
- **Pakai `umask 027`** di server produksi.
- **Backup ACL** sebelum migrasi: `getfacl -R /path > acl.txt`.

### Kasus Nyata

- **Web server:** File HTML `644`, direktori `755`. Owner `www-data`.
- **SSH key:** `~/.ssh/id_rsa` **harus** `600` — kalau tidak, SSH menolak.
- **Shared folder tim:** Direktori `775` + SGID, group dengan anggota tim.
- **Backup user:** ACL memberi akses read ke user `backup` tanpa jadi owner.
- **File konfigurasi rahasia:** `600` atau `640`.

### Pelajaran Kunci dari Teori Ini

1. **Permission diperiksa berurutan:** owner → group → others. Hanya satu yang berlaku.
2. **Umask menentukan default permission** file/direktori baru.
3. **`chmod`, `chown`, `chgrp`** punya peran berbeda — jangan tertukar.
4. **ACL menyelesaikan masalah permission granular** yang tidak bisa diselesaikan permission tradisional.
5. **`chmod 777` bukan solusi** — itu masalah.
6. **Special permission** (SUID, SGID, sticky bit) punya kasus penggunaan spesifik — pahami sebelum pakai.

---

## 🧠 Perintah & Konsep yang Dikuasai

| Perintah / Konsep | Fungsi |
|---|---|
| `ls -l` | Lihat permission + owner + group |
| `chmod` | Ubah permission (symbolic atau octal) |
| `chown` | Ubah owner file |
| `chgrp` | Ubah group file |
| `umask` | Lihat/set umask |
| `getfacl` | Lihat ACL |
| `setfacl` | Set/modifikasi ACL |
| `setfacl -x` | Hapus ACL user/group |
| `setfacl -b` | Hapus semua ACL |
| `setfacl -d` | Set default ACL |
| `chmod u+s` / `g+s` / `+t` | Set SUID / SGID / sticky bit |

**Tabel konversi cepat:**

| Symbolic | Octal |
|---|---|
| `rwx` | 7 |
| `rw-` | 6 |
| `r-x` | 5 |
| `r--` | 4 |
| `-wx` | 3 |
| `-w-` | 2 |
| `--x` | 1 |
| `---` | 0 |

---

## 📌 Kesimpulan

Teori ini memberikan fondasi untuk semua lab permission (13.1–13.3) dan akan terus relevan sepanjang karier sysadmin. Yang paling penting dipahami:

1. **Permission adalah fondasi keamanan Linux** — setiap akses divalidasi kernel.
2. **Ada hierarki pemeriksaan:** owner → group → others. Hanya satu yang berlaku.
3. **Umask memfilter default permission** file/direktori baru.
4. **ACL mengatasi keterbatasan** permission tradisional untuk kasus granular.
5. **Special permission** (SUID, SGID, sticky) punya kasus spesifik — jangan sembarangan pakai.

Kemampuan mengelola permission dengan benar adalah **garis pemisah antara sysadmin pemula dan berpengalaman**. Kesalahan kecil seperti `chmod 777` bisa berujung pada kompromi sistem. Sebaliknya, permission yang terlalu ketat bisa membuat user tidak bisa bekerja.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
