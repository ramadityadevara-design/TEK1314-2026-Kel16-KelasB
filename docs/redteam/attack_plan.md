# Attack Plan Fase 2 — Kelompok 16
Sesuai Bagian IX, Panduan PBL Pertemuan ke-9. Target: Target Node Bilicheck (`192.168.16.5`, Ubuntu Server + Flask + MySQL).

Minimal 3 skenario dari minimal 2 kategori berbeda — terpenuhi (3 skenario, 2 kategori: Reconnaissance & Web/API/Protokol IoT). Tidak ada skenario DoS/DDoS dipilih.

## Skenario 1 — Reconnaissance: Port & Service Scanning

| | |
|---|---|
| **Kategori** | Reconnaissance |
| **Teknik** | Port scanning, OS fingerprinting |
| **Tools** | `nmap` |
| **Target** | `192.168.16.5` (seluruh port, fokus 22, 5000, 3306) |
| **Command rencana** | `nmap -sS -sV -O 192.168.16.5` |
| **Hasil yang diharapkan** | Daftar port terbuka & versi service (Flask dev server, MySQL 8.0.46) — dasar untuk skenario eksploitasi berikutnya |
| **Relevansi** | Mengonfirmasi permukaan serangan sebelum eksploitasi; hasil riset ini jadi dasar penentuan target port 5000 untuk Skenario 2 |

## Skenario 2 — Web/API: SQL Injection pada Login Form

| | |
|---|---|
| **Kategori** | Web/API/Protokol IoT |
| **Teknik** | SQL Injection pada form login |
| **Tools** | Manual payload via `curl`/browser, opsional `sqlmap` untuk otomasi |
| **Target** | Endpoint `POST /` (form login) Flask app di `192.168.16.5:5000` |
| **Command rencana** | Payload contoh: `username = admin' OR '1'='1' -- ` pada field username, password bebas |
| **Hasil yang diharapkan** | Bypass autentikasi tanpa mengetahui password asli, karena query dibangun dengan string formatting langsung (bukan parameterized query) di versi "before" |
| **Relevansi** | Celah ini sengaja ditanam sebagai representasi risiko umum pada aplikasi web yang menjembatani data IoT ke database — relevan dengan tech stack Flask+MySQL yang diadopsi dari proyek asli Bilicheck (lihat `docs/presentation-minggu9/tech_stack_declaration.md`) |

## Skenario 3 — Exploitation/Akses: Brute Force / Default Credential pada MySQL

| | |
|---|---|
| **Kategori** | Exploitation / Akses |
| **Teknik** | Brute-force / default credential check |
| **Tools** | `Hydra` (brute force), atau manual testing kredensial lemah |
| **Target** | MySQL service `192.168.16.5:3306` (jika port sengaja dibuka untuk simulasi) atau SSH port 22 |
| **Command rencana** | `hydra -L userlist.txt -P passlist.txt 192.168.16.5 mysql` (atau target SSH, disesuaikan saat eksekusi) |
| **Hasil yang diharapkan** | Menunjukkan risiko kredensial lemah/default sebelum kontrol fail2ban & rate-limiting diterapkan (lihat `docs/blueteam/hardening_checklist.md`, kategori A) |
| **Relevansi** | Melengkapi bukti bahwa kontrol Access & Authentication (Bagian VIII.A) yang masih "Belum" memang berisiko nyata jika tidak segera dihardening |

## Catatan

- Seluruh skenario di atas dieksekusi HANYA terhadap Target Node Bilicheck lab (`192.168.16.5`), sesuai Rules of Engagement — tidak menyentuh sistem Firebase/IoT asli.
- Eksekusi penuh + bukti Purple Team Loop (5 tahap: Serang → Deteksi → Analisis → Respons → Verifikasi) untuk tiap skenario dicatat di `docs/purple-team-log.md`, mulai Minggu ke-10.
- Status saat ini (Minggu ke-9): rencana skenario sudah final, eksekusi penuh belum dimulai — menunggu jadwal Fase 2 (Minggu 9-11) sesuai Roadmap.
