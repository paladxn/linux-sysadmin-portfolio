# Lab Challenge 5 — Manage User Accounts

**Course:** Linux System Administration (Adinusa)
**Topic:** Advanced user management — comment, expiry, shell, password policy, group
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membuat user dengan **advanced options** (comment, shell, expiry)
- Mengonfigurasi **password policies** dan keamanan akun
- Bekerja dengan **supplementary group**
- **Verifikasi** dan troubleshooting konfigurasi user account

---

## 📘 Konsep Dasar

### Perintah Utama

| Perintah | Fungsi |
|---|---|
| `useradd` | Buat user baru |
| `passwd` | Set password |
| `usermod` | Modifikasi atribut user |
| `chage` | Konfigurasi password aging |
| `groupadd` | Buat group baru |
| `id` | Verifikasi user & group |
| `groups` | Lihat daftar group user |

### File Penting

| File | Isi |
|---|---|
| `/etc/passwd` | Data user (nama, UID, GID, comment, home, shell) |
| `/etc/shadow` | Hash password & aging info |
| `/etc/group` | Data group & anggota |

### Comment (GECOS Field)

Comment = field ke-5 di `/etc/passwd`. Biasanya berisi nama lengkap user. Di-set dengan `usermod -c "Comment" user`.

### Password Aging (`chage`)

| Opsi | Fungsi |
|---|---|
| `chage -M N` | Max days password valid |
| `chage -m N` | Min days before change |
| `chage -W N` | Warning days |
| `chage -d 0` | Force change password on next login |
| `chage -E YYYY-MM-DD` | Account expiration |

### Lock vs Delete

- **Lock** (`usermod -L`) — nonaktifkan login tanpa hapus data.
- **Delete** (`userdel -r`) — hapus user + home directory.
- Untuk user yang cuti/resign, **lock** lebih baik daripada delete.

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-challenge-5
```

---

## 🧪 Challenge Tasks

### Soal

1. Buat user **trainer1**, **trainer2**, **trainer3** dengan:
   - Password: `Train!@#123`
   - Default shell: `/bin/bash`

2. Tambahkan comment:
   - `trainer1` → `Trainer One`
   - `trainer2` → `Trainer Two`
   - `trainer3` → `Trainer Three`

3. Konfigurasi kebijakan akun:
   - Max password age **trainer1**: **7 hari**
   - **Lock** akun `trainer2`
   - **Force change password** `trainer3` saat login pertama

4. Buat group baru bernama **`coaches`**.

5. Tambahkan `trainer1` dan `trainer2` ke group **`coaches`** — **tanpa** mengubah primary group.

6. Ubah default shell `trainer3` menjadi **`/bin/sh`**.

### ✅ Solusi

```bash
# 1. Buat user dengan shell bash
$ sudo useradd -m -s /bin/bash trainer1
$ sudo useradd -m -s /bin/bash trainer2
$ sudo useradd -m -s /bin/bash trainer3

# 2. Set password (non-interaktif pakai chpasswd)
$ echo "trainer1:Train!@#123" | sudo chpasswd
$ echo "trainer2:Train!@#123" | sudo chpasswd
$ echo "trainer3:Train!@#123" | sudo chpasswd

# 3a. Comment untuk tiap user
$ sudo usermod -c "Trainer One" trainer1
$ sudo usermod -c "Trainer Two" trainer2
$ sudo usermod -c "Trainer Three" trainer3

# 3b. Max password age trainer1: 7 hari
$ sudo chage -M 7 trainer1

# 3c. Lock akun trainer2
$ sudo usermod -L trainer2
# atau: sudo passwd -l trainer2

# 3d. Force change password trainer3 saat login
$ sudo chage -d 0 trainer3

# 4. Buat group coaches
$ sudo groupadd coaches

# 5. Tambahkan trainer1 & trainer2 ke coaches (append)
$ sudo usermod -aG coaches trainer1
$ sudo usermod -aG coaches trainer2

# 6. Ubah shell trainer3 ke /bin/sh
$ sudo usermod -s /bin/sh trainer3
```

### 🔍 Penjelasan Opsi Penting

**`useradd -m -s /bin/bash`**
- `-m` = buat home directory (`/home/trainerX`).
- `-s /bin/bash` = set default shell.

**`echo "user:pass" | chpasswd`**
- Non-interaktif — set password dalam satu perintah.
- Lebih cepat daripada `passwd` yang interaktif.
- Password harus dalam plain text, tidak boleh ada karakter `:`.

**`usermod -c "Comment"`**
- Set **GECOS field** (comment) di `/etc/passwd`.

