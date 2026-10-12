# Lab 19.0 — Basic Troubleshooting (Teori)

**Course:** Linux System Administration (Adinusa)
**Topic:** Teori dasar troubleshooting — level, teknik, networking, boot failure, rescue mode
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab teori ini, saya mampu:

- Memahami **level troubleshooting** di Linux
- Menerapkan **teknik troubleshooting** sistematis
- Memeriksa komponen **networking** saat troubleshooting
- Memahami penyebab & solusi **boot failure**
- Menggunakan **Rescue Mode** untuk perbaikan darurat

---

## 📘 Konsep Dasar — Level Troubleshooting

Troubleshooting = proses diagnosis & resolusi masalah. Skill ini berkembang dengan latihan, dibagi 3 level:

| Level | Karakteristik | Kekuatan | Keterbatasan |
|---|---|---|---|
| **Beginner** | Ikut panduan, tangani isu rutin | Cepat belajar tool dasar (`ping`, `systemctl`, `df`, `top`) | Belum paham root cause, sering eskalasi |
| **Experienced** | 2+ tahun praktik, tangani masalah kompleks | Paham "kenapa", bisa bedakan hardware/software/config | Masih perlu eskalasi untuk kasus langka |
| **Wizard** | Expert, go-to orang di organisasi | Debug kernel panic, RAID corrupt, security breach | Bisa burnout kalau semua masalah dibebankan ke dia |

**Key point:** Bahkan sistem terbaik akan bermasalah. Yang membedakan recovery mulus vs bencana:
- **Metodis** dalam langkah troubleshooting
- **Bedakan** local vs network vs Internet issue
- **Bangun expertise** — ada wizard, tapi juga latih beginner & experienced

---

## 📘 Konsep Dasar — Teknik Troubleshooting

Troubleshooting = **iterative process** (observasi → test → verifikasi).

### Resep 6 Langkah

| # | Langkah | Aksi |
|---|---|---|
| 1 | **Characterize problem** | Definisikan apa yang rusak, sistem mana, kapan mulai, gejala apa |
| 2 | **Reproduce problem** | Pastikan bisa direproduksi — bedakan glitch vs real problem |
| 3 | **Try easy things first** | Cek service running, kabel, restart service |
| 4 | **Eliminate one by one** | Divide and conquer — network → log → firewall |
| 5 | **Change one thing at a time** | Isolasi perubahan. Kalau gagal, revert |
| 6 | **Check system logs** | `/var/log/messages`, `/var/log/secure`, service log |

### Dua Filosofi Troubleshooting

| Filosofi | Kapan | Catatan |
|---|---|---|
| **Checklist** | Enterprise, konsistensi | Bisa diikuti siapa pun, reproducible |
| **Intuisi** | Pengalaman panjang | Bukan magic — hasil pengalaman berulang |

> **Tips:** Saat mengikuti hunch, **kumpulkan data cepat** untuk konfirmasi/discard sebelum investasi waktu lebih banyak.

---

## 📘 Konsep Dasar — Networking Troubleshooting

Lima area yang harus dicek berurutan:

### 1. IP Configuration

```bash
$ ip addr show
$ ifconfig

$ ip addr show enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    inet 192.168.1.10/24 brd 192.168.1.255 scope global dynamic enp0s3
```

Kalau tidak ada IP → cek DHCP atau misconfig manual.

### 2. Network Driver

```bash
$ lsmod | grep <driver>
$ dmesg | grep eth
$ cat /proc/interrupts
$ ls /sys/class/net/
```

Kalau NIC tidak muncul di `/sys/class/net/` → driver issue.

### 3. Connectivity

```bash
$ ping -c3 8.8.8.8          # basic connectivity
$ traceroute google.com      # lihat jalur
$ mtr google.com             # continuous diagnosis
```

- Gagal `ping 8.8.8.8` → masalah **lokal** (driver, kabel, interface)
- Berhasil ping tapi web tidak buka → masalah **DNS**

### 4. Default Gateway & Routing

```bash
$ ip route show
$ route -n

Destination     Gateway         Genmask         Iface
0.0.0.0         192.168.1.1     0.0.0.0         enp0s3
192.168.1.0     0.0.0.0         255.255.255.0   enp0s3
```

Tanpa default route → paket tidak bisa keluar LAN.

### 5. Hostname Resolution (DNS)

```bash
$ dig adinusa.id
$ host adinusa.id
$ cat /etc/resolv.conf
```

Ping IP berhasil tapi DNS gagal → masalah name resolution.

---

## 📘 Konsep Dasar — Boot Process Failures

**5 tahap boot:**

```
1. BIOS/UEFI          → init hardware, cari boot device
2. Boot loader (GRUB) → load kernel
3. Kernel loading     → init driver & memory
4. Root FS mounting   → sediakan partisi root
5. Init/systemd       → start service & user process
```

Kegagalan bisa terjadi di **tahap mana pun**. Kenali gejala → cocokkan penyebab.

### Empat Kegagalan Umum

