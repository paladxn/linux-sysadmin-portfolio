# Lab 15.0 — Network Device, IP Command & Diagnostics (Teori)

**Course:** Linux System Administration (Adinusa)
**Topic:** Teori dasar networking — device name, `ip`, routing, DNS, diagnostics
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab teori ini, saya mampu:

- Memahami **penamaan network device** modern (PNIDN)
- Menggunakan perintah **`ip`** dan **`ifconfig`** untuk konfigurasi interface
- Menjelaskan cara kerja **routing** dan **default route**
- Memahami konsep **name resolution** (`/etc/hosts` dan DNS)
- Menggunakan tools **diagnostics**: `ping`, `traceroute`, `mtr`, `dig`

---

## 📘 Konsep Dasar — Network Device Naming

Network device **tidak punya file** di `/dev` seperti block/char device. Mereka diidentifikasi dengan **nama** yang di-assign oleh sistem.

| Tipe Device | Contoh Nama |
|---|---|
| Ethernet | `eth0`, `eno1`, `ens3`, `enp0s3` |
| Wireless | `wlan0`, `wlp3s0` |
| Bridge | `br0`, `br1` |
| Virtual | `vmnct0`, `vmnct1` |

### Sejarah: `eth0` → PNIDN

**Dulu** interface dinamai berurutan: `eth0`, `eth1`, dst. **Masalahnya**: urutan deteksi **tidak deterministik** — kadang `eth0` jadi LAN, kadang jadi internet. Ini bikin konfigurasi rusak setelah reboot.

**Solusi**: **Predictable Network Interface Device Names (PNIDN)** — nama berbasis **atribut hardware**, bukan urutan deteksi.

### Lima Skema Penamaan Modern

| Skema | Contoh | Berdasarkan |
|---|---|---|
| Onboard index | `eno1` | Nomor onboard |
| PCI Express hotplug slot | `ens1` | Slot PCIe |
| Physical bus/slot/function | `enp0s3`, `enp2s0` | Lokasi fisik PCI |
| MAC address | `enx7837d1ea46da` | MAC address |
| Legacy | `eth0`, `wlan0` | Urutan (kalau PNIDN dimatikan) |

**Contoh:** `enp0s3` = **e**thernet, **n**etwork, **p**CI bus **0**, **s**lot **3**.

---

## 📘 Konsep Dasar — Perintah `ip`

`ip` adalah tool **modern** untuk manajemen network di Linux. Menggantikan `ifconfig` yang sudah deprecated. Menggunakan **netlink sockets** — lebih cepat dan lengkap.

**Syntax:**
```
ip [OPTIONS] OBJECT {COMMAND | help}
```

| Object | Fungsi |
|---|---|
| `address` | Kelola alamat IPv4/IPv6 |
| `link` | Kelola interface (up/down, MTU) |
| `route` | Kelola routing table |
| `rule` | Routing policy |
| `monitor` | Pantau perubahan real-time |

**Contoh:**

```bash
# Lihat semua IP
$ ip address show       # atau: ip a

# Assign IP ke interface
$ sudo ip address add 192.168.1.100/24 dev ens33

# Nyalakan / matikan interface
$ sudo ip link set ens33 up
$ sudo ip link set ens33 down

# Lihat routing table
$ ip route show

# Tambah default gateway
$ sudo ip route add default via 192.168.1.1

# Pantau perubahan network real-time
$ ip monitor
```

---

## 📘 Konsep Dasar — Perintah `ifconfig`

`ifconfig` adalah tool **lama** yang masih dipakai di banyak tutorial. Di Ubuntu modern **tidak terinstall default** — perlu `sudo apt install net-tools`.

| Tugas | Perintah `ifconfig` | Perintah `ip` (modern) |
|---|---|---|
| Lihat semua interface | `ifconfig` | `ip a` |
| Lihat satu interface | `ifconfig ens3` | `ip addr show dev ens3` |
| Assign IP | `ifconfig ens3 10.5.5.10` | `ip addr add 10.5.5.10/24 dev ens3` |
| Nyalakan interface | `ifconfig ens3 up` | `ip link set ens3 up` |
| Matikan interface | `ifconfig ens3 down` | `ip link set ens3 down` |
| Set MTU | `ifconfig ens3 mtu 1480` | `ip link set ens3 mtu 1480` |

**Kenapa `ifconfig` deprecated?**
- Dukungan IPv6 terbatas.
- Tidak ada fitur modern (policy routing, tunnel).
- Tidak dikembangkan lagi.

> **Rekomendasi:** Pakai `ip` untuk semua hal baru. `ifconfig` hanya kalau terpaksa.

---

## 📘 Konsep Dasar — Network Configuration Tools

Konfigurasi network di Linux sudah **berevolusi**:

