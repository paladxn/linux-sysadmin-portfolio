# Lab 5.1 — Process Termination dengan `kill`, `killall` & `pkill`

**Course:** Linux System Administration (Adinusa)
**Topic:** Process management & termination
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membuat dan menjalankan **background script** sederhana
- Memonitor dan memverifikasi **running processes**
- Menghentikan proses menggunakan `kill`, `killall`, dan `pkill`
- Memahami perbedaan ketiga perintah tersebut dalam konteks nyata

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-005-1
```

Login menggunakan kredensial akun Adinusa, lalu jalankan script persiapan lab.

---

## 📘 Guided Example

### 1. Membuat direktori kerja

```bash
$ mkdir lab5
```

### 2. Membuat script `killing` dengan vim

```bash
$ vim /home/student/lab5/killing
```

Isi script:

```bash
#!/bin/bash
while true; do
   echo -n "$@ " >> ~/lab5/killing_outfile
   sleep 5
done
```

**Penjelasan:**
- `while true; do ... done` → loop tanpa henti.
- `echo -n "$@ "` → menulis argumen yang diberikan (tanpa newline), diikuti spasi.
- `>> ~/lab5/killing_outfile` → **append** ke file (tidak menimpa).
- `sleep 5` → jeda 5 detik setiap iterasi.
- Script akan terus berjalan sampai dihentikan secara eksternal.

### 3. Membuat script executable

```bash
$ chmod +x /home/student/lab5/killing
```

### 4. Menjalankan script di background

```bash
$ ~/lab5/killing hello &
```

Tanda `&` di akhir perintah menjalankannya di **background**, sehingga terminal tetap bisa dipakai.

**Cek output setelah ~10 detik:**

```bash
$ cat ~/lab5/killing_outfile
hello hello hello
```

**Cek proses berjalan:**

```bash
$ ps aux | grep killing
```

### 5. Menghentikan script dengan `killall`

```bash
$ killall killing
```

`killall` menghentikan **semua proses** dengan nama yang cocok.

---

## 🧪 Practice Task

### Soal

1. Jalankan script `killing` di background dengan argumen `network`.
2. Jalankan instance kedua dengan argumen `interface`, verifikasi keduanya berjalan.
3. Jalankan instance ketiga dengan argumen `connection`, sehingga ada tiga proses berjalan.
4. Gunakan `kill` untuk menghentikan proses dengan argumen `network`, verifikasi hanya `interface` dan `connection` yang tersisa.
5. Gunakan `pkill` untuk menghentikan proses dengan argumen `interface`, verifikasi hanya `connection` yang tersisa.

### ✅ Solusi

```bash
# 1. Jalankan instance pertama (network)
$ ~/lab5/killing network &

# 2. Jalankan instance kedua (interface)
$ ~/lab5/killing interface &

# 3. Jalankan instance ketiga (connection)
$ ~/lab5/killing connection &

# Verifikasi: ketiga proses harus muncul
$ ps aux | grep killing
student   1234  ...  /bin/bash /home/student/lab5/killing network
student   1235  ...  /bin/bash /home/student/lab5/killing interface
student   1236  ...  /bin/bash /home/student/lab5/killing connection

# 4. Hentikan proses "network" dengan kill (butuh PID)
$ pgrep -f "killing network"
1234
$ kill 1234

# Verifikasi: hanya interface & connection yang tersisa
$ ps aux | grep killing

# 5. Hentikan proses "interface" dengan pkill (cocokkan pola nama)
$ pkill -f "killing interface"