| # | Gejala | Penyebab Umum | Solusi |
|---|---|---|---|
| 1 | **GRUB tidak muncul** | Boot sector corrupt, GRUB misconfig, urutan boot salah | Cek BIOS/UEFI; Live USB → `grub-install /dev/sda` + `update-grub` |
| 2 | **Kernel panic** | Kernel corrupt/missing, parameter salah, hardware unsupported | Edit GRUB (`e`) → simplify param; boot kernel lama; reinstall kernel |
| 3 | **Root FS gagal mount** | UUID/partisi salah, fstab salah, driver hilang di initramfs | Cek `root=UUID=` di GRUB; rescue mode → fix fstab; `update-initramfs -u` |
| 4 | **Failure di init/systemd** | FS corrupt, binary hilang, service misconfig | Baca error; single-user mode; `journalctl -xb`; `fsck /dev/sdX` |

---

## 📘 Konsep Dasar — Rescue Mode

**Rescue Mode** = environment minimal untuk perbaikan darurat saat sistem tidak bisa boot normal.

### Kapan Dipakai?

- GRUB corrupt / hilang
- Kernel panic saat boot
- Root filesystem gagal mount
- `/etc/fstab` salah
- Critical service halangi login normal

### Cara Masuk Rescue Mode

**Dari GRUB Menu:**

```
1. Boot → tahan Shift (BIOS) atau Esc/Shift (UEFI) setelah POST
2. GRUB menu → Advanced options for Ubuntu
3. Pilih entry dengan (recovery mode)
4. Recovery Menu muncul:
   - resume  → boot normal
   - clean   → free disk space
   - dpkg    → repair broken packages
   - fsck    → check filesystem
   - grub    → update bootloader
   - network → enable networking
   - root    → root shell  ← pilih ini
5. Masuk root shell dengan FS read-only
6. Remount sebagai read-write:
   $ mount -o remount,rw /
7. Perbaiki sistem
```

**Dari Live CD/USB (kalau GRUB rusak):**

```bash
$ sudo mount /dev/sdaX /mnt
$ sudo chroot /mnt
# repair boot loader, kernel, fstab
```

### Rescue Mode vs Emergency Mode

| Mode | Services | Akses | FS |
|---|---|---|---|
| **Rescue** | Minimal | Root login | Read-only sampai diperbaiki |
| **Emergency** | Lebih sedikit | Manual recovery | Sama |

### Aksi Umum di Rescue Mode

```bash
# Repair filesystem
$ fsck /dev/sdaX

# Fix /etc/fstab (UUID/filesystem type salah)
$ nano /etc/fstab

# Reinstall GRUB
$ grub2-install /dev/sda
$ grub2-mkconfig -o /boot/grub2/grub.cfg

# Lihat log
$ journalctl -xb

# Remount RW
$ mount -o remount,rw /
```

---

## 🧪 Latihan Pemahaman

### Soal

1. Sebutkan **3 level** troubleshooting.
2. Apa langkah **pertama** dalam troubleshooting?
3. Bagaimana membedakan masalah **lokal** vs **DNS**?
4. Gejala: GRUB tidak muncul. Apa penyebab & solusinya?
5. Bagaimana cara remount root FS jadi read-write di rescue mode?

### ✅ Jawaban

1. **Beginner** (ikut panduan), **Experienced** (2+ tahun, paham "kenapa"), **Wizard** (expert, debug kernel-level).

2. **Characterize problem** — definisikan apa yang rusak, sistem mana, kapan mulai, gejala apa.

3. **Ping IP dulu** (`ping 8.8.8.8`):
   - Gagal → masalah **lokal** (driver, kabel, interface)
   - Berhasil tapi DNS gagal → masalah **DNS**

4. **Gejala:** GRUB tidak muncul. **Penyebab:** boot sector corrupt, GRUB misconfig, urutan boot salah. **Solusi:** cek BIOS/UEFI, atau Live USB → `grub-install /dev/sda` + `update-grub`.

5. `mount -o remount,rw /`

---

## 🐛 Troubleshooting & Kesalahan Umum

- **Ubah banyak hal sekaligus:** Sulit melacak mana yang memperbaiki/memperburuk. Ubah **satu hal, test**.

- **Skip "easy things first":** Langsung curiga hal rumit, padahal cuma service yang mati. Selalu cek basic dulu.

- **Tidak cek log:** Log adalah sumber clue terbesar — `/var/log/`, `journalctl`.

- **Mengabaikan warning:** Warning `AH00558` di Apache sering dianggap tidak penting, tapi bisa jadi gejala masalah lebih besar.

- **Langsung reinstall:** 99% masalah bisa di-fix tanpa reinstall. Reinstall malah menghilangkan bukti.

- **Lupa remount RW di rescue mode:** FS read-only by default. Kalau lupa `mount -o remount,rw /`, edit config akan gagal.

- **Salah partisi di Live USB:** `mount /dev/sdaX /mnt` — pastikan `X` benar. Cek dengan `lsblk` dulu.

- **`update-grub` di distro RHEL:** Perintahnya `grub2-mkconfig -o /boot/grub2/grub.cfg`, bukan `update-grub`.

