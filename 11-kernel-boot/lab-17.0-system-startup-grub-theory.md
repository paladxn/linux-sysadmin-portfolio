# Lab 17.0 — System Startup & GRUB (Teori)

**Course:** Linux System Administration (Adinusa)
**Topic:** Teori boot sequence, BIOS/UEFI, boot loader, dan GRUB
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab teori ini, saya mampu:

- Memahami **boot sequence** Linux dan tahapannya
- Menjelaskan peran **BIOS/UEFI** dalam proses boot
- Memahami cara kerja **boot loader** (terutama GRUB)
- Melakukan **shutdown & reboot** dengan aman
- Menggunakan **GRUB interaktif** untuk troubleshooting

---

## 📘 Konsep Dasar — Boot Sequence

**Boot sequence** = proses komputer menginisialisasi hardware dan memuat OS setelah dinyalakan.

### Empat Tahap Utama

```
1. Power-On → BIOS/UEFI (POST)
2. Boot Loader (GRUB)
3. Kernel Initialization
4. Init / systemd
```

| Tahap | Yang Terjadi |
|---|---|
| **1. BIOS/UEFI** | POST (cek hardware) → cari boot device |
| **2. Boot Loader** | Tampilkan menu → load kernel ke memory |
| **3. Kernel** | Deteksi hardware → mount root FS → start PID 1 |
| **4. Init/systemd** | Start service → siapkan login |

**Kenapa penting dipahami?**
- Troubleshooting boot failure
- Konfigurasi boot loader
- Manajemen service startup

---

## 📘 Konsep Dasar — BIOS vs UEFI

| Aspek | BIOS (lama) | UEFI (modern) |
|---|---|---|
| Boot table | MBR | GPT |
| Max disk | 2 TB | Sangat besar |
| Boot mode | Legacy | Modern |
| Secure Boot | ❌ | ✅ |
| Kecepatan | Lambat | Cepat |

### Fungsi BIOS

- **Akses hardware dasar** — keyboard, display, disk, serial port
- **POST (Power-On Self-Test)** — cek memory, CPU, storage
- **Boot loader execution** — cari boot loader di MBR

> BIOS disimpan di **ROM chip** — selalu tersedia meskipun disk rusak.

---

## 📘 Konsep Dasar — Boot Loader

**Boot loader** = program yang memuat kernel Linux ke memory dan meneruskan kontrol ke OS.

### Jenis Boot Loader

| Boot Loader | Keterangan |
|---|---|
| **GRUB** | Default modern Linux — powerful, multi-OS |
| **efibootmgr** | Bukan boot loader, tapi boot **manager** untuk UEFI |
| **LILO** | Lama, obsolete |
| **Das U-Boot** | Embedded systems (IoT, router, ARM) |

### Fitur GRUB

- **Multi-OS boot** — Linux, Windows, dll
- **Multi-kernel** — pilih kernel & initramfs berbeda
- **Boot parameter customization** — edit parameter saat boot
- **Support filesystem modern** — ext4, XFS, Btrfs, GPT, EFI

---

## 📘 Konsep Dasar — Shutdown & Reboot

**Selalu shutdown dengan benar** — jangan langsung matikan power. Bisa menyebabkan:
- Data loss
- Filesystem corruption
- Service crash

### Perintah `shutdown`

**Syntax:**
```bash
shutdown [OPTIONS] [TIME] [MESSAGE]
```

| Opsi | Fungsi |
|---|---|
| `-h` / `--halt` | Halt (stop semua proses) |
| `-P` / `--poweroff` | Power off total |
| `-r` / `--reboot` | Reboot |
| `-k` | Kirim warning, tidak shutdown |
| `-c` | Cancel shutdown terjadwal |
| `--no-wall` | Tanpa broadcast message |

**Contoh:**
```bash
# Shutdown dalam 1 menit dengan pesan
$ sudo shutdown -h +1 "Maintenance"

# Shutdown sekarang
$ sudo shutdown -h now

# Reboot sekarang
$ sudo shutdown -r now

# Cancel shutdown terjadwal
$ sudo shutdown -c
```

### Perintah Legacy

| Perintah | Fungsi |
|---|---|
| `reboot` | Restart |
| `halt` | Halt |
| `poweroff` | Matikan |

> Di sistem modern, ketiganya adalah **wrapper** dari `systemctl` atau `shutdown`.

> [!] **Penting:** Selalu punya alasan valid sebelum shutdown/reboot server. Investigasi root cause dulu — reboot tidak selalu menyelesaikan masalah.

---