| Era | Tool |
|---|---|
| Lama (per distro) | File config di `/etc/` |
| Modern (systemd) | **NetworkManager** |
| Ubuntu modern | **Netplan** (wrapper) |

### File Config Tradisional (berbeda per distro)

| Distro | File |
|---|---|
| Red Hat / CentOS / Fedora | `/etc/sysconfig/network-scripts/ifcfg-ethX` |
| Debian / Ubuntu (lama) | `/etc/network/interfaces` |
| SUSE | `/etc/sysconfig/network` |

### NetworkManager — Tiga Cara Pakai

**1. GUI (desktop applet)** — klik-klik, cocok untuk laptop.

**2. `nmtui`** — text UI, navigasi pakai arrow/Tab:
```bash
$ nmtui
```

**3. `nmcli`** — CLI, cocok untuk automation:
```bash
$ nmcli device status
$ nmcli connection show
$ nmcli connection add type ethernet ifname ens33 con-name mynet autoconnect yes ip4 192.168.1.100/24 gw4 192.168.1.1
```

> **Tips:** `man nmcli-examples` untuk referensi lengkap.

---

## 📘 Konsep Dasar — Routing

Routing = proses memilih **jalur** untuk mengirim traffic. Setiap sistem punya **routing table**.

**Lihat routing table:**

```bash
$ ip route
default via 10.5.5.1 dev ens3 proto static
10.5.5.0/24 dev ens3 proto kernel scope link src 10.5.5.10
```

**Baca:**
- `default via 10.5.5.1` → **default route** — traffic yang tidak match ke mana pun diarahkan ke gateway `10.5.5.1`.
- `10.5.5.0/24 dev ens3` → subnet **langsung terhubung**, di-handle oleh `ens3`.

### Default Route

- Dipakai kalau tidak ada route spesifik yang cocok.
- Biasanya didapat dari **DHCP**, bisa juga di-set manual.

### Static Route

**Temporary (hilang setelah reboot):**
```bash
$ sudo ip route add 10.5.0.0/16 via 192.168.1.100
```

**Persistent** — edit file:
- **Red Hat:** `/etc/sysconfig/network-scripts/route-ethX`
- **Debian/Ubuntu:** `/etc/network/interfaces` dengan `post-up`

> [!] **Hati-hati!** Salah set default gateway bisa **memutus koneksi** sistem dari network.

---

## 📘 Konsep Dasar — Name Resolution

**Name resolution** = mengubah **hostname** (`adinusa.id`) menjadi **IP** (`140.211.169.4`).

**Alurnya:**
1. Browser tanya sistem untuk resolve hostname.
2. Sistem cek **`/etc/hosts`** dulu (lokal).
3. Kalau tidak ada, tanya **DNS server**.
4. IP dipakai untuk komunikasi.

### 1. Static — `/etc/hosts`

File lokal berisi mapping IP ↔ hostname. Dicek **sebelum DNS**.

```
127.0.0.1    localhost
::1          ip6-localhost ip6-loopback
```

File terkait di `/etc/`:
- `/etc/hosts.allow` — izinkan koneksi
- `/etc/hosts.deny` — tolak koneksi
- `/etc/hostname` — hostname sistem

### 2. Dynamic — DNS

Kalau hostname tidak ada di `/etc/hosts`, Linux tanya **DNS server**.

**Konfigurasi di `/etc/resolv.conf`:**

```
nameserver 8.8.8.8
nameserver 127.0.0.53
options edns0 trust-ad
```

- `nameserver` → DNS server (urutan penting).
- Bisa di-set manual atau otomatis dari DHCP.
- NetworkManager sering rewrite file ini.

### Tools Cek DNS

| Tool | Fungsi |
|---|---|
| `dig adinusa.id` | Detail, modern |
| `host adinusa.id` | Ringkas |
| `nslookup adinusa.id` | Lama, deprecated |

**Reverse lookup** (IP → hostname):
```bash
$ dig -x 8.8.8.8
$ host 8.8.8.8
```

---

## 📘 Konsep Dasar — Network Diagnostics

Tools untuk troubleshoot koneksi, latency, routing, dan DNS.

### 1. `ping` — Uji Konektivitas

```bash
$ ping -c3 adinusa.id
```

- Kirim ICMP echo request.
- Tampilkan: reachable?, RTT (ms), packet loss.
- `-c3` → kirim 3 paket saja.

### 2. `traceroute` — Lihat Jalur Paket

```bash
$ traceroute google.com
```

- Tampilkan setiap **hop** (router) yang dilalui.
- Berguna untuk cari di mana delay/drop terjadi.

### 3. `mtr` — Gabungan ping + traceroute

```bash
$ mtr adinusa.id
```

- Real-time, update terus-menerus.
- Seperti `top` untuk network.
- Bagus untuk deteksi masalah intermiten.

### 4. `dig` — Query DNS