# Verifikasi: hanya connection yang tersisa
$ ps aux | grep killing
```

### 🔍 Hasil Akhir yang Diharapkan

Setelah semua langkah:

```bash
$ ps aux | grep killing
student   1236  ...  /bin/bash /home/student/lab5/killing connection
```

Hanya proses **connection** yang masih berjalan dan terus menulis ke `killing_outfile`.

Isi file `~/lab5/killing_outfile` (contoh):

```
network network network interface interface interface connection connection connection
```

Setelah beberapa proses dihentikan, hanya `connection` yang terus bertambah:

```
... connection connection connection connection
```

### 🧹 Membersihkan Sisa Proses

Setelah selesai praktik, hentikan proses `connection`:

```bash
$ killall killing
# atau
$ pkill -f "killing connection"
```

---

## 🐛 Troubleshooting & Kesalahan

- **Salah pakai `&` vs `nohup`:** Saat pertama menjalankan `~/lab5/killing network &`, saya pikir proses akan tetap jalan meskipun terminal ditutup. Ternyata dengan `&` saja, proses masih terikat ke terminal (child process). Kalau terminal ditutup, proses bisa ikut mati. Untuk menjalankan proses yang benar-benar terlepas dari terminal, perlu `nohup ... &` atau `disown`.

- **`pgrep` vs `ps aux | grep`:** Awalnya saya pakai `ps aux | grep killing` untuk cari PID, tapi outputnya termasuk baris `grep killing` itu sendiri (karena `grep` juga mengandung kata "killing"). Lebih bersih pakai `pgrep -f killing` yang langsung mengembalikan PID saja.

- **`pkill` cocok dengan terlalu banyak proses:** Saat menjalankan `pkill -f killing`, saya sempat khawatir semua instance terhenti. Ternyata `pkill` tanpa filter tambahan memang menghentikan semua yang cocok. Setelah itu saya ubah jadi `pkill -f "killing interface"` agar hanya menargetkan instance tertentu. Pelajaran: **selalu gunakan pola yang spesifik** kalau ada beberapa proses dengan nama sama.

- **`kill` butuh PID, `pkill` butuh pola:** Perbedaan mendasar ini bikin saya sempat salah pakai `kill killing network` — ternyata `kill` tidak menerima nama proses, harus PID. `killall` dan `pkill` yang menerima nama/pola.

- **Sinyal default adalah `SIGTERM`:** Saat `kill 1234`, kernel mengirim `SIGTERM` (sinyal 15) yang meminta proses berhenti dengan sopan. Kalau proses tidak merespons, baru pakai `kill -9 1234` (`SIGKILL`) yang memaksa. Saya biasakan pakai `SIGTERM` dulu karena lebih "bersih" — proses punya kesempatan menyimpan state dan menutup file dengan benar.

---

## 💡 Catatan & Insight Pribadi

- **Perbedaan `kill`, `killall`, `pkill`:**

  | Perintah | Argumen | Cakupan |
  |---|---|---|
  | `kill` | PID | Proses tertentu |
  | `killall` | Nama proses | Semua proses dengan nama persis itu |
  | `pkill` | Pola (regex) | Semua proses yang cocok dengan pola |

- **`$@` di dalam script:** Variabel `$@` merepresentasikan **semua argumen** yang diberikan ke script. Jadi `killing network` akan menulis `network `, sedangkan `killing hello world` akan menulis `hello world `.

- **Background process (`&`) vs foreground:** Menjalankan proses di background dengan `&` membuat terminal tetap bisa dipakai untuk perintah lain. Namun, job tetap terikat ke sesi shell.

- **`jobs`, `fg`, `bg`:** Perintah bawaan shell untuk manajemen job. `jobs` menampilkan daftar job, `fg %1` membawa job ke foreground, `bg %1` melanjutkan job di background. Ini berguna saat kita lupa memberi `&`.

- **Sinyal penting:**
  - `SIGTERM` (15) — permintaan berhenti dengan sopan (default `kill`).
  - `SIGKILL` (9) — paksa berhenti, tidak bisa di-handle proses.
  - `SIGHUP` (1) — hangup, sering dipakai untuk reload konfigurasi (misal `nginx -s reload`).
  - `SIGINT` (2) — interrupt, dikirim saat `CTRL + C`.

- **Kasus nyata:** Di server produksi, `pkill -f` sering dipakai di script deployment untuk menghentikan proses lama sebelum menjalankan versi baru. Namun, bahaya utamanya adalah kalau polanya terlalu umum, bisa mematikan proses yang tidak seharusnya. Selalu test dengan `pgrep -f "pola"` dulu sebelum `pkill -f "pola"`.

- **Hati-hati `pkill -f` dengan pola luas:** Menjalankan `pkill -f bash` bisa mematikan **semua shell bash** milik user, termasuk sesi SSH-mu sendiri. Ini pengalaman yang tidak menyenangkan, dan mengajarkan saya untuk selalu spesifik.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `command &` | Jalankan proses di background |
| `ps aux \| grep nama` | Cari proses berdasarkan nama |
| `pgrep -f pola` | Dapatkan PID proses berdasarkan pola |
| `kill PID` | Kirim sinyal ke proses (default `SIGTERM`) |
| `kill -9 PID` | Paksa hentikan proses (`SIGKILL`) |
| `killall nama` | Hentikan semua proses dengan nama persis |
| `pkill -f pola` | Hentikan proses berdasarkan pola |
| `jobs` | Daftar job di shell saat ini |
| `fg` / `bg` | Pindah job ke foreground / background |

---

## 📌 Kesimpulan

Lab ini memperkenalkan **process management** dari sisi praktis: bagaimana proses dijalankan di background, bagaimana cara mengidentifikasi PID-nya, dan bagaimana menghentikannya dengan tepat. Kemampuan ini sangat penting di dunia sysadmin karena sering kali kita perlu menghentikan proses yang **hang**, **runaway**, atau sudah tidak diperlukan. Memahami perbedaan antara `kill`, `killall`, dan `pkill` — serta risiko masing-masing — adalah bekal wajib sebelum menyentuh server produksi.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.