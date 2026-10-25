# Hardening Checklist — Target Node (`SRV-WEB-KEL16B`, 192.168.16.5)
Sesuai Bagian VIII, Panduan PBL Pertemuan ke-9. Minimal 8 kontrol, tersebar di 3 kategori (min. 2/kategori).

Status: **Selesai** / **Sebagian** / **Belum**. Bukti before/after disimpan di `docs/blueteam/evidence/` — lihat `evidence/README.md` untuk panduan penamaan file.

> Kolom "Bukti" di bawah berisi link gambar otomatis. Selama file screenshot sudah ditaruh di `docs/blueteam/evidence/` dengan nama yang persis, gambarnya akan langsung tampil saat halaman ini dibuka di GitHub — tidak perlu edit file ini lagi setelah foto di-upload.

## A. Access & Authentication

| Kontrol | Status | Bukti Before | Bukti After |
|---|---|---|---|
| Nonaktifkan login root langsung via SSH | Belum | ![before](evidence/a1-rootlogin-before.png) | ![after](evidence/a1-rootlogin-after.png) |
| SSH key-based auth (disable password auth) | Belum | ![before](evidence/a2-passwordauth-before.png) | ![after](evidence/a2-passwordauth-after.png) |
| Ganti port default SSH | Belum | ![before](evidence/a3-sshport-before.png) | ![after](evidence/a3-sshport-after.png) |
| fail2ban / rate-limit percobaan login | Belum | ![before](evidence/a4-fail2ban-before.png) | ![after](evidence/a4-fail2ban-after.png) |
| MFA pada dashboard monitoring | Belum (tidak diterapkan — dashboard Flask belum ada fitur MFA) | — | — |

## B. Service & Attack Surface

| Kontrol | Status | Bukti Before | Bukti After |
|---|---|---|---|
| Nonaktifkan service/port yang tidak dipakai | Belum | ![before](evidence/b1-services-before.png) | ![after](evidence/b1-services-after.png) |
| Firewall (UFW) whitelist port perlu saja + MySQL tidak ter-expose | Sebagian — UFW allow port 5000 hanya dari `192.168.16.0/24` sudah aktif; bind-address MySQL & status UFW lengkap perlu diverifikasi ulang | ![bind-address before](evidence/b2-mysql-bind-before.png) ![ufw status](evidence/b2-ufw-status.png) | ![scan after](evidence/b2-mysql-port-after.png) |
| Update & patch OS/paket ke versi aman | Belum | ![before](evidence/b3-update-before.png) | ![after](evidence/b3-update-after.png) |
| Hapus akun/kredensial default | Belum | ![before](evidence/b4-mysql-users-before.png) | ![after](evidence/b4-mysql-users-after.png) |

## C. Data & Application

| Kontrol | Status | Bukti |
|---|---|---|
| Kredensial tidak hardcoded (env var/secrets) | **Selesai** | Kredensial MySQL (`flaskapp`) disimpan di `.env`, dimuat lewat `python-dotenv`, tidak ada di `app.py` |
| Validasi input pada endpoint API | **Belum (sengaja)** | Endpoint `/login` saat ini SENGAJA rentan SQL Injection (versi "before") sebagai bahan skenario Red Team — lihat `docs/redteam/attack_plan.md`. Validasi/parameterized query akan diterapkan di tahap mitigasi (Purple Team Loop Fase 2, lihat `docs/purple-team-log.md`) |
| Least-privilege pada user DB | **Selesai** | User MySQL `flaskapp` hanya diberi privilege `SELECT, INSERT, UPDATE, DELETE` pada database `iot_replica_db`, tidak punya privilege `CREATE`/`DROP`/administratif |
| Logging aktivitas mencurigakan | **Selesai** | Monitoring via Security Onion (sensor `seconion-eth1`) aktif memantau trafik ke Target Node, alert tervalidasi lewat Sguil — lihat `docs/blueteam/security_onion_setup.md` |

## Ringkasan

- **Selesai:** 3 kontrol
- **Sebagian:** 1 kontrol
- **Belum:** 7 kontrol (termasuk 1 kontrol yang sengaja dibiarkan rentan sebagai bagian skenario serangan)

**Catatan penting:** password hash user aplikasi (`users.password_hash`) sudah menggunakan bcrypt (bukan plaintext) sejak awal — bukan bagian checklist wajib di atas, tapi dicatat sebagai praktik baik tambahan.

**TODO sebelum presentasi Minggu ke-9:** jalankan 8 kontrol Kategori A & B sesuai panduan command, taruh screenshot di `docs/blueteam/evidence/` sesuai nama file di `evidence/README.md`, lalu ubah status di atas jadi **Selesai**.

Checklist ini akan diperbarui seiring progres menuju Minggu ke-11 (Vulnerability Assessment Report).
