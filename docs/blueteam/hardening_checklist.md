# Hardening Checklist — Target Node (`SRV-WEB-KEL16B`, 192.168.16.5)
Sesuai Bagian VIII, Panduan PBL Pertemuan ke-9. Minimal 8 kontrol, tersebar di 3 kategori (min. 2/kategori).

Status: **Selesai** / **Sebagian** / **Belum**. Bukti before/after disimpan di `docs/blueteam/evidence/`.

## A. Access & Authentication

| Kontrol | Status | Bukti |
|---|---|---|
| Nonaktifkan login root langsung via SSH | Belum | — |
| SSH key-based auth (disable password auth) | Belum | — |
| Ganti port default SSH | Belum | — |
| fail2ban / rate-limit percobaan login | Belum | — |
| MFA pada dashboard monitoring | Belum (tidak diterapkan — dashboard Flask belum ada fitur MFA) | — |

## B. Service & Attack Surface

| Kontrol | Status | Bukti |
|---|---|---|
| Nonaktifkan service/port yang tidak dipakai | Sebagian | Perlu audit ulang service default Ubuntu |
| Firewall (UFW) whitelist port perlu saja | Sebagian | UFW allow port 5000 hanya dari `192.168.16.0/24` sudah aktif; perlu dicek rule untuk port 22 & 3306 (3306 seharusnya tidak ter-expose sama sekali ke luar `localhost`) |
| Update & patch OS/paket ke versi aman | Sebagian | `apt update && apt upgrade` sudah dijalankan saat setup awal, perlu dijadwalkan ulang mendekati Minggu ke-9/11 |
| Hapus akun/kredensial default | Belum | Perlu audit akun default MySQL & OS |

## C. Data & Application

| Kontrol | Status | Bukti |
|---|---|---|
| Kredensial tidak hardcoded (env var/secrets) | **Selesai** | Kredensial MySQL (`flaskapp`) disimpan di `.env`, dimuat lewat `python-dotenv`, tidak ada di `app.py` |
| Validasi input pada endpoint API | **Belum (sengaja)** | Endpoint `/login` saat ini SENGAJA rentan SQL Injection (versi "before") sebagai bahan skenario Red Team — lihat `docs/redteam/attack_plan.md`. Validasi/parameterized query akan diterapkan di tahap mitigasi (Purple Team Loop Fase 2, lihat `docs/purple-team-log.md`) |
| Least-privilege pada user DB | **Selesai** | User MySQL `flaskapp` hanya diberi privilege `SELECT, INSERT, UPDATE, DELETE` pada database `iot_replica_db`, tidak punya privilege `CREATE`/`DROP`/administratif |
| Logging aktivitas mencurigakan | **Selesai** | Monitoring via Security Onion (sensor `seconion-eth1`) aktif memantau trafik ke Target Node, alert tervalidasi lewat Sguil — lihat `docs/blueteam/security_onion_setup.md` |

## Ringkasan

- **Selesai:** 3 kontrol
- **Sebagian:** 3 kontrol
- **Belum:** 5 kontrol (termasuk 1 kontrol yang sengaja dibiarkan rentan sebagai bagian skenario serangan)

**Catatan penting:** password hash user aplikasi (`users.password_hash`) sudah menggunakan bcrypt (bukan plaintext) sejak awal — bukan bagian checklist wajib di atas, tapi dicatat sebagai praktik baik tambahan.

Checklist ini akan diperbarui seiring progres menuju Minggu ke-11 (Vulnerability Assessment Report).