- **Mengabaikan konteks:** Troubleshooting server produksi ≠ lab. Di produksi, koordinasi dulu sebelum restart.

---

## 💡 Catatan & Insight Pribadi

### Analogi Troubleshooting

Seperti **detektif**:
- **Characterize** = identifikasi korban
- **Reproduce** = rekonstruksi kejadian
- **Eliminate** = buang tersangka satu per satu
- **Change one thing** = jangan kontaminasi bukti
- **Check logs** = cek rekaman CCTV

### Checklist vs Intuisi

- **Checklist** = SOP — bisa diikuti siapa pun, konsisten, reproducible.
- **Intuisi** = hasil pengalaman — bukan magic, tapi "pola" yang sudah sering dilihat.

**Best practice:** Kombinasi keduanya. Ikuti checklist untuk hal rutin, pakai intuisi saat checklist mentok.

### Kapan Troubleshooting Berhenti?

Ketika:
1. Masalah **hilang** (verified dengan test).
2. Root cause **teridentifikasi** (bukan cuma "restart udah bisa").
3. Dokumentasi **update** — pelajaran dicatat.

Kalau cuma restart tanpa tahu root cause → masalah akan kembali.

### Networking: Urutan Cek

```
1. ip a              → interface punya IP?
2. ping 8.8.8.8      → konektivitas dasar?
3. ping google.com   → DNS jalan?
4. ip route          → gateway ada?
5. dig google.com    → DNS server resolve?
6. traceroute        → di mana paket berhenti?
```

### Boot Failure: Kategorikan Berdasarkan Tahap

| Tahap | Gejala | Tool |
|---|---|---|
| BIOS/UEFI | Tidak ada POST, tidak cari boot device | BIOS setup |
| GRUB | Tidak muncul menu | Live USB, `grub-install` |
| Kernel | Kernel panic | GRUB `e`, boot kernel lama |
| Root FS | "Cannot mount /" | Rescue mode, fix fstab |
| Init/systemd | Service error | Single-user, `journalctl -xb` |

### Rescue Mode sebagai "Pintu Darurat"

Analoginya: kalau rumah **terkunci dari dalam**, kamu butuh **pintu darurat** untuk masuk. Rescue mode = pintu darurat sistem operasi.

**Kapan butuh:**
- Lupa password root → reset
- fstab salah → fix
- GRUB corrupt → reinstall
- Service crash sistem → disable sementara

### Kasus Nyata

- **Server produksi tidak boot setelah update:** Rescue mode → pilih kernel lama → uninstall driver bermasalah → reboot.
- **Lupa password root:** Rescue mode → `passwd` → reboot.
- **GRUB corrupt setelah dual-boot Windows:** Live USB → `grub-install` + `update-grub`.
- **Filesystem corrupt:** Rescue mode → `fsck /dev/sdaX` → reboot.

### Pelajaran Kunci dari Teori Ini

1. **Troubleshooting** = skill yang dilatih, bukan bakat.
2. **6 langkah resep** — ikuti urutannya.
3. **Ubah satu hal, test** — jangan ubah banyak sekaligus.
4. **Networking troubleshooting** berurutan: IP → driver → ping → route → DNS.
5. **Boot failure** dikategorikan berdasarkan tahap.
6. **Rescue mode** = pintu darurat sistem.
7. **`mount -o remount,rw /`** wajib sebelum edit di rescue.
8. **Log adalah teman terbaik** troubleshooting.

---

## 🧠 Konsep yang Dikuasai

| Konsep | Fungsi |
|---|---|
| Level troubleshooting | Beginner → Experienced → Wizard |
| Resep 6 langkah | Characterize → Reproduce → Easy → Eliminate → One change → Log |
| Networking check | IP → driver → ping → route → DNS |
| Boot stages | BIOS → GRUB → Kernel → Root FS → systemd |
| Boot failure categories | No GRUB, kernel panic, root mount, init |
| Rescue Mode | Environment minimal untuk perbaikan |
| Emergency Mode | Lebih minimal dari rescue |
| `mount -o remount,rw /` | Remount FS read-write |
| `journalctl -xb` | Log dari boot terakhir |
| `fsck /dev/sdX` | Check & repair filesystem |

---

## 📌 Kesimpulan

Teori ini memberikan fondasi untuk semua topik **troubleshooting** di lab-lab berikutnya. Yang paling penting dipahami:

- **Troubleshooting = iterative process** — bukan satu langkah ajaib.
- **Level skill** berkembang dengan pengalaman (beginner → wizard).
- **Networking troubleshooting** berurutan: IP → driver → connectivity → route → DNS.
- **Boot failure** dikategorikan per tahap — memudahkan isolasi masalah.
- **Rescue mode** = pintu darurat — kapan, kenapa, dan bagaimana.

Kemampuan ini adalah **inti dari sysadmin**. Server produksi pasti akan bermasalah. Yang membedakan sysadmin baik dan buruk adalah **kemampuan diagnosis** — bukan kemampuan menghafal perintah.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
