# Lab 12.4 — Configure SSH Key Authentication

**Course:** Linux System Administration (Adinusa)
**Topic:** SSH key-based authentication & remote command execution
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Membuat **SSH key pair** tanpa passphrase menggunakan `ssh-keygen`
- Menyalin **public key** ke server remote untuk login tanpa password
- Menjalankan **perintah remote** via SSH tanpa membuka shell interaktif
- Memverifikasi bahwa autentikasi berbasis key sudah berfungsi

---

## 📘 Konsep Dasar

| Konsep | Penjelasan |
|---|---|
| **SSH Key Pair** | Pasangan **private key** + **public key** untuk autentikasi |
| **Private Key** | Disimpan di komputer lokal, **tidak boleh dibagikan** |
| **Public Key** | Disimpan di server remote (`~/.ssh/authorized_keys`) |
| **`ssh-keygen`** | Utility untuk membuat key pair |
| **`ssh-copy-id`** | Utility untuk menyalin public key ke server remote |
| **Passwordless Login** | Login SSH tanpa mengetik password, menggunakan key |

**Kenapa key-based authentication lebih baik?**
- Lebih aman dari password (tidak bisa di-brute force).
- Nyaman untuk otomasi (script, cron, CI/CD).
- Bisa ditambahkan passphrase untuk lapisan keamanan ekstra.

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-012-4
```

---

## 🧪 Challenge

### Soal

1. Generate SSH key pair menggunakan `ssh-keygen` **tanpa passphrase**.
2. Kirim public key ke user `btastudent` di `lab.adinusa.id` port `50001`, password `BTABtech`.
3. Jalankan perintah `cat ~/.ssh/authorized_keys` di remote via SSH **tanpa shell interaktif**.
4. Jalankan perintah `hostname` di remote via SSH **tanpa shell interaktif**.

> 💡 **Hint:** Pastikan kamu bisa `ssh` tanpa password.

### ✅ Solusi

#### 1. Generate SSH key pair tanpa passphrase

```bash
$ ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
```

**Penjelasan opsi:**

| Opsi | Arti |
|---|---|
| `-t rsa` | Tipe key: RSA |
| `-b 4096` | Panjang key: 4096 bit (lebih aman) |
| `-f ~/.ssh/id_rsa` | Lokasi file private key |
| `-N ""` | Passphrase kosong (tanpa passphrase) |

**Alternatif interaktif:**

```bash
$ ssh-keygen
```

Lalu tekan **Enter** tiga kali (default path, passphrase kosong, konfirmasi kosong).

#### 2. Salin public key ke server remote

```bash
$ ssh-copy-id -i ~/.ssh/id_rsa.pub -p 50001 btastudent@lab.adinusa.id
```

Saat diminta password, masukkan:

```
BTABtech
```

**Jika `ssh-copy-id` tidak tersedia**, gunakan cara manual:

```bash
$ cat ~/.ssh/id_rsa.pub | ssh -p 50001 btastudent@lab.adinusa.id \
  "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

#### 3. Jalankan perintah remote tanpa shell interaktif

```bash
$ ssh -p 50001 btastudent@lab.adinusa.id "cat ~/.ssh/authorized_keys"
$ ssh -p 50001 btastudent@lab.adinusa.id "hostname"
```

Perintah di atas akan langsung mengeksekusi command dan keluar — tanpa membuka shell interaktif.

#### 4. Verifikasi passwordless login

```bash
$ ssh -p 50001 -o BatchMode=yes -o StrictHostKeyChecking=no btastudent@lab.adinusa.id "hostname"
```

**Penjelasan opsi:**

| Opsi | Arti |
|---|---|
| `BatchMode=yes` | Non-interaktif — gagal jika butuh password |
| `StrictHostKeyChecking=no` | Lewati prompt verifikasi host key (untuk first connect) |

Jika perintah berhasil tanpa diminta password, berarti key-based auth sudah aktif.

---

## 🔍 Verifikasi

### 1. Cek key pair sudah dibuat

```bash
$ ls -l ~/.ssh/id_rsa*
```

Output:

```
-rw------- 1 student student ... id_rsa
-rw-r--r-- 1 student student ... id_rsa.pub
```

### 2. Cek authorized_keys di remote

```bash
$ ssh -p 50001 btastudent@lab.adinusa.id "cat ~/.ssh/authorized_keys"
```

Output akan menampilkan public key yang tadi dikirim.

### 3. Cek hostname remote

```bash
$ ssh -p 50001 btastudent@lab.adinusa.id "hostname"
```

Output: hostname dari `lab.adinusa.id`.

### 4. Pastikan tidak ada prompt password

Jalankan perintah remote dengan `BatchMode=yes`:

```bash
$ ssh -p 50001 -o BatchMode=yes btastudent@lab.adinusa.id "hostname"
```

Jika berhasil, artinya autentikasi key sudah bekerja.

---

## 🐛 Troubleshooting & Kesalahan

- **`ssh-copy-id: command not found`:** Install paket `openssh-client`:
  ```bash
  $ sudo apt install openssh-client -y
  ```
  Atau gunakan cara manual seperti di atas.

- **`Permission denied (publickey)`:** Beberapa kemungkinan:
  - Public key belum tersalin ke `~/.ssh/authorized_keys`.
  - Permission salah di server:
    ```bash
    $ chmod 700 ~/.ssh
    $ chmod 600 ~/.ssh/authorized_keys
    ```
  - Home directory server **group-writable** — SSH akan menolak. Cek dengan `ls -ld ~`.

