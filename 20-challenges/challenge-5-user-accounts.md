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

### ✅ Solusi (Urutan yang Benar)

```bash
# 1. Buat user dengan shell bash
$ sudo useradd -m -s /bin/bash trainer1
$ sudo useradd -m -s /bin/bash trainer2
$ sudo useradd -m -s /bin/bash trainer3

# 2. Set password DULU — pakai KUTIP TUNGGAL karena ada karakter '!'
$ echo 'trainer1:Train!@#123' | sudo chpasswd
$ echo 'trainer2:Train!@#123' | sudo chpasswd
$ echo 'trainer3:Train!@#123' | sudo chpasswd

# 3. Comment untuk tiap user
$ sudo usermod -c "Trainer One" trainer1
$ sudo usermod -c "Trainer Two" trainer2
$ sudo usermod -c "Trainer Three" trainer3

# 4. Max password age trainer1: 7 hari
$ sudo chage -M 7 trainer1

# 5. Lock akun trainer2 (SETELAH password di-set)
$ sudo usermod -L trainer2

# 6. Force change password trainer3 (SETELAH password di-set)
$ sudo chage -d 0 trainer3

# 7. Buat group coaches
$ sudo groupadd coaches

# 8. Tambahkan trainer1 & trainer2 ke coaches (append)
$ sudo usermod -aG coaches trainer1
$ sudo usermod -aG coaches trainer2

# 9. Ubah shell trainer3 ke /bin/sh
$ sudo usermod -s /bin/sh trainer3
```

### ⚠️ Urutan Sangat Penting

```
1. useradd         → buat user
2. chpasswd        → set password DULU ←
3. usermod -c      → comment
4. chage -M        → policy
5. usermod -L      → LOCK SETELAH password
6. chage -d 0      → FORCE CHANGE SETELAH password
7. groupadd + -aG  → group
8. usermod -s      → shell
```

**Kenapa?** `chpasswd` **menghapus** flag `!` (locked) di `/etc/shadow` dan **menimpa** tanggal "last change". Kalau `usermod -L` atau `chage -d 0` dijalankan **sebelum** `chpasswd`, konfigurasinya akan **hilang**.

### 🔍 Penjelasan Opsi Penting

**`useradd -m -s /bin/bash`**
- `-m` = buat home directory (`/home/trainerX`).
- `-s /bin/bash` = set default shell.

**`echo 'user:pass' | chpasswd`**
- Non-interaktif — set password dalam satu perintah.
- **WAJIB** pakai kutip tunggal (`'...'`) kalau password mengandung `!`.

**`usermod -c "Comment"`**
- Set **GECOS field** (comment) di `/etc/passwd`.

**`chage -M 7 trainer1`**
- Max 7 hari password valid.

**`usermod -L trainer2`**
- Lock akun — tambah `!` di depan hash password di `/etc/shadow`.
- Unlock: `usermod -U trainer2`.

**`chage -d 0 trainer3`**
- Set "last password change" ke epoch 0 → paksa ganti password saat login.

**`usermod -aG coaches`**
- `-aG` = **append** + supplementary group.
- **WAJIB pakai `-a`**. Kalau hanya `-G`, group lama akan **diganti**.

**`usermod -s /bin/sh trainer3`**
- Ganti default shell ke `/bin/sh` (POSIX shell, biasanya symlink ke `dash`).

---

## 🔍 Verifikasi

### 1. Cek user di `/etc/passwd`

```bash
$ grep -E "trainer[123]" /etc/passwd
```

Output yang diharapkan:
```
trainer1:x:1001:1001:Trainer One:/home/trainer1:/bin/bash
trainer2:x:1002:1002:Trainer Two:/home/trainer2:/bin/bash
trainer3:x:1003:1003:Trainer Three:/home/trainer3:/bin/sh
```

### 2. Cek status password & lock

```bash
$ sudo passwd -S trainer1
$ sudo passwd -S trainer2
$ sudo passwd -S trainer3
```

Output yang diharapkan:

