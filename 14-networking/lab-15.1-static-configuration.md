# Lab 15.1 — Static Configuration (Netplan)

**Course:** Linux System Administration (Adinusa)
**Topic:** Konfigurasi IP statis dengan Netplan
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Menambahkan **network adapter** baru di VirtualBox (host-only)
- Mengonfigurasi **IP statis** untuk interface baru di Ubuntu
- Menerapkan konfigurasi dengan **Netplan**
- Memverifikasi interface dan IP dengan `ip` dan `ifconfig`

---

## 📘 Konsep Dasar

**Netplan** adalah tool konfigurasi jaringan modern di Ubuntu (sejak 17.10). Menggunakan file YAML di `/etc/netplan/` untuk mendefinisikan interface, IP, gateway, dan DNS.

| File / Perintah | Fungsi |
|---|---|
| `/etc/netplan/*.yaml` | File konfigurasi network |
| `netplan apply` | Terapkan konfigurasi |
| `netplan try` | Coba konfigurasi (rollback jika salah) |
| `ip addr show` | Lihat interface dan IP |
| `ip link set <if> up` | Aktifkan interface |

**Kenapa pakai Netplan?**
- Format YAML — mudah dibaca.
- Abstraksi dari backend (systemd-networkd, NetworkManager).
- Konfigurasi deklaratif — tidak perlu edit banyak file.

---

## 🔧 Persiapan Lab

### 1. Tambah network adapter di VirtualBox

1. **Shut down** VM.
2. **File → Tools → Network Manager**.
3. Pilih **Host-only Networks** → **Create** → **Apply**.
4. **VM Settings → Network → Adapter 3**:
   - ☑ Enable Network Adapter
   - Attached to: **Host-Only Adapter**
   - Name: **VirtualBox Host-Only Ethernet Adapter #2** (Linux: `vboxnet1`)
5. Klik **OK**, lalu **start VM**.

### 2. Login & start lab

```bash
$ nusactl login
$ nusactl start linlab-015-1
```

---

## 📘 Guided Example

### 1. Cek interface yang tersedia

```bash
$ ip link show
```

Output akan menampilkan interface baru: `enp0s9` (belum punya IP).

### 2. Edit file Netplan

```bash
$ sudo vim /etc/netplan/50-cloud-init.yaml
```

Isi file:

```yaml
network:
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      addresses:
      - 10.10.10.11/24
      dhcp4: false
    enp0s9: # From the New Network Adapter
      addresses:
      - 172.17.10.10/24
  version: 2
```

**Penjelasan:**
- `enp0s3` → interface lama, tetap DHCP.
- `enp0s8` → interface statis `10.10.10.11/24`.
- `enp0s9` → **interface baru** dengan IP statis `172.17.10.10/24`.
- `version: 2` → versi Netplan yang dipakai.

### 3. Terapkan konfigurasi

```bash
$ sudo netplan apply
$ sudo ip link set enp0s9 up
```

**Catatan:** `netplan apply` membaca YAML dan menerapkan ke backend. `ip link set up` mengaktifkan interface secara manual (kadang diperlukan setelah apply).

### 4. Verifikasi IP

**Opsi 1 — `ifconfig`:**

```bash
$ ifconfig
```

**Opsi 2 — `ip addr show`:**

```bash
$ ip addr show enp0s9
```

Output yang diharapkan:

```
3: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:xx:xx:xx brd ff:ff:ff:ff:ff:ff
    inet 172.17.10.10/24 brd 172.17.10.255 scope global enp0s9
       valid_lft forever preferred_lft forever
```

Perhatikan baris `inet 172.17.10.10/24` → IP statis sudah terpasang.

---

## 🧪 Practice Task

### Soal

1. Tambahkan adapter host-only baru (Adapter 4) di VirtualBox.
2. Konfigurasi interface baru (`enp0s10`) dengan IP statis `192.168.50.10/24`.
3. Terapkan dengan Netplan.
4. Verifikasi IP terpasang.
5. Uji ping dari host ke IP tersebut.

### ✅ Solusi

```bash
# 1. Tambah adapter di VirtualBox (via GUI)

# 2. Edit Netplan
$ sudo vim /etc/netplan/50-cloud-init.yaml
```

Tambahkan:

```yaml
    enp0s10:
      addresses:
      - 192.168.50.10/24
```

```bash
# 3. Terapkan
$ sudo netplan apply
$ sudo ip link set enp0s10 up

# 4. Verifikasi
$ ip addr show enp0s10

# 5. Uji ping dari host
$ ping 192.168.50.10
```

### 🔍 Hasil yang Diharapkan

- `ip addr show enp0s10` menampilkan `inet 192.168.50.10/24`.
- Ping dari host ke `192.168.50.10` berhasil.

---

## 🐛 Troubleshooting & Kesalahan

- **`netplan apply` gagal karena indentasi YAML salah:** YAML sangat sensitif terhadap indentasi — **wajib pakai spasi**, bukan Tab. Error umum: *"Invalid YAML at line X"*. Solusi: cek ulang pakai `sudo netplan try` yang akan rollback kalau ada error.

- **Interface tidak muncul di `ip link`:** Kemungkinan adapter di VirtualBox belum di-enable, atau nama interface berbeda. Cek dengan `ip link show` — nama bisa `enp0s9`, `enp0s10`, atau `eth2` tergantung tipe dan urutan.

- **IP tidak terpasang setelah `netplan apply`:** Terkadang butuh `ip link set enp0s9 up` manual. Atau reboot VM. Cek status dengan `ip addr`.

