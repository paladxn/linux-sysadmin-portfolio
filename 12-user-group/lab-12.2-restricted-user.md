# Lab 12.2 — Membuat Restricted User

**Course:** Linux System Administration (Adinusa)
**Topic:** Restricted shell (`rbash`), limited sudo, dan batasan akses login
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membuat user dengan **restricted shell** (`rbash`)
- Membatasi **PATH** user hanya ke direktori tertentu
- Memberikan **sudo privilege terbatas** hanya untuk perintah spesifik
- Membatasi **remote login (SSH)** untuk group tertentu
- Memverifikasi semua batasan yang sudah diterapkan

---

## 📘 Konsep Dasar

| Konsep | Penjelasan |
|---|---|
| **`rbash` (restricted bash)** | Versi terbatas bash: tidak bisa `cd`, tidak bisa set PATH, tidak bisa jalankan command dengan `/`, tidak bisa redirect |
| **PATH terbatas** | PATH user hanya diarahkan ke direktori yang berisi symlink command yang diizinkan |
| **Symlink untuk command** | Command asli di system binary tidak diubah ownership-nya, cukup di-symlink ke direktori user |
| **Sudoers terbatas** | Memberi izin `sudo` hanya untuk perintah spesifik, bukan `ALL` |
| **`/etc/security/access.conf`** | Konfigurasi PAM untuk membatasi akses login berdasarkan user/group/sumber |

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-012-2
```

---

## 📘 Guided Example

### 1. Buat user `nusa`

```bash
$ sudo -i
$ useradd -m -s /bin/bash nusa
$ passwd nusa
New password: adinusa
Retype new password: adinusa
```

**Penjelasan opsi:**

| Opsi | Arti |
|---|---|
| `-m` | Buat home directory `/home/nusa` |
| `-s /bin/bash` | Set shell ke bash (nanti diganti `rbash`) |

Verifikasi:

```bash
$ grep nusa /etc/passwd
```

### 2. Aktifkan restricted shell

```bash
$ usermod -s /bin/rbash nusa
```

Sekarang shell user `nusa` adalah `/bin/rbash` — bukan `/bin/bash` biasa.

### 3. Buat direktori `bin` di home user

```bash
$ mkdir -p /home/nusa/bin
$ chown nusa:nusa /home/nusa/bin
$ chmod 755 /home/nusa/bin
```

Direktori ini akan menyimpan **symlink** ke command yang boleh dijalankan user.

### 4. Batasi PATH user

```bash
$ echo 'export PATH=$HOME/bin' >> /home/nusa/.bash_profile
$ chown nusa:nusa /home/nusa/.bash_profile
$ chmod 644 /home/nusa/.bash_profile
```

**Efeknya:** Setelah login, PATH user hanya berisi `/home/nusa/bin`. Command lain di system tidak akan ditemukan.

### 5. Buat symlink untuk command yang diizinkan

```bash
$ ln -s /bin/ls /home/nusa/bin/ls
$ ln -s /bin/cat /home/nusa/bin/cat
$ ln -s /usr/bin/sudo /home/nusa/bin/sudo
```

**Kenapa symlink, bukan copy?**
- Binary system tetap dimiliki root, permission tidak berubah.
- Symlink hanya pointer — user tidak bisa memodifikasi isi binary.
- Kalau binary di-update, symlink tetap valid.

**Pastikan ownership direktori benar:**

```bash
$ chown -R nusa:nusa /home/nusa
```

### 6. Buat restricted group

```bash
$ groupadd -f limitednusa
$ usermod -aG limitednusa nusa
```

**Penjelasan:** Opsi `-f` pada `groupadd` artinya "kalau group sudah ada, jangan error" (idempotent).

### 7. Konfigurasi sudoers terbatas

```bash
$ visudo -f /etc/sudoers.d/limitednusa
```

**Isi file:**

```
# /etc/sudoers.d/limitednusa
%limitednusa ALL=(ALL) NOPASSWD: /bin/ls, /bin/cat
```

**Penjelasan:**
- `%limitednusa` → semua anggota group `limitednusa`.
- `ALL=(ALL)` → di semua host, sebagai semua user.
- `NOPASSWD:` → tanpa diminta password.
- `/bin/ls, /bin/cat` → hanya dua perintah ini yang diizinkan.

Set permission:

```bash
$ chmod 440 /etc/sudoers.d/limitednusa
```

### 8. Batasi login SSH hanya dari local

```bash
$ echo "-:limitednusa:ALL EXCEPT LOCAL" >> /etc/security/access.conf
```

**Penjelasan sintaks `access.conf`:**

```
permission : users/groups : origins
```

| Bagian | Nilai | Arti |
|---|---|---|
| Permission | `-` | Tolak (deny) |
| User/Group | `limitednusa` | Group `limitednusa` |
| Origins | `ALL EXCEPT LOCAL` | Dari semua sumber **kecuali** login lokal |

**Efeknya:** Anggota group `limitednusa` **tidak bisa login via SSH**, hanya bisa login dari terminal lokal.

---

## 🧪 Verification

### Login sebagai `nusa`

```bash
$ su - nusa
```

### Test 1 — Command `which` tidak tersedia

```bash
nusa:~$ which ls
-rbash: /usr/lib/command-not-found: restricted: cannot specify `/' in command names
```

