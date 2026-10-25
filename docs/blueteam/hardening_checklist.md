# Hardening Checklist — Target Node (`SRV-WEB-KEL16B`, 192.168.16.5)
Panduan PBL Pertemuan ke-9.

Bukti before/after disimpan di `docs/blueteam/evidence/`.

## A. Access & Authentication

| Kontrol | Status | Bukti |
|---|---|---|
| Nonaktifkan login root langsung via SSH | **Selesai** | `PermitRootLogin no` & `AllowUsers kelompok16-user` diterapkan di `/etc/ssh/sshd_config.d/00-hardening-kel16.conf`; percobaan login `root` & user asing ditolak — lihat [Before](<evidence/kontrol A1 - Menolak login root langsung/sshd-efektif-sebelum.png>), [After](<evidence/kontrol A1 - Menolak login root langsung/sshd-efektif-setelah.png>), [Uji Login Ditolak](<evidence/kontrol A1 - Menolak login root langsung/percobaan-login-root-ditolak.png>) |
| SSH key-based auth (disable password auth) | **Selesai** | Kunci Ed25519 (`kel16-lab`) dipasang di `~/.ssh/authorized_keys` (mode `600`) dan `PasswordAuthentication no` diaktifkan; login via password ditolak dengan `Permission denied (publickey)` — lihat [Before](<evidence/kontrol A2 - Login SSH berbasis kunci/ssh-password-sebelum.png>), [Kunci Terpasang](<evidence/kontrol A2 - Login SSH berbasis kunci/kunci-publik-terpasang.png>), [Uji Password Ditolak](<evidence/kontrol A2 - Login SSH berbasis kunci/percobaan-login-password-ditolak.png>) |
| Ganti port default SSH | **Selesai** | Port SSH tetap menggunakan port `22` agar konsisten dengan skema IP (`docs/Design/ip_plan.md`) dan skenario pengujian Red Team, namun diamankan dengan key-based auth, `ufw limit`, dan `fail2ban` |
| fail2ban / rate-limit percobaan login | **Selesai** | `MaxAuthTries 3`, `MaxStartups 3:50:10`, `LogLevel VERBOSE`, `ufw limit` pada port `22/tcp`, serta jail `fail2ban` (`sshd`, `maxretry = 3`, `bantime = 10m`) aktif memblokir IP setelah 3x gagal — lihat [Before](<evidence/kontrol A3 - Pembatasan percobaan login/rate-limit-sebelum.png>), [After Config](<evidence/kontrol A3 - Pembatasan percobaan login/sshd-maxstartups-dan-rate-limit.png>), [Uji Blokir Fail2ban](<evidence/kontrol A3 - Pembatasan percobaan login/uji-koneksi-paralel-dari-penyerang.png>) |

## B. Service & Attack Surface

| Kontrol | Status | Bukti |
|---|---|---|
| Nonaktifkan service/port yang tidak dipakai | **Selesai** | Service bawaan Ubuntu yang tidak relevan (`ModemManager.service`, `multipathd.service`, `open-iscsi.service`, `udisks2.service`) sudah di-disable/mask; MySQL (`3306`) diikat hanya ke loopback `127.0.0.1` — lihat [Before](<evidence/kontrol B2 - Nonaktifkan service dan isolasi database/service-dan-listener-sebelum.png>), [After](<evidence/kontrol B2 - Nonaktifkan service dan isolasi database/service-dan-listener-setelah.png>), [Uji Isolasi MySQL](<evidence/kontrol B2 - Nonaktifkan service dan isolasi database/uji-akses-mysql-luar-ditolak.png>) |
| Firewall (UFW) whitelist port perlu saja | **Selesai** | Default `deny (incoming)`, rule warisan port `80` dihapus; UFW hanya mengizinkan port `22/tcp` (`LIMIT IN` dari `192.168.16.0/24` & `ALLOW IN` dari `192.168.56.0/24`) dan port `5000/tcp` dari `192.168.16.0/24`. Hasil scan Nmap menunjukkan port terbuka berkurang dari 4 menjadi 2 port (`22` & `5000`) — lihat [UFW Before](<evidence/kontrol B1 - Firewall whitelist dan persisten/ufw-sebelum.png>), [UFW After](<evidence/kontrol B1 - Firewall whitelist dan persisten/ufw-setelah.png>), [Nmap Before](<evidence/kontrol B1 - Firewall whitelist dan persisten/pemindaian-port-sebelum.png>), [Nmap After](<evidence/kontrol B1 - Firewall whitelist dan persisten/pemindaian-port-setelah.png>) |
| Update & patch OS/paket ke versi aman | **Selesai** | 20 paket upgradable pada Ubuntu 24.04.5 LTS diperbarui lewat `apt update && apt upgrade -y` hingga `0 not upgraded` (kernel `6.8.0-147-generic`), serta `unattended-upgrades.service` aktif — lihat [Before](<evidence/kontrol B3 - Update patch OS dan hapus akun default/audit-patch-dan-akun-sebelum.png>), [After](<evidence/kontrol B3 - Update patch OS dan hapus akun default/apt-patch-setelah.png>) |
| Hapus akun/kredensial default | **Selesai** | Hasil audit `mysql.user` memastikan tidak ada user anonim, `root` MySQL hanya via `localhost` (`auth_socket`), tidak ada database `test`, dan akun `root` OS dikunci (`passwd -S root` → `L`) — lihat [Audit Akun MySQL & OS](<evidence/kontrol B3 - Update patch OS dan hapus akun default/audit-akun-mysql-dan-os-setelah.png>) |

