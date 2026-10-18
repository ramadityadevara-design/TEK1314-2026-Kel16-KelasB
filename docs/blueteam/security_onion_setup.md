# Dokumentasi Setup Security Onion (Monitoring Ubuntu Server)
**Kelompok 16 — TEK1314 Keamanan Siber**
Image: Cisco CyberOps Security Onion (NetAcad, Evaluation Mode, versi legacy/klasik — bukan versi Salt/Docker terbaru)

> Dokumen ini ditulis berdasarkan pengalaman real troubleshooting kelompok kami. Kalau kelompok lain stuck di titik yang sama, semoga bisa dipakai sebagai referensi — bukan cuma command, tapi juga **kenapa** error itu terjadi.

---

## 0. Konteks Topologi

- **Ubuntu Server (Target Node)**: `192.168.16.5` — Flask app + MySQL
- **Kali Linux (Attacker Node)**: `192.168.16.100`
- **Security Onion (Monitoring Node)**:
  - `eth0` (management) → `192.168.16.200/24`, satu subnet dengan Ubuntu & Kali
  - `eth1` (sniffing) → tanpa IP, mode promiscuous, interface ini yang "mengintip" traffic
  - `eth2` (NAT, ditambahkan belakangan) → akses internet (download rules)

Semua VM pakai **Internal Network** yang sama di VirtualBox untuk `eth0` dan `eth1` Security Onion.

---

## 1. Import Ulang `.ova`

1. VirtualBox → **File → Import Appliance** → pilih file `.ova` dari NetAcad.
2. Biarkan setting default (CPU 1, RAM 4096MB, Import Hard Drives as VDI). RAM 3GB sebenarnya juga bisa jalan, tapi 4GB lebih aman.
3. Klik **Finish**, tunggu proses import selesai.

**Kenapa reset dari nol?** Karena troubleshooting manual (edit symlink, config Snort satu-satu) di percobaan pertama kami kepanjangan dan berisiko ada setting lain yang keliru tanpa disadari. Import ulang + ikuti wizard resmi dari awal lebih terjamin konsisten.

---

## 2. Atur Network Adapter SEBELUM Boot Pertama

Buka VM Settings → Network:

| Adapter | Fungsi | Attached To | Catatan |
|---|---|---|---|
| Adapter 1 | `eth0` (management) | Internal Network (nama sama dgn Ubuntu/Kali) | Dapat IP statis |
| Adapter 2 | `eth1` (sniffing) | Internal Network (nama sama) | **Promiscuous Mode → Allow All** di tab Advanced |

> ⚠️ **Penting:** Urutan Adapter 1/2 di VirtualBox **tidak selalu** = `eth0`/`eth1` di dalam Linux — tergantung deteksi hardware saat boot. Setelah boot, selalu cek dengan `ip a` dan cocokkan MAC Address kalau ragu.

---

## 3. Jalankan Wizard `sosetup`

```bash
sudo sosetup
```

Ikuti dialog **fase 1 (network)**:

1. "Would you like to continue?" → **Yes, Continue!**
2. "Which network interface should be the management interface?" → pilih yang jadi `eth0`
3. "Should ethX use DHCP or static addressing?" → **static**
4. IP statis management → `192.168.16.200`
5. Subnet mask → `255.255.255.0`
6. Gateway → `192.168.16.1` (formalitas, Internal Network VirtualBox tidak punya gateway fisik beneran — ini nanti jadi sumber bug, lihat Bagian 6)
7. DNS server → `8.8.8.8`
8. Domain → bebas, misal `example.com`
9. "Would you like to configure sniffing (monitor) interfaces?" → **Yes, configure sniffing interfaces.**
10. Centang interface yang jadi `eth1` → OK
11. Review summary → **Yes, make changes!**
12. **Yes, reboot!**

Setelah reboot, **jalankan `sudo sosetup` lagi** untuk lanjut fase 2. Kalau muncul dialog *"It looks like /etc/network/interfaces has already been configured... skip network configuration?"* → pilih **Yes, skip** (supaya tidak mengulang isian network dari awal).

Fase 2:
1. "Setup will configure: Elasticsearch, Logstash, Kibana, Squert, Sguil, Zeek, Snort/Suricata, netsniff-ng" → **Yes, Continue!**
2. "Evaluation Mode or Production Mode?" → **Evaluation Mode** (paling cocok untuk 1 VM standalone, bukan distributed deployment)
3. "Which network interface should be monitored?" → otomatis terpilih `eth1` (sesuai pilihan sniffing tadi) → OK
4. Buat user account (dipakai login Kibana/Squert/Sguil) → isi username, misal `analyst`, lalu email & password
5. Review summary → **Yes, proceed with the changes!**

