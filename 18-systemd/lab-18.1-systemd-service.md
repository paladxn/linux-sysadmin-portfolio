# Lab 18.1 — Adding a New Startup Service with systemd

**Course:** Linux System Administration (Adinusa)
**Topic:** Membuat custom systemd service
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membuat **custom systemd service**
- **Start**, **stop**, dan **check status** service
- Melakukan **`daemon-reload`** setelah mengubah unit file
- **Enable** service saat boot
- Memverifikasi service melalui **log**

---

## 📘 Konsep Dasar

**systemd** = init system & service manager modern di Linux. Mengelola semua service (daemon) sejak boot pertama.

| Komponen | Fungsi |
|---|---|
| **Unit file** | Definisi service (`.service`) |
| **`systemctl`** | Perintah untuk kontrol service |
| **`daemon-reload`** | Baca ulang config setelah edit unit file |
| **`enable`** | Daftarkan service agar start saat boot |
| **`start`** | Jalankan service sekarang |

**Lokasi unit file:**

| Path | Fungsi |
|---|---|
| `/etc/systemd/system/` | Custom service (buatan sendiri) |
| `/lib/systemd/system/` | Service bawaan package |
| `/run/systemd/system/` | Runtime (sementara) |

> **Kenapa `/etc/systemd/system/`?** Ini prioritas tertinggi — service di sini akan menimpa service bawaan dengan nama sama.

### Tiga Section di Unit File

| Section | Isi |
|---|---|
| `[Unit]` | Metadata: description, dependency |
| `[Service]` | Cara jalankan service: ExecStart, ExecStop |
| `[Install]` | Kapan service start saat boot |

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-018-1
```

---

## 📘 Guided Example

### 1. Buat unit file service

```bash
$ sudo vim /etc/systemd/system/adinusa.service
```

Isi:

```ini
[Unit]
Description=adinusa
After=network.target

[Service]
ExecStart=/bin/sh -c '/bin/echo I am starting the adinusa service ; /bin/sleep 60'
ExecStop=/bin/echo I am stopping the adinusa service

[Install]
WantedBy=multi-user.target
```

**Penjelasan direktif:**

| Direktif | Fungsi |
|---|---|
| `Description` | Deskripsi service |
| `After=network.target` | Start setelah network siap |
| `ExecStart` | Perintah yang dijalankan saat start |
| `ExecStop` | Perintah yang dijalankan saat stop |
| `WantedBy=multi-user.target` | Start di multi-user mode (mirip runlevel 2 & 3) |

**Apa yang dilakukan service ini?**
1. Print `I am starting the adinusa service`
2. Sleep 60 detik
3. Setelah 60 detik, service selesai (exit)

### 2. Start service

```bash
$ sudo systemctl start adinusa.service
```

### 3. Cek status

```bash
$ sudo systemctl status adinusa.service
```

Output yang diharapkan:

```
● adinusa.service - adinusa
     Loaded: loaded (/etc/systemd/system/adinusa.service; disabled; preset: enabled)
     Active: active (running) since ...
   Main PID: 1234 (sh)
      Tasks: 2
     ...
