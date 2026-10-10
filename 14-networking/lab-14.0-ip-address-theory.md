# Lab 14.0 — IP Address & Hostname (Teori)

**Course:** Linux System Administration (Adinusa)
**Topic:** Teori dasar IP address, CIDR, dan hostname
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab teori ini, saya mampu:

- Membedakan **IPv4** dan **IPv6** serta formatnya
- Mengenali **private IP** dan **reserved address**
- Memahami **CIDR** (prefix length) untuk menentukan ukuran network
- Menjelaskan hubungan **hostname**, **domain**, dan **FQDN**

---

## 📘 Konsep Dasar — IPv4 vs IPv6

| Aspek | IPv4 | IPv6 |
|---|---|---|
| Panjang | 32-bit | 128-bit |
| Format | `192.168.1.1` (4 oktet) | `2001:db8::1` (8 grup hex) |
| Jumlah | ~4.3 miliar | ~3.4 × 10³⁸ |

**IPv4** masih dominan, tapi mulai habis. **IPv6** dibuat untuk menggantikan — alamatnya praktis tak terbatas.

---

## 📘 Konsep Dasar — Private & Reserved Address

### Private IP (dipakai di jaringan internal)

| Range | Blok | Umum di |
|---|---|---|
| `10.0.0.0 – 10.255.255.255` | `10/8` | Enterprise, cloud |
| `172.16.0.0 – 172.31.255.255` | `172.16/12` | Kantor, VPN |
| `192.168.0.0 – 192.168.255.255` | `192.168/16` | Home, WiFi |

> Private IP **tidak bisa diakses langsung dari internet**. Butuh NAT/router untuk keluar.

### Reserved Address

| Alamat | Fungsi |
|---|---|
| `127.0.0.1` | Loopback — tes lokal |
| `::1` | Loopback IPv6 |
| `0.0.0.0` | Unknown / DHCP request |
| `255.255.255.255` | Broadcast ke semua host di LAN |
| `fe80::/10` | Link-local IPv6 |

---

## 📘 Konsep Dasar — CIDR

CIDR pakai **prefix length** (`/24`, `/16`) untuk menentukan ukuran network.

| CIDR | Netmask | Host | Umum dipakai |
|---|---|---|---|
| `/8` | `255.0.0.0` | ~16 juta | ISP besar |
| `/16` | `255.255.0.0` | ~65.000 | Enterprise |
| `/24` | `255.255.255.0` | 254 | Kantor, home |
| `/30` | `255.255.255.252` | 2 | Link point-to-point |
| `/32` | `255.255.255.255` | 1 | Single host |

**Rumus host:** `2^(32 - prefix) - 2` (dikurangi network & broadcast address).

**Contoh:**
- `192.168.1.0/24` → 254 host (`192.168.1.1 – 192.168.1.254`)
- `10.0.0.0/8` → ~16 juta host

---

## 📘 Konsep Dasar — Hostname, Domain & FQDN

| Istilah | Contoh | Arti |
|---|---|---|
| **Hostname** | `academy` | Nama pendek perangkat |
| **Domain** | `adinusa.id` | Nama domain |
| **FQDN** | `academy.adinusa.id` | Gabungan lengkap |

**Melihat & mengubah hostname:**

```bash
$ hostname                        # lihat hostname
$ sudo hostname server01          # ubah sementara (hilang setelah reboot)
$ sudo hostnamectl set-hostname server01   # ubah permanent
$ hostnamectl                     # lihat info lengkap
```

---

## 🧪 Latihan Pemahaman

### Soal

Jawab pertanyaan berikut untuk menguji pemahaman:

1. Apa perbedaan utama IPv4 dan IPv6?
2. Range private IP mana yang biasa dipakai di rumah?
3. Berapa jumlah host yang tersedia di network `/24`?
4. Apa perbedaan hostname, domain, dan FQDN?
5. Alamat `127.0.0.1` digunakan untuk apa?

### ✅ Jawaban

1. **IPv4** = 32-bit, format desimal (`192.168.1.1`). **IPv6** = 128-bit, format heksadesimal (`2001:db8::1`). IPv6 menyediakan alamat jauh lebih banyak.

2. `192.168.0.0/16` — biasa dipakai home router (`192.168.1.1`).

3. **254 host** (`2^8 - 2`). Dikurangi 2 karena network address dan broadcast address.

