# Lab 7.1 — Monitoring CPU & Load Average dengan `top`

**Course:** Linux System Administration (Adinusa)
**Topic:** Process states, CPU usage & load average
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membuat **controlled CPU workload** menggunakan script sederhana
- Memonitor **process states**, **%CPU**, dan **load average** menggunakan `top`
- Memahami hubungan antara jumlah proses, jumlah **logical CPU** (dari `lscpu`), dan **system load**

---

## 📘 Konsep Dasar

Sebelum praktik, ada beberapa konsep penting yang perlu dipahami:

| Konsep | Penjelasan |
|---|---|
| **Load Average** | Rata-rata jumlah proses yang menunggu CPU (running + waiting) dalam 1, 5, dan 15 menit terakhir |
| **Logical CPU** | Jumlah core yang terlihat oleh sistem (bisa lebih banyak dari physical core karena hyper-threading) |
| **%CPU** | Persentase waktu CPU yang dipakai oleh sebuah proses |
| **Process State** | Status proses: `R` (running), `S` (sleeping), `D` (uninterruptible sleep), `Z` (zombie), `T` (stopped) |

**Aturan penting:**
> Load average **1.0** pada sistem **single CPU** berarti CPU sudah 100% terpakai. Pada sistem 4 CPU, load **4.0** berarti semua CPU penuh. Jadi load harus dibandingkan dengan jumlah logical CPU.

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-007-1
```

---

## 📘 Guided Example

### 1. Membuat direktori kerja

```bash
$ mkdir /home/student/lab7
```

### 2. Membuat script `monitor` dengan vim

```bash
$ vim /home/student/lab7/monitor
```

Isi script:

```bash
#!/bin/bash
while true; do
    var=1
    while [[ $var -lt 50000 ]]; do
        var=$((var+1))
    done
    sleep 1
done
```

**Penjelasan:**
- Loop luar `while true` → jalan tanpa henti sampai dihentikan.
- Loop dalam `while [[ $var -lt 50000 ]]` → melakukan ~50.000 operasi penjumlahan sebagai beban CPU.
- `sleep 1` → jeda 1 detik sebelum iterasi berikutnya.
- Script ini menghasilkan beban CPU yang **terkendali**, tidak membahayakan sistem.

### 3. Membuat script executable

```bash
$ chmod a+x /home/student/lab7/monitor
```

Opsi `a+x` memberi izin eksekusi untuk **semua user** (owner, group, others).

### 4. Membuka `top`

```bash
$ top
```

### 5. Cek jumlah logical CPU

Di terminal terpisah:

```bash
$ lscpu
```

Contoh output (sebagian):

```
Architecture:           x86_64
CPU(s):                 1
...
```

Catat angka **CPU(s)** — ini yang jadi acuan untuk menafsirkan load average. Untuk lab ini, diasumsikan **1 CPU**.

### 6. Menjalankan instance pertama script

```bash
$ /home/student/lab7/monitor &
[1] 6071
```

Tanda `&` menjalankannya di background. Angka `6071` adalah PID.

### 7. Mengamati `top`

Di dalam `top`, aktifkan semua header dengan tombol:

| Tombol | Fungsi |
|---|---|
| `l` | Toggle baris **load average** |
| `t` | Toggle baris **tasks/threads** |
| `m` | Toggle baris **memory** |

Cari proses `monitor` di daftar:

```
PID   USER   PR  NI  VIRT  RES  SHR S  %CPU  %MEM  TIME+  COMMAND
26549 student 20   0  7892 3808 3352 S 13.6   0.2   0:04.8 monitor
```

Perhatikan kolom `%CPU`. Contoh load average saat 1 instance:

```
top - 12:23:45 up 11 days, 1:09, 3 users, load average: 0.21, 0.14, 0.05
```

### 8. Menjalankan instance kedua

```bash
$ /home/student/lab7/monitor &
[2] 6498
```

Tunggu **minimal 1 menit**, lalu perhatikan **1-minute load average**:

```
top - 12:27:39 up 11 days, 1:13, 3 users, load average: 0.36, 0.25, 0.11
```

Nilai naik dari 0.21 → 0.36 setelah instance kedua dijalankan.

### 9. Menjalankan instance ketiga dan seterusnya

```bash
$ /home/student/lab7/monitor &
[3] 6881

