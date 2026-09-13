# IP Plan - Kelompok 16

**Subnet:** `192.168.16.0/24`

| Hostname | IP Address | OS Direncanakan | Keterangan |
|---|---|---|---|
| Target Server (Korban) | 192.168.16.5 | Metasploitable 2 | Berisi service dengan celah keamanan default (FTP, Samba, MySQL, dsb.) untuk didemonstrasikan Red Team |
| Attacker Node | 192.168.16.100 | Kali Linux | Digunakan Red Team untuk scanning & exploitation |
| Monitoring Node | 192.168.16.200 | Security Onion | Diposisikan agar dapat memantau seluruh trafik di segmen 192.168.16.0/24 |

## Port yang Berpotensi Dibuka/Dieksploitasi (riset Red Team)

| Port | Service | Potensi Celah |
|---|---|---|
| 21 | FTP (vsftpd 2.3.4) | Backdoor command execution (dikenal di Metasploitable) |
| 22 | SSH | Brute force credential lemah |
| 23 | Telnet | Transmisi cleartext, mudah disadap |
| 80 | HTTP (Apache/DVWA) | SQL Injection, XSS pada aplikasi web |
| 139/445 | Samba | Exploit remote code execution (mis. usermap_script) |
| 3306 | MySQL | Weak/default credential, akses database langsung |

## Catatan Routing

Semua node berada dalam satu segmen (`192.168.16.0/24`) melalui satu switch/hub virtual lab, sehingga tidak diperlukan tabel routing antar-subnet pada tahap ini. Gateway lab (jika ada): `192.168.16.1`.