| User | Status | Tanggal | Keterangan |
|---|---|---|---|
| trainer1 | `P` | 2026-10-12 | Password set (normal) ✅ |
| trainer2 | `L` | 2026-10-12 | **Locked** ✅ |
| trainer3 | `P` | **1970-01-01** | **Force change** ✅ |

### 3. Cek password aging trainer1

```bash
$ sudo chage -l trainer1 | grep -E "Maximum|Minimum"
Maximum number of days between password change          : 7
```

### 4. Cek group coaches

```bash
$ grep coaches /etc/group
coaches:x:1004:trainer1,trainer2
```

### 5. Cek group membership

```bash
$ id trainer1
uid=1001(trainer1) gid=1001(trainer1) groups=1001(trainer1),1004(coaches)

$ id trainer2
uid=1002(trainer2) gid=1002(trainer2) groups=1002(trainer2),1004(coaches)
```

Perhatikan: primary group (gid) **tetap** `trainerX`, `coaches` jadi **supplementary**.

### 6. Grading

```bash
$ nusactl grade linlab-challenge-5
```

---

## 🐛 Troubleshooting & Kesalahan

### Masalah Nyata yang Dialami

- **`-bash: !@#123: event not found`:**

  **Penyebab:** Karakter `!` di dalam **double quote** (`"..."`) memicu **bash history expansion**.

  **Contoh error:**
  ```bash
  $ echo "trainer1:Train!@#123" | sudo chpasswd
  -bash: !@#123: event not found
  ```

  **Solusi:** Pakai **kutip tunggal** (`'...'`):
  ```bash
  $ echo 'trainer1:Train!@#123' | sudo chpasswd
  ```

- **Setelah fix password, `trainer2` status berubah dari `L` ke `P`:**

  **Penyebab:** `chpasswd` **menghapus** flag `!` (locked) di `/etc/shadow` saat set password baru. Lock dari `usermod -L` yang dijalankan **sebelum** `chpasswd` akan **hilang**.

  **Gejala:** `passwd -S trainer2` awalnya `L`, setelah `chpasswd` jadi `P`. Case 6 gagal.

  **Solusi:** Jalankan `usermod -L trainer2` **SETELAH** password di-set.

- **`trainer3` tidak lagi diminta ganti password:**

  **Penyebab:** Sama — `chpasswd` **menimpa** field "last password change" dengan tanggal hari ini, sehingga flag *force change* dari `chage -d 0` hilang.

  **Gejala:** `passwd -S trainer3` menunjukkan tanggal hari ini, bukan `1970-01-01`.

  **Solusi:** Jalankan `chage -d 0 trainer3` **SETELAH** password di-set.

- **Ringkasan urutan yang benar:**

  ```bash
  useradd → chpasswd → usermod -L → chage -d 0
  ```
  Bukan:
  ```bash
  useradd → usermod -L → chage -d 0 → chpasswd  ← SALAH
  ```

### Kesalahan Umum Lainnya

- **`useradd` tanpa `-m` tidak buat home:** Di beberapa distro, `useradd` default tidak buat home. Selalu pakai `-m`.

- **`usermod -G` (tanpa `-a`) menghapus group lama:** Selalu pakai `usermod -aG`.

- **`usermod -L` tidak mengunci user yang login aktif:** Lock hanya mencegah **login baru**. User yang sedang login tetap bisa lanjut sampai logout.

- **`chage -d 0` sebelum password di-set:** Bisa bikin field di `/etc/shadow` jadi `0` tapi password masih `!!`. Urutan: `chpasswd` dulu, baru `chage -d 0`.

- **`groupadd coaches` error "already exists":** Pakai `groupadd -f` untuk idempotent.

- **`id trainer1` tidak menampilkan `coaches`:** User perlu logout & login ulang. Untuk sesi saat ini, cek dengan `groups trainer1`.

- **Password ditolak sistem:** Cek `/etc/pam.d/common-password` atau `/etc/security/pwquality.conf`.

- **Log grading gagal di case tertentu:**
  ```bash
  $ cat /var/log/nusactl/labs/linlab-challenge-5.log
  ```