**Kenapa error?** `which` tidak ada di `/home/nusa/bin`, dan `rbash` tidak bisa menjalankan command dengan `/` di path.

**Alternatif:** Gunakan `type`:

```bash
nusa:~$ type ls
ls is /home/nusa/bin/ls

nusa:~$ type cat
cat is /home/nusa/bin/cat
```

### Test 2 — `sudo ls` (diizinkan)

```bash
nusa:~$ sudo ls /tmp/testdir
file1
```

Berhasil karena `ls` ada di daftar izin sudoers.

### Test 3 — `cd /` (diblokir `rbash`)

```bash
nusa:~$ cd /
-rbash: cd: restricted
```

Sesuai desain — `rbash` memang melarang `cd`.

### Test 4 — `sudo rm` (ditolak)

```bash
nusa:~$ sudo rm -rf /tmp/testdir
Sorry, user nusa is not allowed to execute '/usr/bin/rm -rf /tmp/testdir' as root.
```

**Penjelasan:** Karena `rm` tidak ada di daftar sudoers (`/bin/ls, /bin/cat`), sudo menolak perintah tersebut.

### Test 5 — Verifikasi batasan SSH

Coba login via SSH sebagai `nusa` dari mesin lain — seharusnya gagal karena konfigurasi `access.conf`.

---

## 🐛 Troubleshooting & Kesalahan

- **`usermod -s /bin/rbash` tapi `/bin/rbash` tidak ada:** Di beberapa distro, `rbash` bukan file terpisah, tapi symlink ke bash. Buat dulu:
  ```bash
  $ ln -s /bin/bash /bin/rbash
  ```
  Atau langsung set shell ke `/bin/bash -r` di `/etc/passwd` (kurang rapi).

- **PATH tetap menampilkan system directories:** Cek `/home/nusa/.bash_profile` — pastikan hanya berisi `export PATH=$HOME/bin`. Kalau ada isi lain dari `/etc/profile` yang menambahkan system directories, cari dan comment:
  ```bash
  # /etc/profile — pastikan tidak menambahkan /usr/bin, /bin, dll
  ```

- **Symlink tidak jalan di `rbash`:** Ingat `rbash` tidak bisa jalankan command dengan `/`. Karena symlink dibuat dengan path absolut (`/home/nusa/bin/ls`), ini bisa jadi masalah. Solusi: pastikan PATH-nya benar dan panggil tanpa `/` (`ls`, bukan `/home/nusa/bin/ls`).

