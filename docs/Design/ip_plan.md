# IP Plan - Kelompok 16

**Subnet:** `192.168.16.0/24`

| Hostname | IP Address | OS / Stack | Keterangan |
|---|---|---|---|
| Target Server (Korban) — `SRV-WEB-KEL16B` | 192.168.16.5 | Ubuntu Server 24.04.5 LTS + Flask (port 5000) + MySQL 8.0.46 | Backend proyek IoT Bilicheck (asli: Firebase), dibangun untuk simulasi lab. Berisi celah keamanan terkontrol untuk didemonstrasikan Red Team — lihat `docs/presentation-minggu9/tech_stack_declaration.md` |
| Attacker Node | 192.168.16.100 | Kali Linux | Digunakan Red Team untuk scanning & exploitation |
| Monitoring Node | 192.168.16.200 (eth0/management) | Security Onion (Evaluation Mode) | `eth1` = sniffing interface (tanpa IP, promiscuous), memantau seluruh trafik di segmen `192.168.16.0/24`. Lihat `docs/blueteam/security_onion_setup.md` untuk detail setup. |

> **Catatan perubahan:** versi awal tabel ini (dan port list di baliknya) merujuk ke skenario Metasploitable 2 + DVWA generik. Sejak ketentuan baru dari dosen (Minggu ke-9), Target Node WAJIB mengikuti tech stack asli proyek IoT (Bilicheck) kelompok — detail pemetaan lengkap ada di `tech_stack_declaration.md`.

## Port & Service yang Relevan (Target Node saat ini)

| Port | Service | Keterangan |
|---|---|---|
| 22 | SSH | Akses administrasi Target Node |
| 5000 | Flask (HTTP) | Aplikasi web dashboard IoT Bilicheck (tampilan Sign In & Dashboard mengikuti desain asli) — endpoint `/`, `/dashboard`, `/health` |
| 3306 | MySQL 8.0.46 | Database `iot_replica_db`, diakses lokal oleh Flask (tidak di-expose ke luar `localhost`) |

Daftar celah & rencana skenario serangan ada di `docs/redteam/attack_plan.md`.

## Catatan Routing

Semua node berada dalam satu segmen (`192.168.16.0/24`) melalui Internal Network virtual lab, sehingga tidak diperlukan tabel routing antar-subnet pada tahap ini. Gateway lab (formalitas, Internal Network tidak punya gateway fisik): `192.168.16.1`.

Security Onion juga punya interface tambahan (`eth2`, NAT) khusus untuk akses internet (download rule IDS), terpisah dari segmen lab ini — lihat `docs/blueteam/security_onion_setup.md`.