---

## 💡 Catatan & Insight Pribadi

### `usermod -G` vs `-aG` — Perbedaan Krusial

| Perintah | Efek |
|---|---|
| `usermod -G coaches trainer1` | **Ganti** supplementary group — group lama hilang ❌ |
| `usermod -aG coaches trainer1` | **Tambah** ke group — group lama tetap ✅ |

**Aturan:** Selalu pakai `-aG` untuk menambah. Kalau tidak pakai `-a`, group lama akan hilang (meskipun primary group tidak terpengaruh).

### `chpasswd` — Efek Samping yang Sering Dilupakan

`chpasswd` mengubah beberapa field di `/etc/shadow`:

1. Hash password baru.
2. Field "last password change" ke **hari ini**.
3. **Menghapus** flag `!` (locked) di depan hash jika ada.

**Konsekuensi:**
- `usermod -L` yang dijalankan **sebelum** `chpasswd` → **hilang**.
- `chage -d 0` yang dijalankan **sebelum** `chpasswd` → **tertimpa** tanggal baru.

**Solusi:** Set password **DULU**, baru apply policy:
```
useradd → chpasswd → usermod -L → chage -d 0
```

### History Expansion di Bash

Karakter `!` di double quote memicu **history expansion** — bash mencoba menggantinya dengan perintah sebelumnya.

**Contoh:**
```bash
$ echo "test!@#123"
-bash: !@#123: event not found
```

**Tiga solusi:**
1. **Single quote:** `echo 'test!@#123'` ✅ (paling mudah)
2. **Escape:** `echo "test\!@#123"` ✅
3. **Disable history expansion:** `set +H` (permanen: tambah di `.bashrc`)

