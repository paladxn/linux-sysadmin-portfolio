# Lab 13.6 — Quiz: File Permissions & Ownership

**Course:** Linux System Administration (Adinusa)
**Topic:** Quiz — ownership, SGID, immutable, chmod, ACL
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Mengubah ownership (user & group) secara **rekursif** dengan `chown -R`
- Menambahkan **SGID bit** pada file dengan `chmod g+s`
- Menghapus file yang memiliki atribut **immutable** (`chattr -i`)
- Mengatur permission file dengan `chmod` (numeric)
- Memberikan akses tulis spesifik ke user dengan **ACL** (`setfacl`)

---

## 📘 Skenario

Saya diminta menyelesaikan konfigurasi ownership, permission, dan ACL pada direktori `~/lab136`. Beberapa file memiliki atribut khusus yang perlu ditangani.

---

## 🔧 Persiapan Lab

$ nusactl login
$ nusactl start linlab-013-6

# 1. Ubah ownership rekursif
$ sudo chown -R <username>:student ~/lab136/change_me

# 2. Tambah SGID bit pada file perm (tanpa mengubah permission lain)
$ sudo chmod g+s ~/lab136/answer/perm

# 3. Hapus file garbage (hilangkan immutable dulu jika ada)
$ sudo chattr -i ~/lab136/answer/garbage 2>/dev/null
$ sudo rm ~/lab136/answer/garbage

# 4. Set permission permissions.txt ke 750 (rwxr-x---)
$ sudo chmod 750 ~/lab136/answer/permissions.txt

# 5. Tambah ACL write untuk user
$ sudo setfacl -m u:<username>:w ~/lab136/answer/acl_mnop.txt

$ sudo chown -R :student ~/lab136/change_me
# (harusnya: <username>:student)

$ sudo setfacl -m u::w ~/lab136/answer/acl_mnop.txt
# (harusnya: u:<username>:w)
