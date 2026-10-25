# Deklarasi Tech Stack — Kelompok 16
Sesuai Bagian III, Panduan PBL Pertemuan ke-9 (TEK1314 Keamanan Siber)

Dokumen ini memetakan teknologi backend **asli** proyek IoT kelompok (**Bilicheck** — realtime bilirubin checker untuk deteksi jaundice bayi) terhadap bangunan simulasi di **Target Node** lab (VM `SRV-WEB-KEL16B`, `192.168.16.5`). Sesuai ketentuan, Target Node **bukan** sistem produksi — seluruh data yang dipakai adalah data dummy/sintetis.

| Komponen | Teknologi pada Proyek IoT Asli | Dibangun sebagai (Target Node) |
|---|---|---|
| Sistem Operasi backend | Firebase (managed backend/cloud; OS dikelola penyedia) | Ubuntu Server 24.04.5 LTS |
| Database/penyimpanan data | Firebase | MySQL 8.0.46 |
| Protokol komunikasi sensor | HTTP REST | HTTP ke backend Flask pada port 5000 |
| Dashboard/monitoring | Aplikasi Flutter menggunakan bahasa Dart | Dashboard web simulasi menggunakan Flask dan browser |

## Catatan Pembangunan Simulasi

- Karena proyek IoT asli menggunakan **Firebase** (backend as a service, bukan server konvensional), sementara anggota tim lain di proyek IoT menggunakan **Ubuntu Server + MySQL**, kelompok memutuskan mengombinasikan keduanya: OS & database mengikuti pendekatan rekan tim (Ubuntu + MySQL), sedangkan aplikasi backend ditulis dalam **Flask (Python)** sebagai representasi layer API yang menjembatani data sensor ke dashboard — fungsinya setara dengan peran Firebase di proyek asli.
- Tampilan (Sign In & Dashboard) pada Target Node dibuat mengikuti desain asli Bilicheck, supaya representatif terhadap sistem yang sedang dikembangkan kelompok — data yang ditampilkan bersifat statis/dummy.
- Seluruh kredensial & data pada Target Node adalah dummy, dibuat khusus untuk simulasi (lihat `docs/redteam/attack_plan.md` dan `docs/blueteam/hardening_checklist.md`).
- Risiko simulasi ini nol terhadap deliverable ke Mitra karena tidak ada koneksi apa pun antara Target Node dan sistem Firebase/IoT asli yang sedang berjalan.

## Status

Deklarasi ini final per Minggu ke-9 dan menjadi acuan untuk seluruh dokumentasi Fase 2 (hardening checklist, attack plan, purple team log).