**Rule of thumb:** Kalau string mengandung karakter khusus (`!`, `$`, `` ` ``, `\`), **selalu pakai single quote**.

### Lock vs Delete — Kapan Pakai Apa?

| Skenario | Aksi |
|---|---|
| Karyawan resign | **Lock** (preserve data & audit) |
| Karyawan cuti panjang | **Lock** |
| Akun testing selesai | **Delete** |
| Akun duplikat | **Delete** |

**Rekomendasi:** Untuk user production, **lock** lebih aman — bisa di-unlock kapan saja, dan file yang dimiliki tetap ada.

### Status di `passwd -S`

| Kode | Arti |
|---|---|
| `P` | Password set — akun aktif |
| `L` | **Locked** — tidak bisa login |
| `NP` | No Password — tidak butuh password (bahaya) |

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
    └── hash password (kalau diawali `!` = locked)
```

### Real-World Use Cases

- **Onboarding user baru:** `useradd` → `chpasswd` → `chage -d 0` → `usermod -aG`.
- **Offboarding:** `usermod -L` + `chage -E` (set expiry tanggal kemarin).
- **Security policy:** Max password age 90 hari untuk semua user.
- **Team collaboration:** Buat group, tambah user, set permission directory ke group.

### Pelajaran Kunci dari Challenge Ini

1. **`useradd -m -s`** = user + home + shell.
2. **`chpasswd`** = set password non-interaktif.
3. **Single quote** wajib kalau password ada `!`.
4. **`usermod -c`** = set comment (GECOS).
5. **`chage -M`** = max password age.
6. **`usermod -L`** = lock akun.
7. **`chage -d 0`** = force change password.
8. **`usermod -aG`** = append ke group (jangan `-G`!).
9. **`chpasswd` menghapus lock & force change** — urutan penting!
10. **Verifikasi dengan `id` + `chage -l` + `passwd -S`.**

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `sudo useradd -m -s /bin/bash <user>` | Buat user + home + shell |
| `echo 'user:pass' \| sudo chpasswd` | Set password non-interaktif |
| `sudo passwd <user>` | Set password interaktif |
| `sudo usermod -c "Comment" <user>` | Set comment |
| `sudo chage -M 7 <user>` | Max password age 7 hari |
| `sudo chage -d 0 <user>` | Force change password |
| `sudo usermod -L <user>` | Lock user |
| `sudo usermod -U <user>` | Unlock user |
| `sudo passwd -S <user>` | Cek status password |
| `sudo groupadd -f <group>` | Buat group (idempotent) |
| `sudo usermod -aG <group> <user>` | Append ke group |
| `sudo usermod -s /bin/sh <user>` | Ubah shell |
| `id <user>` | Lihat UID, GID, group |
| `groups <user>` | Daftar group user |
| `sudo chage -l <user>` | Lihat aging info |

**File penting:**

| File | Isi |
|---|---|
| `/etc/passwd` | Data user |
| `/etc/shadow` | Hash password & aging |
| `/etc/group` | Data group |
| `/etc/gshadow` | Password group |
| `/etc/login.defs` | Default policy user baru |
| `/etc/skel/` | Template home |

**Urutan perintah lengkap (final):**

```bash
# 1. Buat user dengan shell
sudo useradd -m -s /bin/bash trainer1
sudo useradd -m -s /bin/bash trainer2
sudo useradd -m -s /bin/bash trainer3

# 2. Set password DULU — pakai single quote
echo 'trainer1:Train!@#123' | sudo chpasswd
echo 'trainer2:Train!@#123' | sudo chpasswd
echo 'trainer3:Train!@#123' | sudo chpasswd

# 3. Comment
sudo usermod -c "Trainer One" trainer1
sudo usermod -c "Trainer Two" trainer2
sudo usermod -c "Trainer Three" trainer3

# 4. Policy max age
sudo chage -M 7 trainer1

# 5. Lock & force change SETELAH password
sudo usermod -L trainer2
sudo chage -d 0 trainer3

# 6. Group
sudo groupadd coaches
sudo usermod -aG coaches trainer1
sudo usermod -aG coaches trainer2

# 7. Shell
sudo usermod -s /bin/sh trainer3

# 8. Verifikasi
grep -E "trainer[123]" /etc/passwd
sudo passwd -S trainer2         # harus 'L'
sudo passwd -S trainer3         # harus tanggal 1970-01-01
sudo chage -l trainer1          # max harus 7
grep coaches /etc/group
id trainer1
```

**Verifikasi cepat:**

```bash
# Status semua user
sudo passwd -S trainer1
sudo passwd -S trainer2
sudo passwd -S trainer3

# Output yang diharapkan:
# trainer1 P 2026-10-12 7 7 5 -1       ← normal
# trainer2 L 2026-10-12 7 30 5 -1      ← locked
# trainer3 P 1970-01-01 7 30 5 -1      ← force change
```

---

## 📌 Kesimpulan

Lab Challenge 5 ini menguji kemampuan **manajemen user advanced** — dari membuat user dengan konfigurasi lengkap, sampai mengatur kebijakan password & group. Yang diuji:

- **`useradd`** dengan opsi `-m -s` (home + shell).
- **`chpasswd`** untuk password non-interaktif (dengan kutip tunggal untuk `!`).
- **`usermod -c`** untuk comment (GECOS).
- **`chage`** untuk password aging policy (max age, force change).
- **`usermod -L`** untuk lock akun.
- **`usermod -aG`** untuk tambah group **tanpa** hapus group lama.
- **`usermod -s`** untuk ubah shell.
- **Verifikasi** dengan `id`, `chage -l`, `passwd -S`.

Kemampuan ini adalah **inti dari sysadmin** — hampir setiap hari berurusan dengan user. Yang paling berharga dari challenge ini:

1. **Pelajaran `!` di bash** — single quote wajib untuk password dengan karakter khusus.
2. **Pelajaran urutan `chpasswd`** — set password DULU, baru lock & force change. Kalau dibalik, konfigurasi akan hilang.
3. **Pelajaran `usermod -aG`** — jangan `-G` tanpa `-a`.

Ketiga pelajaran ini **sangat berharga** untuk otomasi user provisioning di production — urutan perintah sangat penting, dan detail kecil seperti tanda kutip bisa bikin gagal.

**Motto sysadmin:** *User management yang rapi = server yang aman.*

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