## 📘 Konsep Dasar — GRUB (Grand Unified Bootloader)

**GRUB** = boot loader default hampir semua distro Linux modern.

**Versi:**
- **GRUB Legacy** (GRUB 1) → lama, obsolete
- **GRUB 2** → standar saat ini (Ubuntu, Fedora, Debian, RHEL 7+)

### File Konfigurasi GRUB

| File | Fungsi |
|---|---|
| `/boot/grub/grub.cfg` | Config utama (auto-generated) |
| `/boot/grub2/grub.cfg` | Config utama (RHEL) |
| `/boot/efi/EFI/<distro>/grub.cfg` | Config UEFI |
| `/etc/default/grub` | **Setting utama** (timeout, kernel default) |
| `/etc/grub.d/` | Script generator menu |

> [X] **Jangan edit `grub.cfg` langsung** — auto-generated. Edit `/etc/default/grub` lalu regenerate.

### Regenerate Config

```bash
# Debian/Ubuntu
$ sudo update-grub

# RHEL/Fedora/CentOS
$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

---

## 📘 Konsep Dasar — GRUB Interaktif

### GRUB Boot Menu

Saat boot, GRUB menampilkan menu:
- Kernel Linux (berbagai versi)
- OS lain (dual-boot)
- Recovery mode
- Submenu

**Aksi:**
- **Arrow keys** → pilih entry
- **Enter** → boot entry
- **e** → edit config sementara
- **c** → masuk GRUB command line

### Edit Boot Parameter (Sementara)

```
1. Highlight entry
2. Tekan 'e'
3. Edit line 'linux' atau 'linux16'
4. Ctrl+X / F10 → boot dengan parameter baru
5. Esc → cancel
```

**Contoh — Single-user mode:**
```
linux ... ro quiet splash single
```

> [!] Edit ini **sementara** — hanya untuk boot saat ini. Untuk permanent, edit `/etc/default/grub` dan `update-grub`.

### GRUB Command Line Mode

Tekan **`c`** di menu → masuk GRUB shell.

**Bisa untuk:**
- Inspect disk/partisi: `ls`
- Set root & kernel manual
- Boot manual jika config broken
- Repair GRUB

> Mode ini **penyelamat** saat sistem gagal boot karena GRUB corrupt.

---

## 🧪 Latihan Pemahaman

### Soal

1. Sebutkan **4 tahap** boot sequence Linux secara berurutan.
2. Apa perbedaan **BIOS** dan **UEFI**?
3. Perintah untuk reboot sekarang?
4. Kenapa `grub.cfg` **tidak boleh** diedit langsung?
5. Bagaimana cara masuk **single-user mode** dari GRUB?

### ✅ Jawaban

1. **BIOS/UEFI (POST)** → **Boot Loader (GRUB)** → **Kernel Initialization** → **Init/systemd**.

2. **BIOS** = legacy, pakai MBR, max disk 2 TB. **UEFI** = modern, pakai GPT, support Secure Boot, lebih cepat.

3. `sudo shutdown -r now` atau `sudo reboot`.

4. Karena **auto-generated** — perubahan akan hilang saat regenerate. Edit `/etc/default/grub` lalu jalankan `sudo update-grub`.

5. Tekan **`e`** di menu GRUB → tambahkan **`single`** di akhir line `linux` → **Ctrl+X**.

---

## 🐛 Troubleshooting & Kesalahan Umum

- **Sistem tidak boot, stuck di GRUB prompt:** GRUB tidak menemukan config. Coba:
  ```
  grub> ls                    # lihat disk
  grub> ls (hd0,1)/boot       # cek file
  grub> set root=(hd0,1)
  grub> linux /boot/vmlinuz-... root=/dev/sda1
  grub> initrd /boot/initrd.img-...
  grub> boot
  ```

- **Lupa password root:** Boot ke single-user mode via GRUB (`single`), lalu `passwd`.

- **GRUB tidak muncul, langsung boot:** Timeout terlalu pendek atau `GRUB_TIMEOUT=0`. Edit `/etc/default/grub`.

- **Perubahan `/etc/default/grub` tidak berlaku:** Lupa `sudo update-grub`.

- **GRUB rusak setelah install Windows:** Windows menimpa MBR. Fix dengan Live USB:
  ```bash
  $ sudo grub-install /dev/sda
  $ sudo update-grub
  ```

- **Shutdown hang:** Cek proses yang masih running. Gunakan `shutdown -h +1` untuk beri waktu.

- **`sudo shutdown` tidak jalan:** Cek tidak ada shutdown terjadwal lain (`shutdown -c` dulu).

- **GRUB menu tidak muncul (UEFI):** Cek `efibootmgr` — pastikan boot entry GRUB ada.

- **Edit GRUB salah, sistem tidak boot:** Boot dari Live USB, mount root, edit `/etc/default/grub`, chroot, `update-grub`.

---

## 💡 Catatan & Insight Pribadi

### Analogi Boot Sequence

Bayangkan **menyalakan rumah**:
- **BIOS/UEFI** = saklar listrik utama — cek semua sistem
- **Boot Loader** = resepsionis — tunjukkan siapa yang masuk
- **Kernel** = manajer gedung — siapkan infrastruktur
- **systemd** = pekerja — nyalakan lampu, buka pintu

### BIOS vs UEFI — Analogi

- **BIOS** = pintu kayu jadul, kunci manual
- **UEFI** = pintu otomatis dengan fingerprint + CCTV

### GRUB Legacy vs GRUB 2

| Aspek | GRUB Legacy | GRUB 2 |
|---|---|---|
| Config | `menu.lst` | `grub.cfg` |
| Edit | Manual | Auto-generated |
| Script | Tidak | Ya (`/etc/grub.d/`) |
| Filesystem | Terbatas | Luas |

### Kapan Masuk GRUB Shell?

| Situasi | Aksi |
|---|---|
| Config GRUB rusak | Boot manual dari shell |
| Lupa password root | Boot single-user mode |
| Troubleshoot disk | `ls` untuk inspect partisi |
| Test parameter kernel | Edit sementara di menu |

### Best Practice

1. **Jangan edit `grub.cfg`** — edit `/etc/default/grub`.
2. **Selalu `update-grub`** setelah ubah config.
3. **Backup GRUB** sebelum ubah:
   ```bash
   $ sudo cp /boot/grub/grub.cfg /boot/grub/grub.cfg.bak
   ```
4. **Shutdown dengan sopan** — jangan cabut power.
5. **Selalu beri alasan** sebelum shutdown/reboot server.
6. **Single-user mode** untuk reset password (rescue mode).
7. **Test reboot** setelah ubah konfigurasi boot.

### Kasus Nyata

- **Server tidak boot setelah update:** Boot ke kernel lama dari GRUB menu.
- **Lupa password root:** Boot single-user, `passwd`, reboot.
- **Dual-boot Windows menimpa GRUB:** Live USB → `grub-install`.
- **Kernel panic setelah update driver:** GRUB → pilih kernel lama → uninstall driver.
- **Maintenance server:** `sudo shutdown -h +30 "Maintenance"` — beri waktu user.

### Pelajaran Kunci dari Teori Ini

1. **Boot sequence** = BIOS/UEFI → GRUB → Kernel → systemd.
2. **BIOS** = legacy (MBR), **UEFI** = modern (GPT, Secure Boot).
3. **GRUB** = boot loader default modern Linux.
4. **`/etc/default/grub`** = config yang diedit, bukan `grub.cfg`.
5. **`update-grub`** = regenerate config.
6. **GRUB interaktif** (`e` dan `c`) = penyelamat saat boot gagal.
7. **`shutdown`** = cara aman mematikan/reboot.

---

## 🧠 Konsep yang Dikuasai

| Konsep | Fungsi |
|---|---|
| Boot sequence | BIOS → GRUB → Kernel → systemd |
| BIOS vs UEFI | Legacy vs modern firmware |
| GRUB | Boot loader default Linux |
| `shutdown` | Perintah shutdown/reboot aman |
| `update-grub` | Regenerate konfigurasi GRUB |
| `/etc/default/grub` | Config utama GRUB |
| Single-user mode | Boot ke root shell untuk rescue |
| GRUB command line | Repair boot manual |

---

## 📌 Kesimpulan

Teori ini memberikan fondasi untuk memahami **bagaimana Linux menyala dari power button sampai login prompt**. Yang paling penting dipahami:

- **Boot sequence** terdiri dari 4 tahap: BIOS/UEFI → GRUB → Kernel → systemd.
- **BIOS** vs **UEFI** adalah dua firmware berbeda — UEFI modern lebih unggul.
- **GRUB** adalah pintu masuk ke sistem — bisa diinteraksi saat boot.
- **`/etc/default/grub`** dan `update-grub` adalah cara aman mengubah konfigurasi.
- **Single-user mode** adalah kunci darurat untuk reset password / rescue.

Kemampuan ini sangat relevan di dunia kerja — sysadmin hampir selalu butuh masuk GRUB untuk troubleshooting saat server gagal boot.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