## C. Data & Application

| Kontrol | Status | Bukti |
|---|---|---|
| Kredensial tidak hardcoded (env var/secrets) | **Selesai** | Kredensial MySQL (`flaskapp`) dan `FLASK_SECRET_KEY` disimpan di `.env` (dimuat lewat `python-dotenv`, fallback hardcoded di `app.py` dihapus), permission `.env` & `seed_users.py` diperketat menjadi `chmod 600` (ditolak saat dibaca user `nobody`) — lihat [Before](<evidence/kontrol C1 - Kredensial tidak hardcoded dan permission ketat/kredensial-sebelum.png>), [After](<evidence/kontrol C1 - Kredensial tidak hardcoded dan permission ketat/kredensial-setelah.png>) |
| Validasi input pada endpoint API | **Selesai** | Endpoint `POST /api/sensor` sudah memvalidasi skema JSON, format `device_id` (`^bilicheck-[0-9]{2}$`), dan rentang fisiologis bilirubin (`0.0..50.0 mg/dL`) dengan parameterized query — lihat [Uji Validasi API Sensor](<evidence/kontrol C2 - Least privilege database dan validasi input API/validasi-input-api-sensor-setelah.png>). Sementara pada endpoint form login (`POST /`), validasi/parameterized query belum diterapkan dan akan diselesaikan pada tahap mitigasi Fase 2 (lihat `docs/purple-team-log.md`) |
| Least-privilege pada user DB | **Selesai** | User MySQL `flaskapp@localhost` hanya diberi privilege `SELECT, INSERT, UPDATE, DELETE` pada database `iot_replica_db`, tidak punya privilege `CREATE`/`DROP`/administratif (uji `DROP TABLE users` & `SELECT mysql.user` ditolak dengan `ERROR 1142`) — lihat [Before](<evidence/kontrol C2 - Least privilege database dan validasi input API/privilege-db-dan-api-sebelum.png>), [After](<evidence/kontrol C2 - Least privilege database dan validasi input API/least-privilege-mysql-setelah.png>) |
| Logging aktivitas mencurigakan | **Selesai** | Di level aplikasi, `logs/security.log` mencatat percobaan login gagal, indikasi payload SQLi, dan pelanggaran rentang sensor (password disensor `[REDACTED]`) — lihat [Before](<evidence/kontrol C3 - Logging aktivitas dan monitoring jaringan/logging-aplikasi-sebelum.png>), [After](<evidence/kontrol C3 - Logging aktivitas dan monitoring jaringan/log-aktivitas-aplikasi-setelah.png>). Di level jaringan, monitoring via Security Onion (sensor `seconion-eth1`) aktif memantau trafik ke Target Node, alert tervalidasi lewat Sguil — lihat [`docs/blueteam/security_onion_setup.md`](security_onion_setup.md) |

## Lampiran Screenshot Bukti Hardening (Before vs After)

### A. Access & Authentication

**1. Nonaktifkan login root langsung via SSH (`PermitRootLogin no` & `AllowUsers`)**
- **Before:**
  ![A1 Before](<evidence/kontrol A1 - Menolak login root langsung/sshd-efektif-sebelum.png>)
- **After:**
  ![A1 After](<evidence/kontrol A1 - Menolak login root langsung/sshd-efektif-setelah.png>)
- **Hasil percobaan login root gagal setelah hardening:**
  ![A1 Login Root Ditolak](<evidence/kontrol A1 - Menolak login root langsung/percobaan-login-root-ditolak.png>)

**2. SSH key-based auth (`PasswordAuthentication no`)**
- **Before:**
  ![A2 Before](<evidence/kontrol A2 - Login SSH berbasis kunci/ssh-password-sebelum.png>)
- **After (Kunci publik terpasang & uji login key berhasil):**
  ![A2 Kunci Terpasang](<evidence/kontrol A2 - Login SSH berbasis kunci/kunci-publik-terpasang.png>)
- **Hasil percobaan login password gagal setelah hardening:**
  ![A2 Login Password Ditolak](<evidence/kontrol A2 - Login SSH berbasis kunci/percobaan-login-password-ditolak.png>)