4. **Hostname** = nama pendek (`academy`). **Domain** = nama domain (`adinusa.id`). **FQDN** = gabungan keduanya (`academy.adinusa.id`).

5. **Loopback** — untuk tes lokal tanpa lewat jaringan fisik. Alias `localhost`.

---

## 🐛 Troubleshooting & Kesalahan Umum

- **Bingung IPv4 vs IPv6:** Kalau ada titik → IPv4. Kalau ada titik dua → IPv6.

- **Salah hitung jumlah host di CIDR:** Rumus `2^(32 - prefix) - 2`. Dikurangi 2 karena network address dan broadcast address tidak bisa dipakai host.

- **Private IP tidak bisa ping ke internet:** Wajar — private IP tidak routable. Butuh NAT/router untuk keluar.

- **`127.0.0.1` vs `localhost`:** `localhost` adalah hostname yang resolve ke `127.0.0.1`. Di sistem dual-stack bisa juga `::1`.

- **Netmask terbalik:** Netmask selalu **1 berurutan dari kiri**. `255.255.0.0` valid, `255.0.255.0` tidak valid.

- **Hostname tidak bisa di-resolve:** Cek `/etc/hosts` — pastikan ada entry `127.0.1.1 <hostname>`. Atau cek konfigurasi DNS.

- **Salah tafsir CIDR:** `/24` bukan berarti "24 host" — melainkan **24 bit** untuk network portion, sisanya 8 bit untuk host (254 host).

- **Lupa perbedaan hostname & FQDN:** Hostname = nama pendek. FQDN = lengkap dengan domain.

---

## 💡 Catatan & Insight Pribadi

### Analogi IP Address

IP address seperti **alamat rumah**:
- **Network portion** → nama jalan + kode pos.
- **Host portion** → nomor rumah.
- **Router** → kantor pos yang menerjemahkan alamat private ke publik.

### Kapan Pakai IPv4 vs IPv6?

- **Internet saat ini** → IPv4 (masih dominan).
- **Server modern** → dual-stack (IPv4 + IPv6).
- **Cloud & data center** → IPv6 untuk skala besar.

### Tips Menghafal Private IP

- `10.x.x.x` → "**s**atu jaringan besar" (enterprise).
- `172.16–31.x.x` → "**t**engah" (VPN, kantor).
- `192.168.x.x` → "**r**umah" (home router).

### CIDR yang Sering Muncul

| CIDR | Kapan ketemu |
|---|---|
| `/32` | Single host (firewall rule) |
| `/30` | Link antar router |
| `/24` | Subnet kantor / home |
| `/16` | VPC cloud (AWS, GCP) |
| `/8` | ISP, enterprise besar |

### Kasus Nyata

- **Home network:** Router `192.168.1.1`, device `192.168.1.2–254`.
- **Kantor:** Network `10.0.0.0/8`, server `10.0.1.5`.
- **Cloud (AWS/GCP):** VPC pakai CIDR `10.0.0.0/16` atau `172.16.0.0/12`.
- **Data center:** `/24` per rack, subnet per layanan.

### Pelajaran Kunci

1. **IPv4** 32-bit, **IPv6** 128-bit.
2. **Private IP** tidak routable di internet — butuh NAT.
3. **CIDR** menggantikan kelas A/B/C — lebih fleksibel.
4. **FQDN** = hostname + domain.
5. **`hostnamectl`** = cara modern set hostname.

---

## 🧠 Konsep yang Dikuasai

| Konsep | Fungsi |
|---|---|
| IPv4 / IPv6 | Format alamat |
| Private IP | Jaringan internal |
| Loopback (`127.0.0.1`, `::1`) | Tes lokal |
| CIDR (`/24`, `/16`) | Ukuran network |
| Hostname / FQDN | Identitas perangkat |
| `hostname` / `hostnamectl` | Lihat & ubah hostname |

---

## 📌 Kesimpulan

Teori ini adalah fondasi untuk semua topik **networking** di Linux — dari konfigurasi interface, routing, DNS, sampai firewall. Yang paling penting dipahami: **perbedaan IPv4 vs IPv6**, **private vs public IP**, **CIDR**, dan **hostname/FQDN**. Sisanya (sejarah kelas A/B/C, detail IPv6 types, netmask per kelas) bisa dipelajari nanti kalau butuh.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
