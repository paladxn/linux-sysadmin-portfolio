# Lab 4.2 — Konfigurasi Persistent Resource Limits

**Course:** Linux System Administration (Adinusa)
**Topic:** Persistent resource limits (group & user)
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Mengonfigurasi **persistent resource limits** untuk sebuah **group**
- Mengonfigurasi **persistent resource limits** untuk sebuah **user**
- Memahami perbedaan limit yang berlaku untuk group vs user
- Memastikan konfigurasi tetap aktif setelah **logout** dan **reboot**
- Membedakan antara `ulimit` (temporary) dan `limits.conf` (persistent)

---

## 📘 Konsep Dasar

Resource limits di Linux bisa diatur di dua level:

| Level | File Konfigurasi | Berlaku untuk |
|---|---|---|
| **Temporary** | `ulimit` (command) | Hanya shell saat ini |
| **Persistent (user)** | `/etc/security/limits.conf` | User tertentu atau group tertentu |
| **Persistent (systemd)** | Unit file / `system.conf` | Service yang dijalankan systemd |

Lab ini fokus pada **persistent limits** via `/etc/security/limits.conf`, yang dibaca oleh PAM saat user login.

### Format Entry

```
<domain>   <type>   <item>   <value>
```

| Field | Keterangan |
|---|---|
| `<domain>` | Username, `@groupname`, atau `*` (semua user) |
| `<type>` | `soft`, `hard`, atau `-` (keduanya) |
| `<item>` | Jenis resource: `nproc`, `nofile`, `core`, dll |
| `<value>` | Nilai limit |

---

## 🧪 Practice Task

### Soal

1. **Group resource limit**
   Konfigurasi group lokal `student` agar:
   - Jumlah maksimum proses dibatasi **100**.

2. **User resource limit**
   Konfigurasi user lokal `student` agar:
   - Maksimum open files:
     - **Soft limit = 2000**
     - **Hard limit = 3000**

3. **Persistence requirement**
   - Konfigurasi harus tetap berlaku setelah **logout** dan **reboot**.
   - **Tidak boleh** mengandalkan perintah `ulimit` temporary.

### ✅ Solusi

#### Langkah 1 — Edit file konfigurasi

```bash
$ sudo nano /etc/security/limits.conf
```

#### Langkah 2 — Tambahkan entry berikut di akhir file

```
# Group limit: max processes untuk group student
@student   hard   nproc   100

# User limit: max open files untuk user student
student    soft   nofile  2000
student    hard   nofile  3000
```

**Penjelasan tiap baris:**

| Baris | Arti |
|---|---|
| `@student hard nproc 100` | Group `student` dibatasi maksimum **100 proses** (hard limit) |
| `student soft nofile 2000` | User `student` punya soft limit **2000 open files** |
| `student hard nofile 3000` | User `student` tidak boleh menaikkan nofile melebihi **3000** |

#### Langkah 3 — Simpan file

```
CTRL + O → Enter → CTRL + X
```

#### Langkah 4 — Verifikasi konfigurasi

**A. Cek syntax file (opsional tapi disarankan):**

Pastikan tidak ada typo. Kamu bisa melihat kembali isinya:

```bash
$ sudo tail -n 10 /etc/security/limits.conf
```

**B. Logout dan login ulang sebagai `student`:**

```bash
$ exit              # keluar dari sesi student
$ su - student      # login baru sebagai student
```

**C. Cek hasil limit:**

```bash
# Cek limit proses (dari group)
$ ulimit -u
100

# Cek limit open files (dari user)
$ ulimit -n
2000

# Cek hard limit open files
$ ulimit -Hn
3000
```

#### Langkah 5 — Uji persistence

Reboot sistem, lalu login ulang sebagai `student`, dan cek lagi:

```bash
$ ulimit -u
100
$ ulimit -n
2000
```

Kalau nilainya tetap sama, konfigurasi sudah **persistent**. ✅

---

## 🔍 Hasil Akhir yang Diharapkan

Isi akhir `/etc/security/limits.conf` (bagian yang relevan):

```
# End of file

# Group limit: max processes untuk group student
@student   hard   nproc   100

# User limit: max open files untuk user student
student    soft   nofile  2000
student    hard   nofile  3000
```

Output verifikasi setelah login ulang:

```bash
$ ulimit -u
100

$ ulimit -n
2000

$ ulimit -Hn
3000
```

---

## 🐛 Troubleshooting & Kesalahan