**`chage -M 7 trainer1`**
- Max 7 hari password valid.
- Setelah 7 hari, user **wajib** ganti password.

**`usermod -L trainer2`**
- Lock akun — **tidak bisa login**.
- Cara kerja: tambah `!` di depan hash password di `/etc/shadow`.
- Unlock: `usermod -U trainer2`.

**`chage -d 0 trainer3`**
- Set "last password change" ke epoch 0.
- Efek: user **wajib ganti password** saat login berikutnya.

**`usermod -aG coaches`**
- `-aG` = **append** + supplementary group.
- **WAJIB pakai `-a`**. Kalau hanya `-G`, group lama akan **diganti**.
- Primary group **tidak berubah**.

**`usermod -s /bin/sh trainer3`**
- Ganti default shell. `/bin/sh` biasanya symlink ke `dash` (POSIX shell).

---

## 🔍 Verifikasi

### 1. Cek user sudah dibuat

```bash
$ grep -E "trainer[123]" /etc/passwd
```

Output yang diharapkan (contoh):
```
trainer1:x:1001:1001:Trainer One:/home/trainer1:/bin/bash
trainer2:x:1002:1002:Trainer Two:/home/trainer2:/bin/bash
trainer3:x:1003:1003:Trainer Three:/home/trainer3:/bin/sh
```

**Perhatikan:**
- Kolom ke-5 = comment (`Trainer One`, dll).
- Kolom ke-7 = shell (`/bin/bash` atau `/bin/sh`).

### 2. Cek password aging

```bash
# Max age trainer1 harus 7
$ sudo chage -l trainer1 | grep "Maximum"
Maximum number of days between password change          : 7

# trainer3 harus force change saat login
$ sudo chage -l trainer3 | grep "Last"
Last password change                                    : password must be changed
```

### 3. Cek status lock trainer2

```bash
$ sudo passwd -S trainer2
trainer2 L ...    ← 'L' = Locked
```

### 4. Cek group coaches

```bash
$ grep coaches /etc/group
coaches:x:1004:trainer1,trainer2
```

### 5. Cek group membership user

```bash
$ id trainer1
uid=1001(trainer1) gid=1001(trainer1) groups=1001(trainer1),1004(coaches)
                                       ↑             ↑
                                    primary    supplementary

$ id trainer2
uid=1002(trainer2) gid=1002(trainer2) groups=1002(trainer2),1004(coaches)
```

### 6. Grading

```bash
$ nusactl grade linlab-challenge-5
```

---

## 🐛 Troubleshooting & Kesalahan

- **Password dengan `!` gagal di shell:** Karakter `!` bisa memicu **history expansion** di bash. Solusi: pakai **kutip tunggal**:
  ```bash
  $ echo 'trainer1:Train!@#123' | sudo chpasswd
  ```

- **`useradd` tanpa `-m` tidak buat home:** Di beberapa distro, `useradd` default **tidak** buat home. Selalu pakai `-m`.

- **`usermod -G` (tanpa `-a`) menghapus group lama:** Kesalahan klasik. Untuk **menambah** group, selalu `usermod -aG`. Kalau sudah kejadian, tambahkan group lama kembali.

- **`chage -M 7` tidak langsung berlaku:** Perubahan berlaku saat user login berikutnya. Cek dengan `chage -l`.

- **`usermod -L` tidak mengunci user yang login aktif:** Lock hanya mencegah **login baru**. User yang sedang login tetap bisa lanjut sampai logout.

- **`chage -d 0` sebelum password di-set:** Bisa bikin field di `/etc/shadow` jadi `0` tapi password masih `!!`. Urutan yang benar: `useradd` → `chpasswd` → `chage -d 0`.

- **User `trainer2` tetap bisa login meski locked:** Cek dengan `passwd -S trainer2`. Kalau status `P` (password set) bukan `L` (locked), lock belum berhasil.

- **`id trainer1` tidak menampilkan `coaches`:** User perlu **logout & login ulang** agar group baru aktif. Untuk sesi saat ini, cek dengan `groups trainer1`.

- **`groupadd coaches` error "already exists":** Pakai `groupadd -f` untuk idempotent:
  ```bash
  $ sudo groupadd -f coaches
  ```

- **`usermod -s /bin/sh` gagal "shell not found":** Pastikan `/bin/sh` ada dan terdaftar di `/etc/shells`:
  ```bash
  $ cat /etc/shells
  ```

