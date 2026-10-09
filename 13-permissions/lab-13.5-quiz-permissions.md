# Lab 13.5 — Quiz: File Permissions, Ownership & ACLs

**Course:** Linux System Administration (Adinusa)
**Topic:** Quiz — ownership, permissions, dan ACL
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Mengubah ownership secara **rekursif** dengan `chown -R`
- Mengatur permission ketat untuk owner dan group dengan `chmod`
- Memberikan akses **read-only** ke user spesifik menggunakan ACL
- Mengonfigurasi **default ACL** agar file baru mewarisi aturan akses

---

## 📘 Skenario

Sebagai sysadmin di sebuah creative agency, saya diminta menyelesaikan konfigurasi akses untuk **Design Collaboration Project**.

Semua file dan direktori sudah disiapkan. Tugas saya:

1. Set ownership ke `designer1:creative_team`
2. Set permission `770` pada `/projects` dan `/projects/design_collab`
3. Beri user `reviewer1` akses **read-only** via ACL
4. Set **default ACL** agar file baru otomatis memberi `reviewer1` akses read-only

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start linlab-013-5
# 1. Ubah ownership rekursif
$ sudo chown -R designer1:creative_team /projects/design_collab

# 2. Set permission rekursif: owner & group rwx, others none
$ sudo chmod -R 770 /projects/design_collab

# 3. Set permission /projects juga ke 770 (parent directory)
$ sudo chmod 770 /projects

# 4. ACL read-only untuk reviewer1 (rekursif)
$ sudo setfacl -R -m u:reviewer1:r /projects/design_collab

# 5. Default ACL agar file baru mewarisi akses reviewer1
$ sudo setfacl -d -m u:reviewer1:r /projects/design_collab
