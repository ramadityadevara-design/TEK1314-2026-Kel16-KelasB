# TEK1314-2026-Kel16-KelasB
Kelompok 16 Siber Security TEK61-B

## Skenario Proyek

Proyek ini mensimulasikan skenario SOC (Security Operations Center) dalam mengamankan infrastruktur backend dari sistem IoT kelompok, dalam lingkungan lab yang terisolasi (subnet `192.168.16.0/24`). Sesuai ketentuan Minggu ke-9, Target Node **mengadopsi tech stack asli proyek IoT (Bilicheck)** untuk simulasi (bukan sistem produksi) — lihat `docs/presentation-minggu9/tech_stack_declaration.md` untuk pemetaan lengkap.

- **Red Team**: menjalankan reconnaissance dan simulasi eksploitasi terhadap Target Node menggunakan **Kali Linux** (`192.168.16.100`). Rencana skenario serangan ada di `docs/redteam/attack_plan.md`.
- **Blue Team**: menyiapkan **Target Node** berbasis **Ubuntu Server 24.04.5 LTS + Flask + MySQL 8.0.46** (`192.168.16.5`), serta mengonfigurasi **Security Onion** (`192.168.16.200`) sebagai Monitoring Node untuk mendeteksi dan menganalisis aktivitas serangan. Checklist hardening ada di `docs/blueteam/hardening_checklist.md`; detail setup Security Onion ada di `docs/blueteam/security_onion_setup.md`.
- **Lead**: mengoordinasikan desain topologi, deklarasi tech stack, dan memastikan seluruh dokumentasi tersimpan sesuai struktur folder di bawah.

## Struktur Dokumentasi

- `docs/design/` — topologi jaringan & IP plan
- `docs/presentation-minggu9/` — slide & deklarasi tech stack (Minggu ke-9)
- `docs/blueteam/` — hardening checklist, setup & troubleshooting Security Onion, evidence
- `docs/redteam/` — attack plan, evidence hasil eksekusi serangan
- `docs/purple-team-log.md` — log 5 tahap Purple Team Loop per skenario serangan
- `docs/phase-2-va/`, `docs/phase-3-incident/` — laporan fase berikutnya (Minggu 11 & 12-15)
- `LOGBOOK.md` — catatan progres mingguan