$ /home/student/lab7/monitor &
[4] 10708

$ /home/student/lab7/monitor &
[5] 11122

$ /home/student/lab7/monitor &
[6] 11338
```

Setelah ~1 menit, cek load average:

```
top - 12:42:32 up 11 days, 1:28, 3 users, load average: 1.23, 2.50, 1.54
```

Load melewati **1.0** — artinya sistem (1 CPU) sudah kewalahan.

---

## 🧪 Practice Task

### Soal

1. Jalankan **1 instance** `monitor`, catat **1-minute load average** setelah 1 menit.
2. Tambahkan instance **kedua**, catat lagi load average-nya.
3. Tambahkan instance **ketiga**, catat lagi.
4. Terus tambahkan instance sampai **1-minute load average > 1.0** (pada sistem 1 CPU).
5. Hentikan semua instance `monitor` setelah selesai.

### ✅ Solusi

```bash
# 1. Jalankan instance pertama
$ /home/student/lab7/monitor &
$ sleep 60
$ top -bn1 | head -5
# Catat load average

# 2. Instance kedua
$ /home/student/lab7/monitor &
$ sleep 60
$ top -bn1 | head -5

# 3. Instance ketiga
$ /home/student/lab7/monitor &
$ sleep 60
$ top -bn1 | head -5

# 4. Tambah sampai load > 1.0 (untuk 1 CPU)
$ /home/student/lab7/monitor &
$ /home/student/lab7/monitor &
$ /home/student/lab7/monitor &
$ sleep 60
$ top -bn1 | head -5