```

**Cek juga dengan `systemctl is-active`:**

```bash
$ systemctl is-active adinusa.service
active
```

### 4. Stop service

```bash
$ sudo systemctl stop adinusa.service
```

Setelah stop, cek status:

```bash
$ sudo systemctl status adinusa.service
```

Status berubah jadi `inactive (dead)`.

### 5. Reload systemd setelah edit unit file

Kalau kamu mengubah file `.service`, systemd **tidak otomatis** membaca perubahan. Harus:

```bash
$ sudo systemctl daemon-reload
```

> **Wajib** setiap kali edit file `.service`. Tanpa ini, perubahan tidak berlaku.

### 6. Monitor log service

**Cara 1 — journalctl (modern):**

```bash
$ sudo journalctl -u adinusa.service -f
```

**Cara 2 — syslog (legacy):**

```bash
$ sudo tail -f /var/log/syslog
```

> **Tips:** `-f` = follow, tampilkan log real-time.

### 7. Enable service saat boot

```bash
$ sudo systemctl enable adinusa.service
```

Output:

```
Created symlink /etc/systemd/system/multi-user.target.wants/adinusa.service → /etc/systemd/system/adinusa.service.
```

Ini membuat symlink di `multi-user.target.wants/` — sehingga service otomatis start saat boot.

### 8. Reboot & verifikasi

```bash
$ sudo reboot
```

Setelah login ulang:

```bash
$ sudo systemctl status adinusa.service
```

Harus muncul `active (running)` — tanpa perlu start manual.

---

## 🧪 Practice Task

### Soal

1. Buat service baru `latihan.service` yang:
   - Print pesan "Service latihan berjalan"
   - Sleep 120 detik
2. Start, cek status, stop.
3. Enable service.
4. Cek log dengan `journalctl`.
5. Reboot, verifikasi auto-start.
6. Disable service.

### ✅ Solusi

```bash
# 1. Buat unit file
$ sudo vim /etc/systemd/system/latihan.service
```

Isi:

```ini
[Unit]
Description=Latihan Service
After=network.target

[Service]
ExecStart=/bin/sh -c '/bin/echo "Service latihan berjalan" ; /bin/sleep 120'
ExecStop=/bin/echo "Service latihan berhenti"

[Install]
WantedBy=multi-user.target
```

```bash
# 2. Reload & manage
$ sudo systemctl daemon-reload
$ sudo systemctl start latihan.service
$ sudo systemctl status latihan.service
$ sudo systemctl stop latihan.service

# 3. Enable
$ sudo systemctl enable latihan.service

# 4. Cek log
$ sudo journalctl -u latihan.service -n 20

# 5. Reboot
$ sudo reboot

# Setelah reboot — verifikasi
$ sudo systemctl status latihan.service