**3. fail2ban / rate-limit percobaan login (`MaxAuthTries 3`, `MaxStartups`, `ufw limit`)**
- **Before:**
  ![A3 Before](<evidence/kontrol A3 - Pembatasan percobaan login/rate-limit-sebelum.png>)
- **After (Konfigurasi `sshd`, `fail2ban`, dan `ufw limit`):**
  ![A3 After](<evidence/kontrol A3 - Pembatasan percobaan login/sshd-maxstartups-dan-rate-limit.png>)
- **Hasil uji percobaan login berulang diblokir `fail2ban`:**
  ![A3 Uji Koneksi Berulang](<evidence/kontrol A3 - Pembatasan percobaan login/uji-koneksi-paralel-dari-penyerang.png>)

---

### B. Service & Attack Surface

**1. Firewall (UFW) whitelist port perlu saja & Nmap Scan Before vs After**
- **UFW Before:**
  ![B1 UFW Before](<evidence/kontrol B1 - Firewall whitelist dan persisten/ufw-sebelum.png>)
- **UFW After:**
  ![B1 UFW After](<evidence/kontrol B1 - Firewall whitelist dan persisten/ufw-setelah.png>)
- **Output Nmap Scan Before (4 port terbuka: `22, 80, 3306, 5000`):**
  ![B1 Nmap Before](<evidence/kontrol B1 - Firewall whitelist dan persisten/pemindaian-port-sebelum.png>)
- **Output Nmap Scan After (berkurang menjadi 2 port terbuka: `22, 5000`):**
  ![B1 Nmap After](<evidence/kontrol B1 - Firewall whitelist dan persisten/pemindaian-port-setelah.png>)

**2. Nonaktifkan service tidak dipakai & isolasi MySQL di loopback (`127.0.0.1:3306`)**
- **Before:**
  ![B2 Before](<evidence/kontrol B2 - Nonaktifkan service dan isolasi database/service-dan-listener-sebelum.png>)
- **After:**
  ![B2 After](<evidence/kontrol B2 - Nonaktifkan service dan isolasi database/service-dan-listener-setelah.png>)
- **Uji koneksi MySQL dari luar ditolak vs lokal diterima:**
  ![B2 Uji MySQL](<evidence/kontrol B2 - Nonaktifkan service dan isolasi database/uji-akses-mysql-luar-ditolak.png>)

**3. Update & patch OS/paket ke versi aman + Hapus akun/kredensial default**
- **Before (20 paket upgradable):**
  ![B3 Before](<evidence/kontrol B3 - Update patch OS dan hapus akun default/audit-patch-dan-akun-sebelum.png>)
- **After (`apt upgrade` selesai & `unattended-upgrades` aktif):**
  ![B3 Patch After](<evidence/kontrol B3 - Update patch OS dan hapus akun default/apt-patch-setelah.png>)
- **After (Audit akun MySQL & status lock `root` OS):**
  ![B3 Audit Akun After](<evidence/kontrol B3 - Update patch OS dan hapus akun default/audit-akun-mysql-dan-os-setelah.png>)

---

### C. Data & Application

**1. Kredensial tidak hardcoded (`.env` mode `600` & penghapusan fallback secret di `app.py`)**
- **Before:**
  ![C1 Before](<evidence/kontrol C1 - Kredensial tidak hardcoded dan permission ketat/kredensial-sebelum.png>)
- **After:**
  ![C1 After](<evidence/kontrol C1 - Kredensial tidak hardcoded dan permission ketat/kredensial-setelah.png>)

**2. Least-privilege pada user DB (`flaskapp@localhost`) & Validasi input pada endpoint API (`POST /api/sensor`)**
- **Before (Kode `app.py` awal):**
  ![C2 Before](<evidence/kontrol C2 - Least privilege database dan validasi input API/privilege-db-dan-api-sebelum.png>)
- **After (Least-privilege user DB MySQL — `DROP TABLE` & `mysql.user` ditolak):**
  ![C2 Least Privilege After](<evidence/kontrol C2 - Least privilege database dan validasi input API/least-privilege-mysql-setelah.png>)
- **After (Validasi input endpoint `POST /api/sensor` — HTTP 400 vs HTTP 201):**
  ![C2 Validasi API After](<evidence/kontrol C2 - Least privilege database dan validasi input API/validasi-input-api-sensor-setelah.png>)

**3. Logging aktivitas mencurigakan (`logs/security.log`)**
- **Before:**
  ![C3 Before](<evidence/kontrol C3 - Logging aktivitas dan monitoring jaringan/logging-aplikasi-sebelum.png>)
- **After (`logs/security.log` aktif mencatat gagal login, pola SQLi, dan input anomali):**
  ![C3 After](<evidence/kontrol C3 - Logging aktivitas dan monitoring jaringan/log-aktivitas-aplikasi-setelah.png>)
