# Lab 16.0 — Firewall, Firewalld & HTTPS (Teori)

**Course:** Linux System Administration (Adinusa)
**Topic:** Teori firewall, firewalld zone, dan HTTPS
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab teori ini, saya mampu:

- Memahami konsep dan fungsi **firewall** dalam keamanan jaringan
- Menggunakan **`firewalld`** dan perintah **`firewall-cmd`**
- Memahami konsep **zone** dan cara mengelola interface
- Menjelaskan cara kerja **HTTPS** dan **TLS/SSL**
- Mengonfigurasi firewalld untuk proteksi server

---

## 📘 Konsep Dasar — Apa Itu Firewall?

**Firewall** = sistem keamanan jaringan yang memonitor dan mengontrol lalu lintas masuk/keluar berdasarkan aturan.

**Cara kerja:** Setiap paket diperiksa header-nya (source IP, dest IP, port, protokol). Berdasarkan aturan, paket bisa:
- **Accept** → diteruskan
- **Reject/Drop** → diblokir
- **Redirect** → diarahkan ke sistem lain
- **Modify (mangle)** → diubah propertinya

**Jenis firewall:**

| Jenis | Contoh |
|---|---|
| **Hardware** | Router, appliance khusus |
| **Software** | `iptables`, `nftables`, `firewalld`, `ufw` |
| **Hybrid** | Kombinasi keduanya |

### Generasi Firewall

| Generasi | Tipe | Kemampuan |
|---|---|---|
| 1 | Packet Filtering | Lihat header (IP, port, protokol) |
| 2 | Stateful Inspection | Track koneksi (new, established, invalid) |
| 3 | Application Layer (NGFW) | Deep packet inspection, paham HTTP/DNS/FTP |

### Tools Firewall di Linux

| Tool | Keterangan |
|---|---|
| `iptables` | Classic, sedang digantikan |
| `nftables` | Modern replacement |
| **`firewalld`** | Dynamic, zone-based — **paling umum sekarang** |
| `ufw` | Simple, default di Ubuntu |
| `firewall-config` | GUI untuk firewalld |

> **Fokus:** `firewalld` dengan `firewall-cmd` — paling stabil lintas distro (RHEL, CentOS, Fedora, SUSE).

---

## 📘 Konsep Dasar — Firewalld & firewall-cmd

**`firewalld`** = dynamic firewall manager berbasis **zone**. Mendukung IPv4 & IPv6, dan memisahkan **runtime** vs **permanent** config.

**File konfigurasi:**

| Path | Isi |
|---|---|
| `/etc/firewalld/` | Custom config (admin) |
| `/usr/lib/firewalld/` | Default config (system) |

> **Penting:** Selalu edit di `/etc/firewalld/`, jangan di `/usr/lib/` — supaya tidak tertimpa update sistem.

### Runtime vs Permanent

| Tipe | Sifat |
|---|---|
| **Runtime** | Instant, hilang setelah reboot |
| **Permanent** | Tersimpan di config, butuh `--reload` |

**Alur kerja:**
```bash
# Runtime saja
sudo firewall-cmd --zone=public --add-service=http

# Permanent
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --reload

# Convert runtime ke permanent
sudo firewall-cmd --runtime-to-permanent
```

### Perintah Dasar `firewall-cmd`

| Perintah | Fungsi |
|---|---|
| `firewall-cmd --state` | Cek status |
| `firewall-cmd --get-active-zones` | Zone aktif |
| `firewall-cmd --get-default-zone` | Zone default |
| `firewall-cmd --get-zones` | Daftar semua zone |
| `firewall-cmd --list-services --zone=public` | Service yang diizinkan |
| `firewall-cmd --zone=public --add-service=http` | Tambah service |
| `firewall-cmd --zone=public --add-port=8080/tcp` | Buka port |
| `firewall-cmd --zone=public --remove-service=http` | Hapus service |
| `firewall-cmd --reload` | Reload permanent config |

> **Jangan jalankan `iptables` dan `firewalld` bersamaan** — konflik.

---

## 📘 Konsep Dasar — Service Status & IP Forwarding

### Service systemd

```bash
# Start/stop
sudo systemctl start firewalld
sudo systemctl stop firewalld

# Enable/disable saat boot
sudo systemctl enable firewalld
sudo systemctl disable firewalld

# Cek status
sudo systemctl status firewalld
# atau
sudo firewall-cmd --state
```

### IP Forwarding (untuk router/gateway)

**Temporary:**
```bash
sudo sysctl net.ipv4.ip_forward=1
```

**Permanent** — edit `/etc/sysctl.conf`, tambah:
```
net.ipv4.ip_forward=1
```

Apply:
```bash
sudo sysctl -p
```

---

## 📘 Konsep Dasar — Zone Firewalld

**Zone** = set aturan yang mendefinisikan **tingkat kepercayaan** untuk interface. Setiap interface di-assign ke satu zone.