- **Verifikasi user di `/etc/passwd` tidak muncul comment:** Cek apakah `usermod -c` berhasil. Kalau tidak, mungkin ada typo di nama user.

- **Log grading gagal di case tertentu:** Cek log:
  ```bash
  $ cat /var/log/nusactl/labs/linlab-challenge-5.log
  ```

- **Password ditolak sistem:** Password `Train!@#123` mungkin tidak lolos **password quality check** (PAM). Cek `/etc/pam.d/common-password` atau `/etc/security/pwquality.conf`.

---

## 💡 Catatan & Insight Pribadi

### `usermod -G` vs `-aG` — Perbedaan Krusial

| Perintah | Efek |
|---|---|
| `usermod -G coaches trainer1` | **Ganti** supplementary group — group lama hilang ❌ |
| `usermod -aG coaches trainer1` | **Tambah** ke group — group lama tetap ✅ |

**Aturan:** Selalu pakai `-aG` untuk menambah. Kalau tidak pakai `-a`, group lama akan hilang (meskipun primary group tidak terpengaruh).

### Lock vs Delete — Kapan Pakai Apa?

| Skenario | Aksi |
|---|---|
| Karyawan resign | **Lock** (preserve data & audit) |
| Karyawan cuti panjang | **Lock** |
| Akun testing selesai | **Delete** |
| Akun duplikat | **Delete** |
| User sementara | **Delete** setelah selesai |

**Rekomendasi:** Untuk user production, **lock** lebih aman — bisa di-unlock kapan saja, dan file yang dimiliki tetap ada.

### `chage -d 0` — Force Password Change

**Cara kerja:**
- Set field "last password change" di `/etc/shadow` ke `0` (epoch = 1 Jan 1970).
- Saat user login, sistem lihat tanggalnya sudah lewat jauh → paksa ganti password.

**Alternatif:** `passwd -e trainer3` — efeknya sama.

**Urutan yang benar:**
1. `useradd trainer3`
2. `passwd trainer3` — set password
3. `chage -d 0 trainer3` — paksa ganti

Kalau `chage -d 0` dijalankan **sebelum** password di-set, field jadi `0` tapi password masih `!!` — sistem bingung.

### `chpasswd` vs `passwd`

| Perintah | Mode | Cocok untuk |
|---|---|---|
| `passwd user` | Interaktif | Set manual |
| `echo "user:pass" \| chpasswd` | Non-interaktif | Script, otomasi |

**`chpasswd`** sangat berguna untuk **provisioning user massal** — bisa loop:
```bash
for user in trainer1 trainer2 trainer3; do
  echo "$user:Train!@#123" | sudo chpasswd
done
```

### Kapan Group Baru Aktif?

- **Saat login berikutnya** (logout-login ulang).
- **Bukan** langsung setelah `usermod -aG`.
- Untuk verifikasi cepat, pakai `groups trainer1` (query langsung ke `/etc/group`).

### Detail `/etc/passwd` — 7 Kolom

```
trainer1:x:1001:1001:Trainer One:/home/trainer1:/bin/bash
    │    │  │    │         │           │              │
    │    │  │    │         │           │              └── shell
    │    │  │    │         │           └── home directory
    │    │  │    │         └── comment (GECOS)
    │    │  │    └── GID (primary group)
    │    │  └── UID
    │    └── password placeholder (x = lihat /etc/shadow)
    └── username
```

### Detail `/etc/shadow` — 9 Kolom

```
trainer1:$6$...:18000:0:7:5:::
    │      │      │   │ │ │ │
    │      │      │   │ │ │ └── expiry date (kosong)
    │      │      │   │ │ └── inactive days
    │      │      │   │ └── warning days
    │      │      │   └── max days (7)
    │      │      └── min days (0)
    │      └── last change (18000 = tanggal)
    └── hash password
```

### Real-World Use Cases

- **Onboarding user baru:** `useradd` → `chpasswd` → `chage -d 0` → `usermod -aG`.
- **Offboarding:** `usermod -L` + `chage -E` (set expiry tanggal kemarin).
- **Security policy:** Max password age 90 hari untuk semua user.
- **Team collaboration:** Buat group, tambah user, set permission directory ke group.

### Kasus Nyata

- **Server production:** User baru di-set `chage -d 0` agar langsung ganti password default.
- **Karyawan resign:** Akun di-lock, tapi file tetap ada untuk audit.
- **Tim DevOps:** Group `devops` untuk akses shared directory `/var/www`.
- **Contractor:** Akun dengan expiry date — otomatis tidak bisa login setelah tanggal.

### Pelajaran Kunci dari Challenge Ini

