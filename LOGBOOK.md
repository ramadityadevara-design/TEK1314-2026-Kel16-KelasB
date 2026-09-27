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