```bash
$ dig adinusa.id
```

- Tampilkan IP, DNS record (A, MX, CNAME), query time, nameserver.

---

## 🧪 Latihan Pemahaman

### Soal

1. Apa perbedaan `eth0` dan `enp0s3`?
2. Kenapa `ifconfig` deprecated?
3. Apa fungsi default route?
4. Di mana Linux cek hostname dulu — `/etc/hosts` atau DNS?
5. Tools apa untuk lihat jalur paket ke server?

### ✅ Jawaban

1. `eth0` = legacy (urutan deteksi). `enp0s3` = PNIDN (lokasi PCI bus 0, slot 3). PNIDN lebih stabil antar reboot.

2. Karena dukungan IPv6 terbatas, tidak ada fitur modern, dan tidak dikembangkan lagi. Digantikan `ip`.

3. Mengarahkan traffic yang **tidak match** route spesifik ke gateway. Biasanya didapat dari DHCP.

4. **`/etc/hosts`** dulu (static), baru DNS (dynamic).

5. **`traceroute`** (satu kali) atau **`mtr`** (real-time).

---

## 🐛 Troubleshooting & Kesalahan Umum

- **`ifconfig: command not found`:** Install `net-tools` — atau langsung pakai `ip a`.
- **Salah set default gateway = koneksi putus:** Cek dulu dengan `ip route`, baru ubah.
- **Interface tidak punya IP:** Cek `ip addr`, mungkin perlu DHCP atau set static.
- **DNS tidak resolve:** Cek `/etc/resolv.conf` dan `/etc/hosts`. Uji dengan `dig`.
- **Ping gagal tapi web tetap jalan:** Mungkin firewall block ICMP, tapi HTTP tetap lewat.
- **Traceroute stuck di hop tertentu:** Mungkin router di hop itu block ICMP — bukan berarti putus.

---

## 💡 Catatan & Insight Pribadi

### Analogi Network Device

Bayangkan interface seperti **pintu keluar-masuk rumah**:
- `eth0` = pintu lama (nomor urut).
- `enp0s3` = pintu dengan **alamat tetap** (lokasi ruang).
- Kalau punya banyak pintu, alamat tetap lebih mudah diingat.

### `ip` vs `ifconfig` — Kapan Pakai?

| Skenario | Pakai |
|---|---|
| Modern Linux | `ip` |
| Tutorial lama | `ifconfig` (tapi hati-hati) |
| Automation / script | `ip` |
| Debugging cepat | `ip a` |

### Alur Troubleshooting Network

```
1. ping 8.8.8.8          → ada koneksi internet?
2. ping google.com       → DNS bekerja?
3. ip addr               → interface punya IP?
4. ip route              → default gateway ada?
5. dig google.com        → DNS resolve?
6. traceroute            → di mana masalahnya?
```

### Kasus Nyata

- **Server tidak bisa SSH:** Cek `ip addr` dulu, mungkin interface DOWN.
- **Web lambat:** Jalankan `mtr` — cari hop yang latency-nya tinggi.
- **DNS error:** `dig` untuk cek, `/etc/resolv.conf` untuk config.
- **VPN tidak jalan:** Cek routing table dengan `ip route`.

### Pelajaran Kunci

1. **PNIDN** = penamaan modern, stabil antar reboot.
2. **`ip`** = tool modern, lengkap, direkomendasikan.
3. **`ifconfig`** = legacy, pakai kalau terpaksa.
4. **Routing** = pilih jalur; default route untuk traffic umum.
5. **Name resolution** = `/etc/hosts` dulu, baru DNS.
6. **Diagnostics**: `ping` → `traceroute` → `mtr` → `dig`.

---

## 🧠 Konsep yang Dikuasai

| Konsep | Fungsi |
|---|---|
| PNIDN | Penamaan interface modern (`enp0s3`, `ens3`) |
| `ip a` / `ip link` | Lihat & kelola interface |
| `ip route` | Lihat routing table |
| `ifconfig` | Legacy (butuh `net-tools`) |
| `/etc/hosts` | Static name resolution |
| `/etc/resolv.conf` | DNS server |
| `ping`, `traceroute`, `mtr`, `dig` | Diagnostics |

---

## 📌 Kesimpulan

Teori ini adalah fondasi untuk semua topik **networking** di Linux — dari konfigurasi interface, routing, DNS, sampai troubleshooting. Yang paling penting dipahami: **penamaan device modern** (`enp0s3`, bukan `eth0`), **`ip` sebagai tool utama**, **routing** (default route & static), **name resolution** (`/etc/hosts` vs DNS), dan **tools diagnostics** (`ping`, `traceroute`, `mtr`, `dig`). Kemampuan ini adalah bekal wajib sebelum masuk ke konfigurasi server produksi, cloud, atau container.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
