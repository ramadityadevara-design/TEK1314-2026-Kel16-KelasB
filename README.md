## Skenario Proyek - Kelompok 16

Proyek ini mensimulasikan skenario serangan terhadap **Web/Database Server** dalam lingkungan lab yang terisolasi (subnet `192.168.16.0/24`). Kelompok terdiri dari tiga peran:

- **Red Team**: menjalankan reconnaissance dan eksploitasi terhadap Target Server menggunakan **Kali Linux** (`192.168.16.100`), menyasar celah pada service seperti FTP, Samba, dan MySQL.
- **Blue Team**: menyiapkan **Target Server** berbasis **Metasploitable 2** (`192.168.16.5`) sebagai korban, serta menempatkan **Security Onion** (`192.168.16.200`) sebagai Monitoring Node untuk mendeteksi dan menganalisis aktivitas serangan.
- **Lead**: mengoordinasikan desain topologi dan memastikan dokumentasi (topologi jaringan & IP plan) tersimpan di `docs/design/`.

Detail desain jaringan dan tabel IP dapat dilihat di:
- `docs/design/topology.png`
- `docs/design/ip_plan.md`
