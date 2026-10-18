# Logbook Proyek PBL - Kelompok 16

**Mata Kuliah:** TEK1314 Keamanan Siber

**Skenario:** Simulasi serangan terhadap web server (target: aplikasi DVWA) dalam jaringan tertutup, dengan Kali Linux sebagai attacker dan Security Onion sebagai node monitoring.

---

## Minggu 3 — Perancangan Awal
Tim menyusun topologi jaringan yang terdiri dari tiga node utama: attacker, target, dan monitoring. Alokasi IP ditetapkan menggunakan subnet `192.168.16.0/24`. Untuk target server, disepakati penggunaan Ubuntu Server dengan DVWA sebagai aplikasi web yang sengaja dibuat rentan.

## Minggu 5 — Setup Monitoring
Security Onion di-install dan dikonfigurasi pada node monitoring (`192.168.16.200`). Setelah instalasi selesai, dilakukan pengujian sederhana dengan mengirim ping dari attacker (`192.168.16.100`) menuju target (`192.168.16.5`) untuk memastikan trafik ICMP terekam dengan benar di Sguil/Squert.

## Minggu 6 — Hardening Target Server
Fokus minggu ini ada di pengamanan target server:
- Firewall UFW diaktifkan dengan kebijakan default menolak koneksi masuk; hanya port 22 (SSH, akses dibatasi) dan port 80 (HTTP untuk web app) yang dibuka.
- Beberapa service bawaan yang tidak dibutuhkan skenario dinonaktifkan.
- Dibuat akun non-root untuk keperluan administrasi sehari-hari, sehingga login root langsung tidak digunakan.
- Sistem diperbarui melalui `apt update && apt upgrade`.
- Hostname target diset menjadi `SRV-WEB-KEL16B` mengikuti konvensi penamaan kelompok.

## Minggu 7 — Finalisasi & Persiapan Demo
Baseline report dirampungkan di `docs/phase-1-baseline/baseline-report.md`, mencakup dokumentasi topologi, hasil hardening, dan bukti verifikasi logging. Tim juga menyusun alur presentasi demo: review topologi, walkthrough konfigurasi hardening, serta demonstrasi logging secara langsung.

## Minggu 9 — Perubahan Skenario & Checkpoint

**Perubahan ketentuan dari dosen:** Target Node WAJIB mengikuti tech stack asli proyek IoT kelompok (bukan skenario Metasploitable/DVWA generik seperti sebelumnya). Setelah berkoordinasi dengan tim proyek IoT (**Bilicheck** — realtime bilirubin checker untuk deteksi jaundice bayi), tech stack asli: **Firebase** (backend & database). Karena anggota tim lain di proyek IoT menggunakan Ubuntu Server + MySQL, Target Node dibangun sebagai **Ubuntu Server 24.04.5 LTS + Flask (Python) + MySQL 8.0.46** — detail lengkap di `docs/presentation-minggu9/tech_stack_declaration.md`.

**Rombak ulang Target Node:**
- Server dibersihkan total dari konfigurasi DVWA/Metasploitable sebelumnya.
- Install Python3, venv, Flask, MySQL Server 8.0.46; `mysql_secure_installation` dijalankan.
- Dibuat database `iot_replica_db`, user MySQL `flaskapp` dengan privilege least-privilege (tanpa `CREATE`/admin).
- Dibuat aplikasi Flask skeleton (`app.py`) dengan endpoint `/login` — sengaja mengandung celah SQL Injection versi "before" (string formatting langsung ke query, bukan parameterized), sebagai bahan skenario Red Team. Password user di-hash bcrypt, kredensial DB disimpan di `.env` (tidak hardcode).
- Aplikasi dijadikan systemd service (`iot-replica.service`) agar berjalan permanen.
- Detail checklist hardening (status per kontrol) ada di `docs/blueteam/hardening_checklist.md`.

**Reset & setup ulang Security Onion:** VM Security Onion di-reset total (import ulang `.ova`) karena troubleshooting manual sensor terlalu kompleks di percobaan pertama. Setup ulang mengikuti wizard resmi (`sosetup`, Evaluation Mode), dengan `eth1` sebagai sniffing interface (bukan `eth0` seperti instalasi awal). Berhasil diverifikasi: Sguil menampilkan alert real-time dari trafik Kali → Target Node (ICMP & port scan Nmap). Detail lengkap proses & troubleshooting (gagal download rules, default route bentrok, sensor tidak terdaftar) didokumentasikan di `docs/blueteam/security_onion_setup.md` — dibuat juga untuk membantu kelompok lain yang mengalami kendala serupa.

**Rencana Attack Plan Fase 2:** 3 skenario dari 2 kategori (Reconnaissance: Nmap scan; Web/API: SQL Injection pada login; Exploitation/Akses: brute force/default credential) — detail di `docs/redteam/attack_plan.md`. Template Purple Team Loop sudah disiapkan di `docs/purple-team-log.md`, akan diisi bertahap mulai Minggu ke-10.

**Perubahan struktur folder repo:** folder `docs/phase-1-baseline/` (laporan Minggu ke-7) dihapus karena sudah tidak relevan dengan struktur yang diminta panduan Minggu ke-9; folder `docs/Design/` di-rename jadi `docs/design/` (lowercase, konsisten dengan path di panduan); folder kosong `sripts` (typo) diperbaiki jadi `scripts/`; ditambahkan struktur baru `docs/presentation-minggu9/`, `docs/blueteam/`, `docs/redteam/`, `docs/purple-team-log.md` sesuai Bagian XI panduan.

**Penyesuaian tampilan aplikasi:** Form login & dashboard Target Node awalnya masih berupa HTML polos (bare-bones), berpotensi terlihat jelas sebagai aplikasi dummy saat direview dosen. Tampilan disesuaikan agar meniru UI asli Bilicheck (halaman Sign In & Dashboard), dengan data dashboard bersifat statis/dummy. Logic login turut disesuaikan: verifikasi password dipisah menggunakan `bcrypt.checkpw` (bukan dibandingkan mentah di query SQL) agar login dengan kredensial yang benar tetap berhasil secara wajar, sementara celah SQL Injection tetap dipertahankan pada field `username` di query pencarian user.