# 6. Disable
$ sudo systemctl disable latihan.service
$ sudo systemctl stop latihan.service
```

### 🔍 Hasil yang Diharapkan

- `systemctl status latihan.service` menampilkan `active (running)`.
- `systemctl is-enabled latihan.service` menampilkan `enabled`.
- Setelah reboot, service otomatis running.
- Setelah `disable`, service tidak auto-start di boot berikutnya.

---

## 🐛 Troubleshooting & Kesalahan

- **Service tidak start:** Cek log:
  ```bash
  $ sudo systemctl status adinusa.service
  $ sudo journalctl -u adinusa.service -n 50 --no-pager
  ```
  Pesan error biasanya jelas (path salah, permission, dll).

- **`Failed to start ... Unit not found`:** File `.service` tidak ada di path yang benar, atau typo nama. Cek:
  ```bash
  $ ls /etc/systemd/system/*.service
  ```

- **Perubahan tidak berlaku setelah edit:** Lupa `sudo systemctl daemon-reload`.

- **Service start tapi langsung mati (`inactive (dead)`):** Ini normal untuk service yang `ExecStart`-nya pendek (seperti `echo`). Service dianggap selesai begitu perintah selesai. Untuk service yang "hidup terus", `ExecStart` harus menjalankan program yang tidak exit (misalnya `sleep 60` atau daemon).

- **`enable` gagal:** Cek format `[Install]` — harus ada `WantedBy=`. Tanpa ini, systemd tidak tahu kapan service harus start.

- **`ExecStart` gagal karena path:** Gunakan **path absolut** (`/bin/echo`, bukan `echo`). systemd tidak baca `PATH` user.

- **`daemon-reload` tanpa `sudo`:** Tidak akan berhasil — systemd unit file milik root.

- **Service tidak start setelah reboot:** Cek `systemctl is-enabled adinusa.service`. Kalau `disabled`, jalankan `sudo systemctl enable`.

- **Edit unit file langsung tanpa reload:** Perubahan **tidak berlaku** sampai `daemon-reload`. Jadi bisa kelihatan seperti "edit tidak berfungsi".

- **Conflicting service names:** Kalau kamu buat `apache2.service` di `/etc/systemd/system/`, itu akan **menimpa** service bawaan. Hati-hati.

- **`journalctl -u` tidak menampilkan log:** Log tersimpan di `/var/log/journal/` (persistent) atau `/run/log/journal/` (volatile, hilang saat reboot). Cek:
  ```bash
  $ sudo journalctl --disk-usage
  ```

---

## 💡 Catatan & Insight Pribadi

### Analogi systemd Service

Bayangkan service seperti **karyawan di kantor**:
- **Unit file** = kontrak kerja (tugas, jam kerja)
- **`start`** = masuk kerja sekarang
- **`stop`** = pulang
- **`enable`** = dimasukkan ke jadwal tetap (masuk setiap hari)
- **`disable`** = dikeluarkan dari jadwal
- **`daemon-reload`** = HRD baca ulang kontrak

### Anatomi Unit File

```ini
[Unit]                              ← SECTION 1: metadata
Description=adinusa
After=network.target

[Service]                           ← SECTION 2: cara jalan
ExecStart=/bin/sh -c '...'
ExecStop=/bin/echo ...

[Install]                           ← SECTION 3: kapan start
WantedBy=multi-user.target
```

### Perintah `systemctl` yang Wajib Dihafal

| Perintah | Fungsi |
|---|---|
| `systemctl start <svc>` | Jalankan sekarang |
| `systemctl stop <svc>` | Hentikan |
| `systemctl restart <svc>` | Restart |
| `systemctl reload <svc>` | Baca ulang config (tanpa stop) |
| `systemctl status <svc>` | Cek status |
| `systemctl enable <svc>` | Auto-start saat boot |
| `systemctl disable <svc>` | Jangan auto-start |
| `systemctl is-active <svc>` | Cek running |
| `systemctl is-enabled <svc>` | Cek auto-start |
| `systemctl daemon-reload` | Baca ulang semua unit file |
| `systemctl list-units --type=service` | Daftar service aktif |
| `systemctl list-unit-files --type=service` | Daftar semua unit file |
| `journalctl -u <svc>` | Log service |
| `systemctl mask <svc>` | Block service (tidak bisa di-start) |
| `systemctl unmask <svc>` | Unblock |

### Target vs Runlevel

| Runlevel (lama) | Target (systemd) |
|---|---|
| 0 | poweroff.target |
| 1 | rescue.target |
| 3 | multi-user.target |
| 5 | graphical.target |
| 6 | reboot.target |

`WantedBy=multi-user.target` = service start di runlevel 3 (server mode, tanpa GUI).

### Jenis Service

| Type | Keterangan |
|---|---|
| `simple` | Default — proses utama langsung jalan |
| `forking` | Proses fork ke background (daemon klasik) |
| `oneshot` | Jalan sekali, selesai |
| `notify` | Proses kasih sinyal ke systemd saat siap |
| `idle` | Tunggu semua job selesai |

Untuk service yang `ExecStart`-nya `echo` lalu selesai, default `Type=simple` akan tampak seperti service selesai langsung. Ini normal.

### Best Practice

1. **Selalu `daemon-reload`** setelah edit unit file.
2. **Gunakan path absolut** di `ExecStart`/`ExecStop`.
3. **Set `User=`** kalau service tidak butuh root.
4. **Log ke journal** dengan `StandardOutput=journal`.
5. **Test dengan `systemctl start`** sebelum `enable`.
6. **Cek `journalctl`** kalau service gagal.
7. **Jangan edit service bawaan** — buat override dengan `systemctl edit`.
8. **Backup unit file** sebelum ubah.

### `systemctl edit` — Cara Aman Override

Kalau ingin ubah service bawaan (misal `nginx.service`) tanpa mengedit file aslinya:

```bash
$ sudo systemctl edit nginx.service
```

Ini akan buka editor untuk file **override** di:
```
/etc/systemd/system/nginx.service.d/override.conf
```

Isi override hanya bagian yang ingin diubah. systemd akan merge dengan file asli.

### Kasus Nyata

- **Deploy aplikasi custom:** Bikin service `myapp.service` untuk jalankan binary di `/opt/myapp/`.
- **Backup otomatis:** Service `backup.service` yang jalankan script backup.
- **Monitoring agent:** Service untuk agent monitoring (Prometheus node_exporter, dll).
- **Cron alternatif:** `systemd timer` lebih modern dari `cron` untuk jadwal tugas.

### Pelajaran Kunci dari Lab Ini

1. **systemd** = init + service manager modern.
2. **Unit file** ada di `/etc/systemd/system/`.
3. **3 section**: `[Unit]`, `[Service]`, `[Install]`.
4. **`daemon-reload`** wajib setelah edit.
5. **`enable`** = auto-start saat boot.
6. **`journalctl -u`** untuk lihat log service.
7. **Target** = pengganti runlevel.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `sudo vim /etc/systemd/system/<name>.service` | Buat unit file |
| `sudo systemctl daemon-reload` | Baca ulang unit file |
| `sudo systemctl start <svc>` | Jalankan service |
| `sudo systemctl stop <svc>` | Hentikan service |
| `sudo systemctl restart <svc>` | Restart service |
| `sudo systemctl status <svc>` | Cek status |
| `sudo systemctl enable <svc>` | Auto-start saat boot |
| `sudo systemctl disable <svc>` | Matikan auto-start |
| `systemctl is-active <svc>` | Cek running |
| `systemctl is-enabled <svc>` | Cek auto-start |
| `journalctl -u <svc> -f` | Log real-time |
| `journalctl -u <svc> -n 50` | Log 50 baris terakhir |
| `sudo systemctl mask <svc>` | Block service |
| `sudo systemctl unmask <svc>` | Unblock service |
| `sudo systemctl edit <svc>` | Override service bawaan |

**File penting:**

| File | Isi |
|---|---|
| `/etc/systemd/system/<name>.service` | Custom service |
| `/etc/systemd/system/multi-user.target.wants/` | Symlink service yang enabled |
| `/lib/systemd/system/<name>.service` | Service bawaan package |
| `/etc/systemd/system/<name>.service.d/override.conf` | Override service |

**Template unit file:**

```ini
[Unit]
Description=<deskripsi service>
After=network.target

[Service]
Type=simple
User=<user>
ExecStart=<path-absolut-perintah>
ExecStop=<path-absolut-perintah>
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

**Direktif `[Service]` lanjutan:**

| Direktif | Fungsi |
|---|---|
| `Type` | Jenis service (simple, forking, oneshot, notify) |
| `User` / `Group` | User yang menjalankan service |
| `WorkingDirectory` | Direktori kerja |
| `Environment` | Environment variable |
| `Restart` | Kapan restart (no, on-failure, always) |
| `RestartSec` | Jeda sebelum restart |
| `StandardOutput` | Ke mana output (journal, file, null) |

---

## 📌 Kesimpulan

Lab ini memperkenalkan **systemd service** — fondasi untuk mengelola aplikasi & daemon di Linux modern. Kemampuan yang didapat:

- Membuat **custom service unit**.
- Mengelola service dengan **`systemctl`** (start, stop, status).
- **Enable** service agar otomatis start saat boot.
- Melihat **log** service dengan `journalctl`.
- **`daemon-reload`** setelah edit unit file.

Yang paling penting dipahami: **systemd adalah pengelola service utama** di Linux modern. Semua daemon — dari SSH, Apache, MySQL, sampai aplikasi custom — dikelola oleh systemd. Menguasai cara membuat dan mengelola service adalah **skill wajib sysadmin** untuk deployment aplikasi, otomasi, dan manajemen server.

Kemampuan ini juga membuka jalan ke topik lanjutan seperti **systemd timer** (pengganti cron), **systemd socket activation**, dan **container orchestration**.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