- **Interface DOWN:** Cek dengan `ip link show enp0s9`. Kalau `state DOWN`, jalankan `sudo ip link set enp0s9 up`.

- **Tidak bisa ping ke IP statis:** Cek apakah adapter host-only di sisi **host** sudah punya IP di subnet yang sama. Host-only adapter harus punya IP misal `172.17.10.1` di sisi host.

- **Perubahan hilang setelah reboot:** Pastikan file Netplan disimpan dengan benar dan `netplan apply` sukses. Kalau ada error saat apply, perubahan tidak persist.

- **`netplan try` timeout:** Perintah ini menunggu 120 detik untuk konfirmasi. Kalau tidak ada konfirmasi, akan rollback otomatis — ini fitur keamanan, bukan error.

- **Konflik dengan NetworkManager:** Di Ubuntu Desktop, NetworkManager bisa menimpa konfigurasi Netplan. Cek `/etc/netplan/` — pastikan hanya ada satu file konfigurasi aktif.

- **Interface tidak up otomatis:** Kalau `dhcp4: false` dan `addresses` di-set, interface **seharusnya** otomatis UP. Kalau tidak, tambahkan `optional: true` di konfigurasi.

- **YAML pakai Tab, bukan spasi:** Error klasik. YAML **tidak boleh** pakai Tab. Gunakan 2 spasi per level. Cek dengan `cat -A file.yaml`.

---

## 💡 Catatan & Insight Pribadi

### Kenapa Netplan?

Sebelum Netplan, Ubuntu pakai `/etc/network/interfaces` (ifupdown). Sekarang Netplan menggantikannya dengan YAML yang lebih terstruktur. Kelebihannya:
- Satu file untuk semua interface.
- Format konsisten.
- Abstraksi dari backend (systemd-networkd / NetworkManager).
- Rollback otomatis dengan `netplan try`.

### Struktur File Netplan

```yaml
network:
  version: 2                    # versi Netplan
  ethernets:                    # tipe interface
    enp0s3:                     # nama interface
      dhcp4: true               # pakai DHCP
    enp0s9:
      addresses:
      - 172.17.10.10/24         # IP statis
      gateway4: 172.17.10.1     # gateway (opsional)
      nameservers:
        addresses: [8.8.8.8]    # DNS (opsional)
```

### Kapan Pakai DHCP vs Static?

| Skenario | Pakai |
|---|---|
| Laptop / device mobile | DHCP |
| Server produksi | Static |
| VM testing | Static (biar konsisten) |
| Container | DHCP (dari runtime) |
| Network internal | Static |

### Perbedaan `ip` vs `ifconfig`

| Aspek | `ip` | `ifconfig` |
|---|---|---|
| Status | Modern, aktif dikembangkan | Legacy (deprecated) |
| Fitur | Lengkap | Terbatas |
| Instalasi | Built-in | Perlu `net-tools` |
| Rekomendasi | ✅ Pakai ini | ⚠️ Bisa dipakai tapi usang |

### Anatomi IP Statis

```
172.17.10.10/24
│           │  │
│           │  └── prefix length (24 bit network)
│           └───── host portion
└───────────────── network portion
```

- Network: `172.17.10.0`
- Broadcast: `172.17.10.255`
- Host range: `172.17.10.1 – 172.17.10.254`
- Gateway: biasanya `172.17.10.1` (di host)

### Kasus Nyata

- **Server produksi:** IP statis supaya DNS record konsisten.
- **Database cluster:** IP statis untuk replikasi antar node.
- **Kubernetes nodes:** IP statis untuk kestabilan cluster.
- **Home lab:** IP statis untuk VM supaya mudah di-SSH.

### Pelajaran Kunci dari Lab Ini

1. **Netplan** = tool konfigurasi network modern di Ubuntu.
2. File YAML di `/etc/netplan/` — **indentasi pakai spasi**, bukan Tab.
3. **`netplan apply`** menerapkan konfigurasi.
4. **`netplan try`** — rollback otomatis kalau error (aman untuk eksperimen remote).
5. **`ip addr`** dan **`ip link`** untuk verifikasi.
6. Interface baru perlu **diaktifkan manual** kadang.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `sudo vim /etc/netplan/*.yaml` | Edit konfigurasi network |
| `sudo netplan apply` | Terapkan konfigurasi |
| `sudo netplan try` | Coba dengan rollback otomatis |
| `ip link show` | Lihat daftar interface |
| `ip link set <if> up` | Aktifkan interface |
| `ip addr show <if>` | Lihat IP interface |
| `ifconfig` | Lihat interface (legacy) |
| `ping <ip>` | Uji konektivitas |

**Struktur YAML Netplan:**

```yaml
network:
  version: 2
  ethernets:
    <interface>:
      dhcp4: true|false
      addresses: [IP/prefix]
      gateway4: IP
      nameservers:
        addresses: [DNS]
```

---

## 📌 Kesimpulan

Lab ini memperkenalkan **konfigurasi IP statis dengan Netplan** — cara modern mengelola network di Ubuntu. Berbeda dari cara lama (`ifconfig` + `/etc/network/interfaces`), Netplan pakai YAML yang lebih terstruktur dan mendukung rollback otomatis. Kemampuan ini penting untuk sysadmin, terutama saat mengonfigurasi server dengan IP tetap, VM di lab, atau node cluster yang butuh alamat konsisten.

Kunci utamanya: **hati-hati dengan indentasi YAML**, selalu `netplan try` dulu sebelum `apply` (terutama di server remote), dan verifikasi dengan `ip addr`.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