- **`Host key verification failed`:** Pertama kali connect, SSH meminta konfirmasi host key. Gunakan:
  ```bash
  $ ssh -o StrictHostKeyChecking=no -p 50001 btastudent@lab.adinusa.id "hostname"
  ```
  Atau tambahkan host key secara manual:
  ```bash
  $ ssh-keyscan -p 50001 lab.adinusa.id >> ~/.ssh/known_hosts
  ```

- **Masih diminta password setelah `ssh-copy-id`:** Cek apakah SSH server mengizinkan `PubkeyAuthentication yes` di `/etc/ssh/sshd_config`. Kalau tidak, hubungi admin server.

- **Key pair dibuat dengan passphrase:** Karena challenge meminta tanpa passphrase, gunakan `-N ""` saat `ssh-keygen`. Kalau sudah terlanjur, buat ulang:
  ```bash
  $ ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_new -N ""
  $ ssh-copy-id -i ~/.ssh/id_rsa_new.pub -p 50001 btastudent@lab.adinusa.id
  ```

- **`ssh` menggunakan key yang salah:** Jika punya banyak key, tentukan secara eksplisit:
  ```bash
  $ ssh -i ~/.ssh/id_rsa -p 50001 btastudent@lab.adinusa.id "hostname"
  ```

- **Port salah:** Port default SSH adalah 22. Di lab ini digunakan port **50001**, jadi wajib pakai `-p 50001`.

- **`ssh-copy-id` gagal karena host key berubah:** Hapus entry lama di `~/.ssh/known_hosts`:
  ```bash
  $ ssh-keygen -R "[lab.adinusa.id]:50001"
  ```

---

## 💡 Catatan & Insight Pribadi

### Kenapa Key-Based Auth Lebih Aman?

| Aspek | Password | Key-Based |
|---|---|---|
| Brute force | Rentan | Praktis tidak mungkin |
| Penyadapan | Bisa dicuri | Hanya public key yang lewat jaringan |
| Otomasi | Sulit | Mudah |
| Passphrase | Selalu | Opsional |

### Struktur File SSH

| File | Permission | Fungsi |
|---|---|---|
| `~/.ssh/id_rsa` | `600` | Private key |
| `~/.ssh/id_rsa.pub` | `644` | Public key |
| `~/.ssh/authorized_keys` | `600` | Public key yang diizinkan login |
| `~/.ssh/known_hosts` | `644` | Host key server yang pernah diakses |
| `~/.ssh/config` | `600` | Konfigurasi SSH per host |

### Tips Menjalankan Perintah Remote

```bash
# Perintah tunggal
$ ssh user@host "command"

# Beberapa perintah
$ ssh user@host "command1 && command2"

# Dengan sudo
$ ssh user@host "sudo command"

# Simpan output ke file lokal
$ ssh user@host "command" > local.txt

# Copy file dari remote
$ scp -P 50001 user@host:/remote/file .

# Copy file ke remote
$ scp -P 50001 local.txt user@host:/remote/path
```

**Catatan:** Untuk `scp`, opsi port adalah `-P` (huruf besar), bukan `-p`.

### Best Practice SSH Key

- Gunakan **passphrase** untuk key yang dipakai manual.
- Gunakan **key tanpa passphrase** hanya untuk otomasi (dengan izin terbatas).
- Jangan pernah bagikan **private key**.
- Backup key pair di tempat aman.
- Gunakan `ssh-agent` untuk menghindari ketik passphrase berulang.
- Rotasi key secara berkala.

### Kasus Nyata

- **CI/CD:** Runner menggunakan SSH key untuk deploy ke server.
- **Backup otomatis:** Script `rsync` via SSH key tanpa password.
- **Ansible:** Menggunakan SSH key untuk mengelola banyak server.
- **Git:** GitHub/GitLab menggunakan SSH key untuk push/pull tanpa password.

### Pelajaran Kunci

1. **`ssh-keygen -N ""`** → key pair tanpa passphrase.
2. **`ssh-copy-id -p PORT`** → salin public key ke server.
3. **`ssh -p PORT user@host "command"`** → jalankan perintah remote.
4. **`BatchMode=yes`** → verifikasi passwordless login.
5. **Permission file SSH** harus ketat — `600` untuk key dan `authorized_keys`.

---

## 🧠 Perintah yang Dikuasai di Lab Ini

| Perintah | Fungsi |
|---|---|
| `ssh-keygen -t rsa -b 4096 -N ""` | Generate SSH key pair tanpa passphrase |
| `ssh-copy-id -i key.pub -p PORT user@host` | Salin public key ke server |
| `ssh -p PORT user@host "command"` | Jalankan perintah remote |
| `ssh -o BatchMode=yes ...` | Verifikasi passwordless |
| `ssh -i keyfile ...` | Gunakan key spesifik |
| `ssh-keyscan -p PORT host` | Ambil host key server |
| `ssh-keygen -R "[host]:PORT"` | Hapus host key lama |
| `scp -P PORT file user@host:/path` | Copy file via SSH |

---

## 📌 Kesimpulan

Lab ini memberikan keterampilan praktis tentang **SSH key-based authentication** — salah satu mekanisme paling penting untuk mengakses server remote secara aman dan efisien. Dengan key pair, kita bisa login tanpa password, menjalankan perintah remote untuk otomasi, dan mengintegrasikan SSH ke dalam script atau CI/CD pipeline.

Yang paling berharga dari lab ini adalah pemahaman bahwa **private key tidak pernah dikirim ke server** — hanya public key yang disalin. Server menggunakan public key untuk memverifikasi bahwa kita benar-benar pemilik private key. Prinsip ini yang membuat SSH key jauh lebih aman dari password.

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan di repositori ini.
