# Lab 14.1 — Configure the Hostname

**Course:** Linux System Administration (Adinusa)
**Topic:** Hostname — temporary vs permanent
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Melihat hostname saat ini dengan `hostname` dan `hostnamectl`
- Mengubah hostname **sementara** (hilang setelah reboot)
- Mengubah hostname **permanen** (bertahan setelah reboot)
- Memverifikasi perubahan hostname di `/etc/hostname`

---

## 📘 Konsep Dasar

Hostname adalah **nama** yang mengidentifikasi sebuah sistem di jaringan. Linux mengenal tiga tipe hostname:

| Tipe | Sifat | Disimpan di |
|---|---|---|
| **Static** | Permanen, bertahan setelah reboot | `/etc/hostname` |
| **Transient** | Sementara, hilang setelah reboot | Kernel runtime |
| **Pretty** | Nama deskriptif (bisa ada spasi) | `/etc/machine-info` |

**Perintah utama:**

| Perintah | Fungsi |
|---|---|
| `hostname` | Lihat/set hostname (cara lama) |
| `hostnamectl` | Lihat/set hostname (cara modern, systemd) |

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-014-1
```

---

## 📘 Guided Example

### 1. Cek hostname saat ini

```bash
$ hostname
servera
```

### 2. Set hostname sementara

```bash
$ sudo hostname mytempserver
$ hostname
mytempserver
```

Cek dengan `hostnamectl`:

```bash
$ hostnamectl
   Static hostname: servera
 Transient hostname: mytempserver
           Icon name: computer-vm
             Chassis: vm 🖴
          Machine ID: 2b7e4b658ca349bb867889b0cc52780e
             Boot ID: 425827fa71d441cc9273328703dd5f63
      Virtualization: oracle
    Operating System: Ubuntu 24.04.3 LTS
              Kernel: Linux 6.8.0-78-generic
        Architecture: x86-64
```

**Penjelasan:** `Static hostname` tetap `servera`, tapi `Transient hostname` berubah jadi `mytempserver`. Perubahan ini **hanya sementara** — akan hilang setelah reboot.

### 3. Set hostname permanen

```bash
$ sudo hostnamectl set-hostname vm-linlab
$ hostname
vm-linlab
```

Cek lagi dengan `hostnamectl`:

```bash
$ hostnamectl
   Static hostname: vm-linlab
           Icon name: computer-vm
             Chassis: vm 🖴
          Machine ID: 2b7e4b658ca349bb867889b0cc52780e
             Boot ID: 425827fa71d441cc9273328703dd5f63
      Virtualization: oracle
    Operating System: Ubuntu 24.04.3 LTS
              Kernel: Linux 6.8.0-78-generic
        Architecture: x86-64
```

**Penjelasan:** Sekarang `Static hostname` sudah berubah jadi `vm-linlab` — ini bertahan setelah reboot.

### 4. Verifikasi di `/etc/hostname`

```bash
$ cat /etc/hostname
vm-linlab
```

File ini menyimpan hostname permanent yang dibaca saat boot.

---

## 🧪 Practice Task

### Soal

1. Cek hostname saat ini dengan `hostname` dan `hostnamectl`.
2. Set hostname sementara ke `temp-server`.
3. Verifikasi perbedaan static vs transient di `hostnamectl`.
4. Set hostname permanen ke `prod-server-01`.
5. Verifikasi `/etc/hostname`.
6. Cek FQDN dengan `hostname -f`.

### ✅ Solusi

```bash
# 1. Cek hostname
$ hostname
$ hostnamectl

# 2. Set hostname sementara
$ sudo hostname temp-server
$ hostname
temp-server

# 3. Verifikasi static vs transient
$ hostnamectl
# Static hostname: <asli>
# Transient hostname: temp-server

# 4. Set hostname permanen
$ sudo hostnamectl set-hostname prod-server-01

# 5. Verifikasi
$ cat /etc/hostname
prod-server-01

