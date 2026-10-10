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

## 📘 IPv4 vs IPv6

| Aspek | IPv4 | IPv6 |
|---|---|---|
| Panjang | 32-bit | 128-bit |
| Format | `192.168.1.1` (4 oktet) | `2001:db8::1` (8 grup hex) |
| Jumlah | ~4.3 miliar | ~3.4 × 10³⁸ |

**IPv4** masih dominan, tapi mulai habis. **IPv6** dibuat untuk menggantikan — alamatnya praktis tak terbatas.

---

## 📘 Private & Reserved Address

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

## 📘 CIDR — Cara Modern Menentukan Network

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

## 📘 Hostname, Domain & FQDN

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