Tunggu proses (bisa 10-20 menit). Kalau selesai, muncul **"Security Onion Setup is now complete!"**.

---

## 4. Masalah #1: Setup Gagal Download Rules (Tidak Ada Internet)

**Gejala:** Muncul dialog *"Setup was not able to download rules from the Internet. LOCAL_NIDS_RULE_TUNING has been set to 'yes'"*.

**Penyebab:** `eth0` dan `eth1` sama-sama di-set ke Internal Network (terisolasi dari internet) — jadi SO tidak bisa `apt`/download rule ET (Emerging Threats) sama sekali.

**Solusi: tambah Adapter 3 khusus NAT**

1. Matikan VM: `sudo shutdown now`
2. VirtualBox → Settings → Network → **Adapter 3** → Enable → Attached to: **NAT**
3. Nyalakan VM lagi, cek interface baru:
   ```bash
   ip -br a
   ```
   Harusnya muncul `eth2` dengan IP otomatis dari NAT DHCP (misal `10.0.4.x`).

---

## 5. Masalah #2: Internet Masih Gagal Walau `eth2` Sudah Ada IP

**Gejala:** `ping google.com` gagal dengan source IP `192.168.16.200` (IP `eth0`), padahal `eth2` sudah dapat IP NAT.

**Penyebab:** Ada **2 default route** bentrok:
```bash
ip route
# default via 192.168.16.1 dev eth0            <- SALAH (gateway formalitas, tidak ada device nyata)
# default via 10.0.4.2 dev eth2 metric 100      <- BENAR tapi kalah prioritas
```
Baris `gateway 192.168.16.1` yang kita isi di wizard (Bagian 3, poin 6) ternyata dibaca sebagai default route juga, padahal di situ cuma isian formalitas tanpa device routing beneran.

**Solusi sementara (langsung cek koneksi):**
```bash
sudo ip route del default via 192.168.16.1 dev eth0
ping -c 3 google.com
```

**Solusi permanen (supaya tidak balik lagi pas reboot):**
```bash
sudo nano /etc/network/interfaces
```
- Di stanza `iface eth0 inet static`, **hapus baris** `gateway 192.168.16.1` (baris `address`, `netmask`, `dns-*` tetap dipertahankan).
- Tambahkan stanza baru di paling bawah:
  ```
  auto eth2
  iface eth2 inet dhcp
  ```
Simpan, lalu:
```bash
sudo systemctl restart networking
ip route   # pastikan cuma 1 default route, lewat eth2
```

---

## 6. Download Rules yang Sebenarnya (Setelah Internet Jalan)

```bash
sudo rule-update
```

Kalau sebelumnya pernah gagal karena belum ada internet, `LOCAL_NIDS_RULE_TUNING` mungkin masih `yes` (artinya pakai rule lokal/bundled, bukan download fresh). Tidak masalah dijalankan ulang — begitu internet sudah jalan, `rule-update` otomatis download rule terbaru dari GitHub (Emerging Threats), proses ini menghasilkan puluhan ribu rule baru dan otomatis restart Barnyard2 & Snort untuk sensor yang terdaftar.

---

## 7. Masalah #3: Sensor Tidak Terdaftar (`sensortab` Kosong)

**Gejala:** `sudo nsm_sensor_ps-status` cuma menampilkan `ossec_agent (sguil)`, **tidak ada** Snort/Barnyard2/PCAP agent. File `/etc/nsm/sensortab` kosong (0 bytes).

**Penyebab:** Proses registrasi sensor untuk interface sniffing gagal diam-diam saat instalasi awal (kemungkinan besar karena waktu itu belum ada internet, lihat Bagian 4).

**Cara cek & perbaiki:**

1. Cek file backup/template original:
   ```bash
   cat /etc/nsm/sensortab.orig
   ```
   Biasanya ada baris untuk interface sniffing kalian, tapi **di-comment** (ada `#` di depan), misal:
   ```
   #seconion-eth1   1   8100   eth1
   ```

2. Buat `sensortab` baru dengan baris itu di-**uncomment**. **Wajib pakai TAB** sebagai pemisah kolom (bukan spasi biasa), karena script parsing-nya strict:
   ```bash
   printf 'seconion-eth1\t1\t8100\teth1\n' | sudo tee /etc/nsm/sensortab
   cat -A /etc/nsm/sensortab   # pastikan ada tanda ^I (tab) di antara kolom, bukan spasi
   ```