# 6. Cek FQDN
$ hostname -f
prod-server-01
```

### 🔍 Hasil yang Diharapkan

- `hostname` menampilkan `prod-server-01`.
- `cat /etc/hostname` menampilkan `prod-server-01`.
- Setelah reboot, hostname tetap `prod-server-01`.

---

## 🐛 Troubleshooting & Kesalahan

- **`hostname` tidak berubah setelah `hostnamectl set-hostname`:** Prompt shell biasanya belum ter-update. Buka terminal baru, atau logout & login ulang. `hostnamectl` sendiri sudah mengubah hostname di kernel, hanya prompt lama yang masih menampilkan yang lama.

- **Prompt masih menampilkan hostname lama:** Prompt bash mengambil hostname saat login. Setelah ganti hostname, prompt tidak langsung berubah. Logout-login ulang atau jalankan `exec bash`.

- **`hostnamectl` command not found:** Sistem tidak pakai systemd (misal container minimal atau distro lama). Gunakan `hostname` dan edit `/etc/hostname` manual.

- **Hostname tidak berubah setelah reboot:** Cek `/etc/hostname` — mungkin masih berisi nama lama. Edit manual dengan `sudo nano /etc/hostname` lalu reboot.

- **Perbedaan `hostname` vs `hostnamectl`:** `hostname` hanya mengubah **transient** hostname (tidak persist). `hostnamectl set-hostname` mengubah **static** (persist).

- **Error "Could not set property: Access denied":** Lupa `sudo`. Perintah `hostnamectl set-hostname` butuh root.

- **Hostname mengandung karakter tidak valid:** Hostname hanya boleh huruf, angka, tanda hubung (`-`), dan titik (`.`). Tidak boleh ada spasi atau underscore.

- **FQDN tidak muncul:** `hostname -f` butuh konfigurasi `/etc/hosts` yang benar. Tambahkan baris `127.0.1.1 hostname.domain hostname` jika perlu.

---

## 💡 Catatan & Insight Pribadi

### Kenapa Ada Tiga Tipe Hostname?

- **Static** → nama permanen sistem. Dipakai saat boot, disimpan di `/etc/hostname`. Ini yang biasanya dimaksud "hostname".
- **Transient** → nama sementara yang diset di kernel. Kadang di-set oleh DHCP client. Hilang saat reboot.
- **Pretty** → nama deskriptif untuk tampilan (misal `"Web Server Production"`). Bisa ada spasi. Disimpan di `/etc/machine-info`.

### Kapan Pakai `hostname` vs `hostnamectl`?

| Kebutuhan | Pakai |
|---|---|
| Set hostname permanen | `hostnamectl set-hostname` |
| Set hostname sementara (testing) | `hostname` |
| Lihat semua info hostname | `hostnamectl` |
| Script lama / container minimal | `hostname` + edit `/etc/hostname` |

**Rekomendasi modern:** Selalu pakai `hostnamectl` untuk permanent. Lebih aman dan konsisten.

### Hostname dan Jaringan

Hostname sering dipakai untuk:
- **Identifikasi di DNS** — nama yang resolve ke IP.
- **SSH prompt** — `user@hostname` memudahkan tahu server mana yang diakses.
- **Log** — hostname muncul di syslog untuk membedakan server.
- **Konfigurasi service** — beberapa aplikasi butuh FQDN yang benar.

### Best Practice Hostname

- Gunakan **lowercase** (huruf kecil).
- Hindari **spasi** dan **underscore**.
- Untuk server: pakai pola deskriptif, misal `web-01`, `db-prod-02`, `cache-sg-01`.
- Hindari nama generic seperti `localhost`, `server`, `test`.
- Update `/etc/hosts` jika butuh FQDN.

### Kasus Nyata

- **Server produksi:** Hostname `web-01.prod.example.com` untuk identifikasi cepat di log dan monitoring.
- **Cluster:** Pola `node-01`, `node-02`, dst.
- **VM:** Hostname sering di-set otomatis oleh cloud-init.
- **Container:** Hostname biasanya di-set oleh runtime (Docker, K8s).

### Pelajaran Kunci dari Lab Ini

1. **`hostname`** = transient (hilang setelah reboot).
2. **`hostnamectl set-hostname`** = static (persist).
3. **`/etc/hostname`** menyimpan hostname permanent.
4. **`hostnamectl`** menampilkan static + transient sekaligus.
5. **Prompt shell** tidak langsung berubah setelah ganti hostname.
6. **Hostname valid** = huruf, angka, `-`, `.` saja.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `hostname` | Lihat/set hostname (transient) |
| `hostnamectl` | Lihat info hostname lengkap |
| `hostnamectl set-hostname <nama>` | Set hostname permanent |
| `hostname -f` | Lihat FQDN |
| `cat /etc/hostname` | Lihat hostname permanent |
| `cat /etc/hosts` | Lihat mapping hostname ↔ IP |

---

## 📌 Kesimpulan

Lab ini memperkenalkan konsep **hostname** dan perbedaan antara **temporary** (transient) dan **permanent** (static). Meskipun terlihat sederhana, hostname adalah identitas sistem di jaringan — muncul di prompt, log, DNS, dan konfigurasi service. Menguasai cara mengubah hostname dengan benar adalah keterampilan dasar sysadmin, terutama saat menyiapkan server baru atau VM di cloud.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