**Zone default** di banyak distro: `public`.

### Daftar Zone (dari paling ketat ke paling longgar)

| Zone | Kepercayaan | Keterangan |
|---|---|---|
| `drop` | Sangat rendah | Drop semua incoming, no reply |
| `block` | Sangat rendah | Reject incoming dengan error |
| `public` | Rendah | **Default**. Hanya service eksplisit yang diizinkan |
| `external` | Rendah | Untuk NAT/masquerading |
| `dmz` | Sedang | Untuk server publik (web, mail) |
| `work` | Sedang | Jaringan kantor |
| `home` | Sedang-tinggi | Jaringan rumah |
| `internal` | Sedang | Mirip work |
| `trusted` | Sangat tinggi | Semua diizinkan |

> **Best practice:** Pakai zone **paling ketat** yang masih fungsional.

### Manajemen Zone

| Perintah | Fungsi |
|---|---|
| `sudo firewall-cmd --get-default-zone` | Lihat default zone |
| `sudo firewall-cmd --set-default-zone=trusted` | Ubah default |
| `sudo firewall-cmd --get-active-zones` | Zone aktif + interface |
| `sudo firewall-cmd --get-zones` | Semua zone |
| `sudo firewall-cmd --zone=internal --change-interface=enp0s8` | Assign interface (runtime) |
| `sudo firewall-cmd --permanent --zone=internal --change-interface=enp0s8` | Assign permanent |
| `sudo firewall-cmd --get-zone-of-interface=eno1` | Cek zone interface |
| `sudo firewall-cmd --zone=public --list-all` | Detail zone |

**Output `--list-all`:**
```
public (active)
  target: default
  interfaces: eno1
  services: ssh http https
  ports:
  masquerade: no
  forward-ports:
  rich rules:
```

---

## 📘 Konsep Dasar — Service & Port Management

### Service

| Perintah | Fungsi |
|---|---|
| `sudo firewall-cmd --get-services` | Daftar service predefined |
| `sudo firewall-cmd --list-services --zone=public` | Service di zone |
| `sudo firewall-cmd --permanent --zone=home --add-service=dhcp` | Tambah service |
| `sudo firewall-cmd --reload` | Apply |

Custom service bisa dibuat di `/etc/firewalld/services/`.

### Port

| Perintah | Fungsi |
|---|---|
| `sudo firewall-cmd --zone=home --add-port=21/tcp` | Buka port (runtime) |
| `sudo firewall-cmd --zone=home --list-ports` | Lihat port |
| `grep " 21/tcp" /etc/services` | Cari service untuk port |

**Referensi lengkap:** `man firewall-cmd`.

---

## 📘 Konsep Dasar — HTTPS

**HTTPS** = HTTP + TLS/SSL. Versi aman dari HTTP.

| Aspek | Keterangan |
|---|---|
| **Port default** | TCP **443** |
| **Enkripsi** | TLS (modern) atau SSL (legacy) |

### Tiga Pilar HTTPS

| Pilar | Fungsi |
|---|---|
| **Confidentiality** | Data terenkripsi — tidak bisa disadap |
| **Integrity** | Data tidak bisa diubah tanpa terdeteksi |
| **Authentication** | Server diverifikasi via certificate dari CA |

### SSL vs TLS

| Protokol | Status |
|---|---|
| **SSL 2.0/3.0** | ❌ Obsolete, tidak aman |
| **TLS 1.0/1.1** | ⚠️ Deprecated |
| **TLS 1.2** | ✅ Aman, umum |
| **TLS 1.3** | ✅ Paling aman |

> Istilah *"SSL certificate"* masih umum dipakai, tapi teknologinya sekarang **TLS**.

### HTTPS di Firewalld

Firewalld sudah punya service predefined `https` — otomatis buka TCP 443.

```bash
# Cek tersedia
firewall-cmd --get-services | grep https

# Runtime
sudo firewall-cmd --zone=public --add-service=https

# Permanent
sudo firewall-cmd --zone=public --add-service=https --permanent
sudo firewall-cmd --reload

# Verifikasi
sudo firewall-cmd --zone=public --list-all
```

Output contoh:
```
services: dhcpv6-client https ssh
```

---

## 🧪 Latihan Pemahaman

### Soal

1. Apa perbedaan **runtime** dan **permanent** di firewalld?
2. Zone mana yang paling **ketat** dan paling **longgar**?
3. Port default HTTPS?
4. Apa perbedaan **SSL** dan **TLS**?
5. Kenapa **tidak boleh** jalankan `iptables` dan `firewalld` bersamaan?

### ✅ Jawaban

1. **Runtime** = instant, hilang setelah reboot. **Permanent** = tersimpan di config, butuh `--reload` untuk apply.

2. Paling **ketat**: `drop` (drop semua tanpa reply). Paling **longgar**: `trusted` (semua diizinkan).

3. **TCP 443**.