- **Kesalahan pertama:** Saya sempat hanya menambahkan baris `hard` untuk user (`student hard nofile 3000`) tanpa `soft`. Akibatnya, saat login, `ulimit -n` tetap menunjukkan nilai default (1024), sedangkan `ulimit -Hn` = 3000. Ini membingungkan karena **soft limit** adalah yang aktif dipakai, sementara **hard limit** hanya batas atasnya. Solusinya: selalu set **keduanya** jika ingin hasil yang konsisten — soft untuk nilai aktif, hard untuk batas maksimum.

- **Group `student` vs user `student`:** Saya sempat bingung kenapa `ulimit -u` = 100 padahal user `student` tidak punya entry `nproc` secara eksplisit. Ternyata limit dari **group** (`@student`) juga berlaku untuk semua anggota group tersebut. Ini penting dicatat: kalau user berada di beberapa group dengan limit berbeda, limit yang **paling ketat** akan dipakai.

- **Perubahan tidak langsung berlaku:** Setelah edit `limits.conf`, saya coba cek `ulimit -n` di terminal yang sama, tapi nilainya belum berubah. Ternyata konfigurasi ini dibaca **saat login**, jadi harus **logout & login ulang** (atau `su - student`) agar berlaku. Reboot juga bisa jadi cara paling pasti untuk memverifikasi persistence.

- **User `root` dikecualikan:** Saat login sebagai root, limit `nproc` dan `nofile` tetap besar (tidak terpengaruh oleh konfigurasi `student`). Ini memang by design — root sengaja dikecualikan dari limit user agar tidak terkunci dari sistemnya sendiri.

- **Cek syntax `@` untuk group:** Kalau lupa menambahkan tanda `@` di depan nama group, maka `student` akan diartikan sebagai **username**, bukan **group**. Ini bisa menyebabkan konfigurasi tidak berlaku untuk anggota group yang lain.

---

## 💡 Catatan & Insight Pribadi

- **Perbedaan `soft` dan `hard`:**
  - `soft` = nilai yang **aktif** dipakai saat login.
  - `hard` = **batas atas** yang boleh dinaikkan user (soft tidak boleh melebihi hard).
  - Kalau hanya `hard` yang di-set, soft tetap default → bisa menyebabkan inkonsistensi.

- **`-` sebagai type:** Kalau kita menulis `-` di kolom type, artinya **soft dan hard disamakan**. Contoh:
  ```
  student   -   nproc   100
  ```
  Sama dengan menulis `soft nproc 100` + `hard nproc 100`.

- **Prioritas limit:** Kalau ada beberapa entry yang cocok untuk satu user (misal `*`, `@group`, dan `username`), maka:
  - Entry **lebih spesifik** (username) menang atas group.
  - Entry group menang atas wildcard `*`.
  - Kalau ada konflik, limit yang **paling ketat** yang dipakai.

- **`limits.conf` vs `limits.d/`:** Selain `/etc/security/limits.conf`, ada folder `/etc/security/limits.d/` yang bisa berisi file `.conf` tambahan. File di folder ini **dibaca setelah** `limits.conf`, jadi bisa menimpa nilai di sana. Di beberapa distro modern, konfigurasi default malah ditaruh di `limits.d/`, bukan di `limits.conf`.

- **Kapan perlu limit?** Contoh nyata: membatasi `nproc` untuk user yang menjalankan **cron job** atau **shell service**, agar kalau ada **fork bomb** atau bug yang menyebabkan proses spawn tak terkendali, sistem tidak langsung kehabisan resource.

- **Catatan soal systemd:** Untuk service yang dijalankan systemd (misal `nginx.service`), limit di `limits.conf` **tidak berlaku**. Harus diatur di unit file dengan `LimitNOFILE=`, `LimitNPROC=`, dll. Ini penting di server produksi.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `sudo nano /etc/security/limits.conf` | Edit konfigurasi limit persistent |
| `ulimit -n` | Cek soft limit open files |
| `ulimit -Hn` | Cek hard limit open files |
| `ulimit -u` | Cek limit proses |
| `su - student` | Login baru sebagai user (untuk apply limit) |
| `exit` | Logout dari sesi |

---

## 📌 Kesimpulan

Lab ini melengkapi pemahaman dari Lab 4.1 tentang `ulimit`. Kalau di Lab 4.1 kita belajar mengatur limit **temporary** untuk eksperimen, di Lab 4.2 kita belajar mengatur limit **persistent** yang benar-benar dipakai di sistem produksi. Kemampuan mengelola `limits.conf` adalah skill wajib sysadmin, terutama untuk men-tuning aplikasi server (web server, database) yang butuh file descriptor besar, sekaligus sebagai proteksi terhadap **resource exhaustion** (seperti fork bomb) dengan membatasi `nproc`.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.