# 5. Bersihkan semua proses monitor
$ killall monitor
# atau
$ pkill -f "/home/student/lab7/monitor"
```

### 🔍 Hasil Akhir yang Diharapkan

Tabel contoh hasil pengamatan:

| Jumlah Instance | 1-min Load Avg | Catatan |
|---|---|---|
| 1 | ~0.2 – 0.3 | CPU belum penuh |
| 2 | ~0.4 – 0.6 | Mulai naik |
| 3 | ~0.7 – 1.0 | Nyaris penuh |
| 6 | > 1.0 | CPU overloaded |

Verifikasi tidak ada proses `monitor` yang tersisa:

```bash
$ ps aux | grep [m]onitor
# (tidak ada output)
```

---

## 🐛 Troubleshooting & Kesalahan

- **Load average tidak langsung naik:** Saya sempat bingung kenapa setelah menjalankan instance kedua, nilai `load average` masih rendah. Ternyata load average dihitung sebagai **rata-rata bergerak** dalam 1 menit, jadi butuh **waktu ~1 menit** untuk benar-benar mencerminkan kondisi saat ini. Solusinya: tunggu minimal 60 detik sebelum membaca ulang.

- **Salah tafsir load average:** Saya awalnya mengira `load average: 1.0` itu berarti CPU 100% terpakai. Ternyata ini hanya benar untuk sistem **1 CPU**. Di sistem 4 CPU, load `1.0` berarti hanya 25% CPU terpakai. Solusinya: **selalu cek dulu `lscpu`** atau `nproc` untuk tahu jumlah CPU sebelum menafsirkan load.

- **`top` di dalam `top`:** Saat pertama kali membuka `top`, saya menjalankan instance `monitor` dari terminal lain. Ternyata lebih praktis kalau `top` dibuka di satu terminal dan `monitor` dijalankan di terminal lain, sehingga bisa melihat perubahan `%CPU` secara real-time.

- **`%CPU` naik-turun:** Kolom `%CPU` untuk proses `monitor` tampak tidak stabil (kadang 13.6%, kadang 40%). Ini wajar karena script melakukan loop komputasi lalu `sleep 1` — jadi beban CPU-nya bergelombang. Kalau ingin beban yang lebih stabil, `sleep` bisa dikurangi atau dihilangkan (tapi hati-hati, bisa membuat sistem panas).

- **Lupa membersihkan proses:** Setelah selesai praktik, saya lupa menghentikan proses `monitor`, sehingga VM tetap berjalan dengan beban tinggi. Solusinya: biasakan **selalu `killall monitor`** setelah selesai praktik, atau cek dengan `ps aux | grep [m]onitor`.

---

## 💡 Catatan & Insight Pribadi

- **Load average vs %CPU:** Load average mengukur **panjang antrean** proses yang ingin menggunakan CPU. Kalau load > jumlah CPU, berarti ada proses yang mengantre. Kalau load < jumlah CPU, CPU masih ada ruang.

- **3 angka pada load average:**
  - Angka pertama: rata-rata **1 menit terakhir** (paling sensitif terhadap perubahan).
  - Angka kedua: rata-rata **5 menit terakhir**.
  - Angka ketiga: rata-rata **15 menit terakhir** (paling stabil).

  Dengan membandingkan ketiganya, kita bisa tahu **tren**: apakah beban naik, turun, atau stabil.

- **Tombol berguna di `top`:**
  | Tombol | Fungsi |
  |---|---|
  | `l` | Toggle load average |
  | `t` | Toggle tasks/threads |
  | `m` | Toggle memory |
  | `P` | Sort by %CPU |
  | `M` | Sort by %MEM |
  | `1` | Tampilkan per-CPU (multi-core) |
  | `k` | Kill proses (masukkan PID) |
  | `q` | Keluar |

- **`top -bn1`:** Menjalankan `top` dalam mode **batch** satu iterasi, cocok untuk scripting atau logging. Sangat berguna kalau ingin capture load average ke file:
  ```bash
  $ top -bn1 | head -5 >> /var/log/load-monitor.log
  ```

- **Alternatif `htop`:** Lebih ramah pengguna, ada warna, bisa scroll horizontal. Tapi `top` hampir selalu tersedia bahkan di sistem minimal.

- **Proses `sleep` juga proses:** Meskipun script di lab ini sebagian besar waktunya di `sleep`, saat bangun dan menjalankan loop 50.000 iterasi, dia mengonsumsi CPU. Itu sebabnya `%CPU` muncul signifikan.

- **Kasus nyata:** Di server produksi, **load average tinggi** adalah salah satu alarm pertama yang muncul. Kalau load terus di atas jumlah CPU dalam waktu lama, artinya sistem kekurangan resource. Solusinya bisa menaikkan CPU, optimasi aplikasi, atau membatasi proses (lihat kembali Lab 4.2 tentang `ulimit` dan `nproc`).

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `mkdir /path` | Membuat direktori |
| `vim file` | Edit file dengan vim |
| `chmod a+x file` | Beri izin eksekusi untuk semua user |
| `lscpu` | Info CPU (jumlah, arsitektur, dll) |
| `nproc` | Jumlah logical CPU |
| `top` | Monitor proses real-time |
| `command &` | Jalankan proses di background |
| `top -bn1` | Mode batch 1 iterasi |
| `killall monitor` | Hentikan semua proses `monitor` |
| `ps aux \| grep [m]onitor` | Cek proses `monitor` yang masih jalan |

---

## 📌 Kesimpulan

Lab ini memberikan pemahaman praktis tentang **CPU load** dan bagaimana ia berhubungan dengan jumlah CPU. Konsep **load average** sangat penting dalam monitoring server: load yang tinggi belum tentu masalah kalau CPU-nya banyak, dan sebaliknya. Dengan mengamati `top` dan membandingkan dengan `lscpu`, saya bisa mulai menilai apakah sistem dalam kondisi sehat atau tidak. Ini adalah fondasi untuk topik monitoring yang lebih dalam (seperti `vmstat`, `iostat`, `sar`, atau alat monitoring modern seperti Prometheus + Grafana).

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.