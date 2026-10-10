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