4. **SSL** = protokol lama (obsolete, tidak aman). **TLS** = penerus SSL, versi modern (1.2 & 1.3) yang aman.

5. Karena keduanya mengelola netfilter kernel secara berbeda — bisa konflik dan bikin aturan tidak konsisten. `firewalld` adalah layer di atas `nftables`/`iptables`.

---

## 🐛 Troubleshooting & Kesalahan Umum

- **`firewall-cmd: command not found`:** Firewalld belum terinstall. `sudo apt install firewalld -y`.

- **Firewalld dan UFW bentrok:** Di Ubuntu, `ufw` default aktif. Matikan dulu:
  ```bash
  sudo systemctl disable --now ufw
  sudo systemctl enable --now firewalld
  ```

- **Perubahan hilang setelah reboot:** Lupa `--permanent`. Runtime config memang hilang setelah reboot.

- **`--permanent` tidak langsung berlaku:** Harus `firewall-cmd --reload` setelah perubahan permanent.

- **Zone interface tidak berubah:** Cek `firewall-cmd --get-active-zones`. Kalau interface masih di zone lama, mungkin ada konfigurasi NetworkManager yang override.

- **HTTPS sudah di-allow tapi masih tidak bisa diakses:** Cek service web (Apache/Nginx) benar-benar listen di port 443. Firewall hanya izinkan; service harus jalan juga.

- **`iptables` rules hilang setelah install firewalld:** Memang by design — firewalld menggantikan iptables.

- **SSH terputus setelah enable firewalld:** Zone `public` default sudah izinkan `ssh`, tapi kalau tidak, dari console VM:
  ```bash
  sudo firewall-cmd --add-service=ssh --permanent
  sudo firewall-cmd --reload
  ```

---

## 💡 Catatan & Insight Pribadi

### Analogi Firewall

Firewall = **satpam gedung**:
- **Packet filtering** = cek KTP di pintu masuk.
- **Stateful** = ingat siapa yang sudah masuk, izinkan keluar.
- **Application layer** = periksa isi tas juga.

### Zone sebagai "Level Kepercayaan"

Analoginya seperti **tamu di rumah**:
- **`trusted`** = keluarga — bebas masuk.
- **`home`** = teman dekat — boleh masuk ruang tamu.
- **`public`** = tamu — hanya ruang tamu, tidak boleh ke kamar.
- **`drop`** = tidak dikenal — tidak dibukakan pintu.

### Kapan Pakai `firewalld` vs `ufw`?

| Skenario | Pakai |
|---|---|
| Ubuntu home user | `ufw` |
| RHEL/CentOS/Fedora server | `firewalld` |
| Multi-interface server | `firewalld` (zone-based lebih fleksibel) |
| Lab Adinusa | `firewalld` |

### Runtime vs Permanent — Analogi

- **Runtime** = tulis di papan tulis — cepat, tapi bisa terhapus.
- **Permanent** = tulis di buku — perlu waktu, tapi tersimpan.

### Kasus Nyata

- **Server publik:** Interface publik di zone `public`, hanya buka port 80/443/22.
- **Server internal:** Zone `internal`, izinkan lebih banyak service.
- **DMZ:** Server web di zone `dmz`, diisolasi dari LAN.
- **Router/gateway:** Pakai `external` + IP forwarding + masquerading.

### Pelajaran Kunci dari Teori Ini

1. **Firewall** = kontrol lalu lintas berdasarkan aturan.
2. **`firewalld`** = zone-based, modern, lintas distro.
3. **Zone** = tingkat kepercayaan (drop → trusted).
4. **Runtime vs permanent** = instant vs persistent.
5. **`--permanent` + `--reload`** untuk commit.
6. **HTTPS** = HTTP + TLS, port **443**.
7. **SSL obsolete**, pakai **TLS 1.2/1.3**.

---

## 🧠 Konsep yang Dikuasai

| Konsep | Fungsi |
|---|---|
| `firewall-cmd` | CLI manajemen firewalld |
| Zone | Tingkat kepercayaan interface |
| Runtime vs Permanent | Config temporary vs persistent |
| `--add-service` / `--add-port` | Buka akses |
| `--reload` | Apply permanent config |
| IP forwarding | Router/gateway |
| HTTPS | HTTP + TLS, port 443 |

---

## 📌 Kesimpulan

Teori ini memberikan fondasi untuk semua lab firewall (16.x). Yang paling penting dipahami:

- **Firewalld** adalah firewall dinamis berbasis **zone** — lebih fleksibel dari `ufw` untuk server multi-interface.
- **Zone** memudahkan manajemen kepercayaan per interface.
- **Runtime vs permanent** adalah konsep kunci di firewalld.
- **HTTPS** wajib untuk server publik — modern pakai **TLS 1.2/1.3**.

Kemampuan ini akan terus dipakai di dunia kerja — hampir semua server produksi punya firewall. Kesalahan konfigurasi firewall bisa bikin server tidak bisa diakses atau justru rentan.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