- **`sudo` tidak ditemukan di PATH user:** Symlink `sudo` harus ada di `/home/nusa/bin/sudo`. Kalau tidak, user tidak bisa jalankan sudo sama sekali.

- **File di `/etc/sudoers.d/` diabaikan:** Cek permission — harus `0440` atau `0400`, dan **nama file tidak boleh mengandung `.`**. File seperti `limitednusa.conf` akan diabaikan.

- **`visudo -f` gagal:** Ini bisa terjadi kalau nama file mengandung karakter aneh, atau direktori tidak accessible. Cek syntax dengan:
  ```bash
  $ visudo -c
  ```

- **`access.conf` tidak berpengaruh:** Modul PAM `pam_access.so` harus aktif di `/etc/pam.d/sshd` dan `/etc/pam.d/login`. Cek dengan:
  ```bash
  $ grep pam_access /etc/pam.d/*
  ```

- **User masih bisa login SSH meskipun sudah diblokir:** Mungkin login pakai key-based auth yang bypass PAM. Batasi juga di `/etc/ssh/sshd_config`:
  ```
  DenyGroups limitednusa
  ```
  lalu restart SSH:
  ```bash
  $ systemctl restart sshd
  ```

- **`chown -R nusa:nusa /home/nusa` menghapus root ownership symlink:** Symlink selalu dimiliki user yang membuatnya (root), jadi `-R` tidak akan mengubah symlink. Aman.

- **User bisa bypass dengan `sudo bash`:** Kalau di sudoers hanya `ls` dan `cat`, user tidak bisa jalankan `sudo bash`. Tapi kalau `cat` bisa digunakan untuk membaca file sensitif — batasi juga dengan path yang aman.

- **`rbash` bukan security tool:** Sama seperti catatan di Lab 12 teori, `rbash` **mudah di-bypass** oleh user berpengalaman. Untuk keamanan sejati, gunakan SELinux, AppArmor, atau container.

---

## 💡 Catatan & Insight Pribadi

### Alur Restricted User

```
1. useradd -m -s /bin/bash → buat user
2. usermod -s /bin/rbash    → ganti shell jadi restricted
3. mkdir ~/bin              → direktori command yang diizinkan
4. export PATH=$HOME/bin    → batasi PATH
5. ln -s /bin/xxx ~/bin/    → symlink command yang diizinkan
6. groupadd limitednusa     → group untuk sudo
7. sudoers.d/limitednusa    → izin sudo terbatas
8. access.conf              → batasi login SSH
```

### Kenapa Pakai Symlink, Bukan Copy?

| Aspek | Symlink | Copy |
|---|---|---|
| Update binary | Otomatis ikut | Perlu copy ulang |
| Permission file | Sama dengan asli | Bisa berbeda |
| Disk usage | Hampir nol | Duplikat |
| Maintenance | Mudah | Rumit |

### Batasan `rbash` yang Perlu Diketahui

| Aksi | Diblokir? |
|---|---|
| `cd /` | ✅ Diblokir |
| `echo $PATH` | ❌ Tidak diblokir (tapi tidak bisa set) |
| `export PATH=/usr/bin` | ✅ Diblokir |
| `ls -la` | ❌ Tidak diblokir (kalau `ls` ada di PATH) |
| `/bin/ls` | ✅ Diblokir (ada `/`) |
| `ls > file.txt` | ✅ Diblokir (redirection) |
| `ls \| grep foo` | ⚠️ Tergantung implementasi |

### Kapan Restricted User Cocok Dipakai?

| Skenario | Cocok? |
|---|---|
| Guest account | ✅ |
| Demo environment | ✅ |
| Akun khusus aplikasi | ⚠️ Bisa, tapi container lebih baik |
| Akun developer | ❌ Terlalu ketat |
| Akun admin | ❌ Tidak cocok |

### Best Practice Sudoers

