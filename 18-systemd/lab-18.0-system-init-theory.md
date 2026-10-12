# Lab 18.0 — System Init: systemd, SysVinit & Upstart (Teori)

**Course:** Linux System Administration (Adinusa)
**Topic:** Teori init system — systemd, SysVinit, Upstart
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab teori ini, saya mampu:

- Memahami peran dan fungsi proses **init** di Linux
- Membedakan **SysVinit**, **Upstart**, dan **systemd**
- Menjelaskan fitur utama **systemd**
- Memahami struktur **file konfigurasi systemd**
- Menggunakan **`systemctl`** untuk mengelola service
- Membuat **startup service** dengan systemd

---

## 📘 Konsep Dasar — Proses `init`

Setelah kernel selesai loading, proses user-space pertama yang dijalankan adalah **`/sbin/init`** (biasa disebut **init**). Proses ini menjadi **orkestrator** startup sistem.

**Peran init:**
- **Boot coordination** — menyelesaikan tahap akhir startup
- **Environment setup** — konfigurasi device, filesystem, daemon
- **Service management** — jalankan service background, siapkan login
- **Process cleanup** — bersihkan proses saat terminate

> **Catatan:** init sering disebut "parent of all processes", tapi tidak sepenuhnya benar — beberapa proses dispawn langsung oleh kernel. init menjadi **adoptive parent** untuk proses orphan.

---

## 📘 Konsep Dasar — Evolusi Init System

### 1. SysVinit (Legacy)

Sistem init tradisional dari UNIX klasik.

**Karakteristik:**
- Organisasi berdasarkan **runlevel** (0–6)
- Setiap runlevel = state sistem tertentu (single-user, multi-user, shutdown, reboot)
- **Serial** — satu script selesai baru jalan berikutnya

| Runlevel | Fungsi |
|---|---|
| 0 | Shutdown |
| 1 | Single-user (rescue) |
| 2 | Multi-user tanpa network |
| 3 | Multi-user dengan network |
| 4 | (Custom) |
| 5 | Multi-user + GUI |
| 6 | Reboot |

**Kelemahan:**
- ❌ Tidak bisa manfaatkan multi-core CPU (tidak paralel)
- ❌ Boot/shutdown lambat
- ❌ Kurang fleksibel untuk event dinamis (hot-plug)

### 2. Upstart (Obsolete)

Dikembangkan Ubuntu (2006), event-driven.

**Karakteristik:**
- **Event-driven** — service start/stop sebagai respons event
- Cocok untuk device hot-plug, dynamic service
- Diadopsi Fedora 9 (2008), RHEL 6

**Kelemahan:**
- ❌ Kurang advanced dependency handling
- ❌ Sulit scale untuk sistem kompleks
- ❌ Digantikan systemd di hampir semua distro

### 3. systemd (Modern Standard)

Dikembangkan Fedora (2011), sekarang jadi **standar industri**.

**Adopsi:**
- Fedora → RHEL → SUSE → Ubuntu 16.04 LTS
- Hampir **semua** distro Linux modern: Red Hat, Fedora, Ubuntu, Debian, SUSE, Arch

### Tabel Perbandingan

| Aspek | SysVinit | Upstart | **systemd** |
|---|---|---|---|
| Model | Serial | Event-driven | **Paralel + dependency** |
| Startup | Lambat | Sedang | **Cepat** |
| Config | Shell script | `.conf` | **Unit file** |
| Log | File terpisah | File terpisah | **journald** |
| Status | Legacy | Obsolete | **Standard** |

> **"Init Wars"** — perdebatan sengit antar komunitas. Hasil akhir: **systemd menang** dan menjadi fondasi Linux modern.

---

## 📘 Konsep Dasar — Fitur systemd

**10 Fitur Utama systemd:**

| # | Fitur | Keterangan |
|---|---|---|
| 1 | **Faster boot** | Paralelisasi service startup |
| 2 | **Dependency management** | Service start sesuai dependency, paralel jika independen |
| 3 | **Socket & D-Bus activation** | Service start on-demand saat dibutuhkan |
| 4 | **Binary programs** | Pakai binary + config, bukan shell script |
| 5 | **On-demand daemon** | Daemon tidak perlu jalan terus, trigger saat perlu |
| 6 | **cgroups tracking** | Setiap proses di cgroup → tracking CPU/mem/IO |
| 7 | **Mount & automount** | Handle mount point otomatis |
| 8 | **Transactional control** | Startup/shutdown divalidasi dulu |
| 9 | **SysVinit compatibility** | Support script SysVinit + telinit |
| 10 | **Service files** | `.service` unit file — deklaratif, sederhana |

### Analogi Fitur

