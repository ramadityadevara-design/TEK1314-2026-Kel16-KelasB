# Purple Team Log — Kelompok 16
Sesuai Bagian X, Panduan PBL Pertemuan ke-9. Log 5 tahap (Serang → Deteksi → Analisis → Respons → Verifikasi) untuk setiap skenario di `docs/redteam/attack_plan.md`. Diisi bertahap mulai Minggu ke-10, dimutakhirkan hingga Minggu ke-11.

---

## Skenario 1 — Reconnaissance (Nmap Scan)

| Tahap | Red Team | Blue Team | Bukti |
|---|---|---|---|
| 1. Serang (Baseline) | _Belum dieksekusi_ | _Belum diamati_ | — |
| 2. Deteksi | — | _Menunggu_ | — |
| 3. Analisis | _Menunggu_ | _Menunggu_ | — |
| 4. Respons (Mitigasi) | — | _Menunggu_ | — |
| 5. Verifikasi | _Menunggu_ | _Menunggu_ | — |

## Skenario 2 — Web/API (SQL Injection pada Login)

| Tahap | Red Team | Blue Team | Bukti |
|---|---|---|---|
| 1. Serang (Baseline) | _Belum dieksekusi_ | _Belum diamati_ | — |
| 2. Deteksi | — | _Menunggu_ | — |
| 3. Analisis | _Menunggu_ | _Menunggu_ | — |
| 4. Respons (Mitigasi) | — | _Menunggu (rencana: ganti ke parameterized query)_ | — |
| 5. Verifikasi | _Menunggu_ | _Menunggu_ | — |

## Skenario 3 — Exploitation/Akses (Brute Force / Default Credential)

| Tahap | Red Team | Blue Team | Bukti |
|---|---|---|---|
| 1. Serang (Baseline) | _Belum dieksekusi_ | _Belum diamati_ | — |
| 2. Deteksi | — | _Menunggu_ | — |
| 3. Analisis | _Menunggu_ | _Menunggu_ | — |
| 4. Respons (Mitigasi) | — | _Menunggu (rencana: fail2ban + rate-limiting)_ | — |
| 5. Verifikasi | _Menunggu_ | _Menunggu_ | — |

---

**Status infrastruktur monitoring (prasyarat Purple Team Loop):** Security Onion (sensor `seconion-eth1`) sudah terverifikasi aktif memantau trafik ke Target Node sejak Minggu ke-9 — lihat `docs/blueteam/security_onion_setup.md` untuk detail setup & troubleshooting.