3. Start sensornya:
   ```bash
   sudo nsm_sensor_ps-start --sensor-name=seconion-eth1
   sudo nsm_sensor_ps-status --sensor-name=seconion-eth1
   ```
   Harus muncul 5 komponen: `netsniff-ng`, `pcap_agent`, `snort_agent-1`, `snort-1`, `barnyard2-1` — semua status `[ OK ]`.

   > Error `grep: /opt/bro/etc/node.cfg: No such file` yang mungkin muncul **tidak fatal** — itu soal Zeek/Bro, komponen terpisah dari pipeline Snort/Sguil.

**Catatan untuk kelompok lain:** nama sensor (`seconion-eth1`, `8100`, dll) itu pola standar dari template bawaan image ini. Kalau punya kalian interface sniffing-nya ternyata `eth0` (bukan `eth1`), sesuaikan baris di atas (`seconion-eth0 ... eth0`, port biasanya beda juga, cek `sensortab.orig` kalian masing-masing).

---

## 8. Verifikasi di Sguil

1. Buka aplikasi **Sguil** di desktop Security Onion.
2. Login: **Sguild Host** = `localhost`, **Port** = `7734` (bukan "localhost" lagi di field port), username/password sesuai yang dibuat di Bagian 3 fase 2.
3. Di layar "Select Network(s) to Monitor", centang sensor kalian (misal `seconion-eth1`), **jangan** centang `seconion-ossec` (itu host-based log, bukan network traffic).
4. Klik **Start SGUIL**.

**Test traffic dari Kali:**
```bash
ping -c 5 192.168.16.5
nmap -sS 192.168.16.5
```

> Kenapa pakai Nmap, bukan cuma ping? Karena rule default Snort/ET lebih banyak match untuk pola **scanning**, sementara ping biasa sering tidak match rule apa pun (hasilnya "Read 0 records" di log Barnyard2 — bukan error, cuma memang tidak ada rule yang cocok).

Kalau berhasil, akan muncul alert baru di tab **RealTime Events** dengan tanggal hari ini, contoh:
- `GPL ICMP_INFO PING *NIX` (dari ping)
- `ET SCAN Suspicious inbound...` / `ET SCAN Potential VNC Scan` (dari nmap)

---

## 9. Supaya Tidak Perlu Setup Manual Lagi Tiap Reboot

Sensor **tidak auto-start** sendiri saat boot di image ini (cek `systemctl list-unit-files | grep nsm` dan `sudo crontab -l` — tidak ada entry `@reboot` bawaan). Tambahkan manual:

```bash
sudo EDITOR=nano crontab -e
```
*(pakai `EDITOR=nano` eksplisit — kalau tidak, `crontab -e` bisa kebuka pakai `ed`, editor command-line kuno yang membingungkan)*

Tambahkan baris di paling bawah:
```
@reboot sleep 60 && /usr/sbin/nsm_sensor_ps-start --sensor-name=seconion-eth1 >> /var/log/nsm/sensor-autostart.log 2>&1
```
(`sleep 60` memberi waktu networking selesai boot dulu sebelum sensor di-start, mencegah race condition)

Simpan, verifikasi: `sudo crontab -l`.

---

## 10. Checklist Ringkas (Kalau Sudah Paham Alurnya)

- [ ] Import `.ova`, Adapter 1 & 2 = Internal Network sama, Promiscuous Allow All di Adapter 2
- [ ] `sudo sosetup` fase 1 (network, static IP management) → reboot
- [ ] `sudo sosetup` fase 2 (skip network, Evaluation Mode, pilih interface sniffing, bikin user) → tunggu selesai
- [ ] Kalau gagal download rules: tambah Adapter 3 NAT, fix default route (hapus `gateway` dari `eth0`, tambah stanza `eth2 dhcp`)
- [ ] `sudo rule-update`
- [ ] Cek `/etc/nsm/sensortab` — kalau kosong, uncomment dari `sensortab.orig`, pakai `printf ... \t ... | sudo tee`
- [ ] `sudo nsm_sensor_ps-start --sensor-name=<nama-sensor>`
- [ ] Buka Sguil, centang sensor, Start SGUIL
- [ ] Test `ping` + `nmap -sS` dari Kali, cek RealTime Events
- [ ] Tambahkan cron `@reboot` untuk auto-start sensor

---

*Disusun oleh Kelompok 16 — boleh dipakai/disesuaikan kelompok lain yang mengalami masalah serupa.*