- **Paralelisasi** = buka banyak pintu sekaligus (bukan satu-satu)
- **Socket activation** = panggil tukang saat butuh (bukan standby terus)
- **cgroups** = setiap karyawan punya ID card terpisah
- **Transactional** = cek semua persiapan sebelum mulai

---

## 📘 Konsep Dasar — File Konfigurasi systemd

systemd **menstandarisasi** file konfigurasi lintas distro. Meskipun tetap mendukung file lama untuk kompatibilitas.

### File Konfigurasi Standar

| File | Fungsi |
|---|---|
| `/etc/hostname` | Hostname sistem |
| `/etc/vconsole.conf` | Keyboard layout & font console |
| `/etc/sysctl.d/*.conf` | Drop-in kernel parameter |
| `/etc/os-release` | Info identitas OS |

**Contoh `/etc/vconsole.conf`:**

```
KEYMAP=us
FONT=latarcyrheb-sun16
```

**Contoh `/etc/sysctl.d/99-custom.conf`:**

```
net.ipv4.ip_forward=1
net.ipv6.conf.all.disable_ipv6=1
```

**Contoh `/etc/os-release`:**

```
NAME="Ubuntu"
VERSION="20.04.6 LTS (Focal Fossa)"
ID=ubuntu
PRETTY_NAME="Ubuntu 20.04.6 LTS"
```

### Kompatibilitas Legacy

systemd tetap mendukung konsep lama:

| Legacy | Modern |
|---|---|
| Runlevel 3 | `multi-user.target` |
| `telinit` | Emulated |
| Script SysVinit | Bisa dijalankan |

---

## 📘 Konsep Dasar — `systemctl`

**`systemctl`** = command-line utama untuk kontrol systemd.

**Syntax:**
```bash
systemctl [options] command [unit]
```

### Perintah Umum

**Melihat service:**

| Perintah | Fungsi |
|---|---|
| `systemctl` | Tampilkan semua yang dikontrol systemd |
| `systemctl list-units -t service --all` | Semua service (aktif & inaktif) |
| `systemctl list-units -t service` | Service aktif saja |

**Start / stop / restart:**

| Perintah | Fungsi |
|---|---|
| `systemctl start sshd` | Start sekarang |
| `systemctl stop sshd` | Stop |
| `systemctl restart sshd` | Restart |
| `systemctl reload sshd` | Reload config tanpa stop |

**Enable / disable:**

| Perintah | Fungsi |
|---|---|
| `systemctl enable sshd` | Auto-start saat boot |
| `systemctl disable sshd` | Jangan auto-start |
| `systemctl is-enabled sshd` | Cek status enable |

**Status:**

| Perintah | Fungsi |
|---|---|
| `systemctl status sshd` | Cek status + log terakhir |

Output `status` menunjukkan:
- Service **active (running)** atau **inactive (dead)**
- Log & pesan start/stop terakhir

### Apa Itu **Unit**?

Unit = objek yang dikelola systemd.

| Tipe Unit | Fungsi |
|---|---|
| `.service` | Service/daemon |
| `.socket` | Network socket |
| `.device` | Hardware device |
| `.mount` | Filesystem mount |
| `.timer` | Scheduled task (pengganti cron) |

> **Tips:** Suffix `.service` bisa dihilangkan. `systemctl start sshd` = `systemctl start sshd.service`.

---

## 🧪 Latihan Pemahaman

### Soal

1. Apa itu proses **init** dan apa perannya?
2. Apa perbedaan **SysVinit** dan **systemd**?
3. Sebutkan **3 fitur utama** systemd.
4. Apa itu **unit** di systemd?
5. Bagaimana cara enable service agar auto-start saat boot?

### ✅ Jawaban

1. **init** = proses user-space pertama yang dijalankan kernel. Perannya: koordinasi boot, setup environment, manajemen service, cleanup proses.

2. **SysVinit** = serial, pakai runlevel, lambat. **systemd** = paralel, pakai unit/target, cepat, terintegrasi dengan logging (journald).

3. **Paralelisasi startup**, **dependency management**, **socket activation** (atau: cgroups, mount management, service files).

4. **Unit** = objek yang dikelola systemd. Bisa `.service`, `.socket`, `.mount`, `.timer`, `.device`, dll.

5. `sudo systemctl enable <service>`.

---

## 🐛 Troubleshooting & Kesalahan Umum

- **`systemctl: command not found`:** Sistem tidak pakai systemd (misal container minimal, WSL 1, distro lama). Cek dengan `ps -p 1 -o comm=`.

- **Service tidak start:** Cek `systemctl status <svc>` dan `journalctl -u <svc>`.

- **Perubahan config tidak berlaku:** Lupa `sudo systemctl daemon-reload` setelah edit unit file.

- **`enable` gagal:** Cek section `[Install]` di unit file — harus ada `WantedBy=`.

- **Service lama (SysVinit) tidak jalan:** systemd tetap dukung, tapi pastikan script executable dan ada shebang.