- **Spesifik perintah:** Jangan `ALL` kalau hanya butuh beberapa perintah.
- **Gunakan path lengkap:** `/bin/ls`, bukan `ls` — untuk mencegah path hijacking.
- **Hindari wildcard berlebihan:** `/bin/*` sama bahayanya dengan `ALL`.
- **Test dengan `sudo -l -U username`:** Pastikan hanya perintah yang diinginkan yang muncul.
- **Log semua sudo:** Default-nya sudah dicatat di `/var/log/auth.log`.

### Alternatif untuk Keamanan Lebih Kuat

| Metode | Kelebihan |
|---|---|
| **SELinux** | Mandatory access control, granular |
| **AppArmor** | Mirip SELinux, lebih mudah |
| **Container (Docker/LXC)** | Isolasi penuh |
| **chroot jail** | Batasi view filesystem |
| **systemd sandbox** | Batasi service dengan `ProtectSystem=`, dll |

### Audit Akses User

```bash
$ last nusa              # riwayat login user
$ lastlog -u nusa        # login terakhir
$ who                    # user yang sedang login
$ sudo cat /var/log/auth.log | grep nusa
```

### Kasus Nyata

- **Cloud VM:** Terkadang user diberikan akses SSH terbatas dan hanya bisa menjalankan script tertentu (`sudo /opt/scripts/deploy.sh`).
- **CI/CD runner:** User build sering dibatasi hanya boleh jalankan `git`, `docker`, `make`.
- **Jump host:** User hanya boleh `ssh` ke server lain, tidak bisa eksplorasi sistem.

### Pelajaran Kunci

1. **`rbash` + PATH terbatas + sudoers terbatas** = tiga lapis pertahanan.
2. **Symlink adalah cara elegan** untuk memberi akses ke command tanpa mengubah system binary.
3. **`access.conf`** efektif untuk membatasi login SSH, tapi butuh `pam_access.so` aktif.
4. **Test selalu dengan `su - user`** setelah konfigurasi — jangan asumsikan sudah jalan.
5. **`rbash` bukan security tool** — anggap sebagai *speed bump*, bukan *fortress*.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `useradd -m -s /bin/bash <user>` | Buat user + home + shell |
| `usermod -s /bin/rbash <user>` | Ganti shell ke restricted bash |
| `mkdir -p ~/bin` | Buat direktori command |
| `chown user:user ~/bin` | Set ownership |
| `chmod 755 ~/bin` | Set permission |
| `echo 'export PATH=$HOME/bin' >> ~/.bash_profile` | Batasi PATH |
| `ln -s /bin/ls ~/bin/ls` | Symlink command |
| `groupadd -f <group>` | Buat group (idempotent) |
| `usermod -aG <group> <user>` | Tambah user ke group |
| `visudo -f /etc/sudoers.d/<name>` | Edit sudoers spesifik |
| `chmod 440 /etc/sudoers.d/<name>` | Set permission sudoers |
| `echo "-:group:ALL EXCEPT LOCAL" >> /etc/security/access.conf` | Batasi login SSH |
| `su - <user>` | Test sebagai user |
| `type <cmd>` | Cek lokasi command (alternatif `which`) |

---

## 📌 Kesimpulan

Lab ini adalah praktik **membuat restricted user** dengan tiga lapis batasan:

1. **Restricted shell (`rbash`)** — membatasi perintah shell yang bisa dijalankan.
2. **PATH terbatas** — hanya direktori `~/bin` yang berisi symlink command yang diizinkan.
3. **Sudoers terbatas** — hanya perintah spesifik yang bisa dijalankan dengan privilege root.
4. **Tambahan:** Batasi login SSH lewat `access.conf`.

Konsep ini penting untuk skenario di mana kita perlu memberi akses terbatas ke sistem — seperti akun guest, akun untuk otomasi tertentu, atau akun untuk user yang hanya perlu menjalankan beberapa perintah tertentu. Namun, perlu diingat bahwa `rbash` **bukan security tool** — dia hanya *speed bump*. Untuk keamanan sejati, gunakan SELinux, AppArmor, atau container.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