1. **`useradd -m -s`** = user + home + shell.
2. **`chpasswd`** = set password non-interaktif.
3. **`usermod -c`** = set comment (GECOS).
4. **`chage -M`** = max password age.
5. **`usermod -L`** = lock akun.
6. **`chage -d 0`** = force change password.
7. **`usermod -aG`** = append ke group (jangan `-G`!).
8. **`usermod -s`** = ubah shell.
9. **Verifikasi dengan `id` + `chage -l` + `passwd -S`.**

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `sudo useradd -m -s /bin/bash <user>` | Buat user + home + shell |
| `echo "user:pass" \| sudo chpasswd` | Set password non-interaktif |
| `sudo passwd <user>` | Set password interaktif |
| `sudo usermod -c "Comment" <user>` | Set comment |
| `sudo chage -M 7 <user>` | Max password age 7 hari |
| `sudo chage -d 0 <user>` | Force change password |
| `sudo usermod -L <user>` | Lock user |
| `sudo usermod -U <user>` | Unlock user |
| `sudo passwd -l <user>` | Lock via passwd |
| `sudo passwd -e <user>` | Expire password (force change) |
| `sudo groupadd <group>` | Buat group |
| `sudo usermod -aG <group> <user>` | Append ke group |
| `sudo usermod -s /bin/sh <user>` | Ubah shell |
| `id <user>` | Lihat UID, GID, group |
| `groups <user>` | Daftar group user |
| `sudo chage -l <user>` | Lihat aging info |
| `sudo passwd -S <user>` | Cek status password |

**File penting:**

| File | Isi |
|---|---|
| `/etc/passwd` | Data user |
| `/etc/shadow` | Hash password & aging |
| `/etc/group` | Data group |
| `/etc/gshadow` | Password group |
| `/etc/login.defs` | Default policy user baru |
| `/etc/skel/` | Template home |

**Urutan perintah lengkap:**

```bash
# 1. Buat user dengan shell
sudo useradd -m -s /bin/bash trainer1
sudo useradd -m -s /bin/bash trainer2
sudo useradd -m -s /bin/bash trainer3

# 2. Set password non-interaktif
echo 'trainer1:Train!@#123' | sudo chpasswd
echo 'trainer2:Train!@#123' | sudo chpasswd
echo 'trainer3:Train!@#123' | sudo chpasswd

# 3. Set comment
sudo usermod -c "Trainer One" trainer1
sudo usermod -c "Trainer Two" trainer2
sudo usermod -c "Trainer Three" trainer3

# 4. Password policy
sudo chage -M 7 trainer1       # max 7 hari
sudo usermod -L trainer2       # lock
sudo chage -d 0 trainer3       # force change

# 5. Buat group + tambah user
sudo groupadd coaches
sudo usermod -aG coaches trainer1
sudo usermod -aG coaches trainer2

# 6. Ubah shell trainer3
sudo usermod -s /bin/sh trainer3

# 7. Verifikasi
grep -E "trainer[123]" /etc/passwd
sudo chage -l trainer1
sudo passwd -S trainer2
grep coaches /etc/group
id trainer1
```

---

## 📌 Kesimpulan

Lab Challenge 5 ini menguji kemampuan **manajemen user advanced** — dari membuat user dengan konfigurasi lengkap, sampai mengatur kebijakan password & group. Yang diuji:

- **`useradd`** dengan opsi `-m -s` (home + shell).
- **`chpasswd`** untuk password non-interaktif.
- **`usermod -c`** untuk comment (GECOS).
- **`chage`** untuk password aging policy (max age, force change).
- **`usermod -L`** untuk lock akun.
- **`usermod -aG`** untuk tambah group **tanpa** hapus group lama.
- **`usermod -s`** untuk ubah shell.
- **Verifikasi** dengan `id`, `chage -l`, `passwd -S`.

Kemampuan ini adalah **inti dari sysadmin** — hampir setiap hari berurusan dengan user. Pola yang benar:

1. **Buat user** dengan `useradd -m -s`.
2. **Set password** dengan `chpasswd`.
3. **Set policy** dengan `chage` dan `usermod`.
4. **Tambah ke group** dengan `usermod -aG`.
5. **Verifikasi** dengan `id` + `chage -l` + `passwd -S`.

Yang paling berharga dari challenge ini: **pengalaman langsung** melihat bagaimana `usermod -G` (tanpa `-a`) bisa menghapus group lama — pelajaran berharga yang tidak akan dilupakan.

**Motto sysadmin:** *User management yang rapi = server yang aman.*

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