- **Runlevel vs Target membingungkan:** `runlevel` command masih ada untuk kompatibilitas, tapi mapping ke `multi-user.target` dll.

- **Log service tidak muncul di `/var/log/syslog`:** systemd pakai **journald** — cek dengan `journalctl`. Kalau mau fallback ke syslog, set `ForwardToSyslog=yes` di `/etc/systemd/journald.conf`.

- **`systemctl status` menampilkan "loaded" tapi "inactive":** Service terdaftar tapi tidak sedang jalan. Jalankan `systemctl start`.

---

## 💡 Catatan & Insight Pribadi

### Analogi Init System

Bayangkan **manajer gedung** (init):
- **SysVinit** = manajer lama, buka pintu satu-satu
- **Upstart** = manajer yang reaktif — buka pintu saat ada tamu
- **systemd** = manajer modern — buka semua pintu paralel, cek dependency, log semua event

### Kenapa systemd Kontroversial?

- **Pro:** Cepat, terintegrasi, standar.
- **Kontra:** Monolitik (banyak fungsi dalam satu binary), kompleks, "bukan UNIX philosophy".

Tapi kenyataannya: **systemd menang** dan sekarang standar industri.

### Target vs Runlevel — Mapping

| Runlevel | Target systemd |
|---|---|
| 0 | `poweroff.target` |
| 1 | `rescue.target` |
| 3 | `multi-user.target` |
| 5 | `graphical.target` |
| 6 | `reboot.target` |

Cek target aktif:
```bash
$ systemctl get-default
graphical.target
```

Ubah target default:
```bash
$ sudo systemctl set-default multi-user.target
```

### Kapan Pakai `reload` vs `restart`?

| Perintah | Efek | Kapan |
|---|---|---|
| `reload` | Baca ulang config tanpa stop | Perubahan kecil (config) |
| `restart` | Matikan lalu nyalakan | Perubahan besar / update |

`reload` lebih halus — user tidak terputus.

### `journalctl` — Log Modern

**journalctl** = tool lihat log systemd.

```bash
# Log semua service
$ journalctl

# Log service tertentu
$ journalctl -u sshd

# Log real-time
$ journalctl -f

# Log sejak boot terakhir
$ journalctl -b

# Log 50 baris terakhir
$ journalctl -n 50
```

> **Kelebihan:** Terintegrasi dengan service, filter by unit, timestamp konsisten.

### Kasus Nyata

- **Server boot lambat:** Cek `systemd-analyze blame` — service mana yang paling lama.
- **Service crash:** `journalctl -u <svc>` untuk lihat error.
- **Migrasi dari SysVinit:** Buat unit file `.service`, enable, disable script lama.
- **Deploy aplikasi:** Bikin `.service` unit untuk otomatis restart jika crash.

### Pelajaran Kunci dari Teori Ini

1. **init** = proses pertama user-space, orkestrator startup.
2. **SysVinit** = legacy, serial, runlevel.
3. **Upstart** = obsolete, event-driven.
4. **systemd** = standard modern, paralel, unit-based.
5. **Unit** = objek systemd (`.service`, `.socket`, `.mount`, `.timer`).
6. **`systemctl`** = CLI utama kontrol systemd.
7. **`journalctl`** = log systemd.
8. **Target** = pengganti runlevel.

---

## 🧠 Konsep yang Dikuasai

| Konsep | Fungsi |
|---|---|
| `init` | Proses pertama user-space |
| SysVinit | Init legacy (runlevel) |
| Upstart | Init event-driven (obsolete) |
| systemd | Init modern (unit-based) |
| `systemctl` | CLI kontrol systemd |
| Unit | Objek systemd |
| Target | Pengganti runlevel |
| `journalctl` | Log systemd |
| `daemon-reload` | Baca ulang unit file |
| `/etc/os-release` | Info OS |
| `/etc/sysctl.d/` | Kernel parameter |

---

## 📌 Kesimpulan

Teori ini memberikan fondasi untuk memahami **bagaimana Linux menginisialisasi sistem** dari kernel sampai siap digunakan. Yang paling penting dipahami:

- **init** adalah proses pertama user-space — orkestrator seluruh startup.
- **SysVinit** (legacy) → **Upstart** (obsolete) → **systemd** (standar modern).
- **systemd** menawarkan paralelisasi, dependency management, dan integrasi logging.
- **`systemctl`** adalah CLI utama — start, stop, enable, status.
- **Unit** menggantikan konsep script SysVinit.
- **Target** menggantikan runlevel.

Kemampuan ini sangat relevan di dunia kerja — hampir semua server Linux modern pakai systemd. Menguasai systemd adalah **wajib** untuk sysadmin yang mengelola service, deploy aplikasi, atau troubleshooting boot.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
