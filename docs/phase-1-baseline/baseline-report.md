# Baseline Report - Fase 1 (Hardening Review)
## Kelompok 16

**Skenario:** Simulasi serangan web server pada jaringan berskala kecil.
**Tanggal Demo:** Minggu ke-7

---

## 1. Identitas Sistem

| Node | Hostname | IP Address | OS |
|---|---|---|---|
| Target Server | SRV-WEB-KEL16B | 192.168.16.5 | Ubuntu Server + web app rentan (DVWA) |
| Attacker Node | ATTACKER-KEL16B | 192.168.16.100 | Kali Linux |
| Monitoring Node | MONITORING-KEL16B | 192.168.16.200 | Security Onion |

> Referensi topologi lengkap: lihat `docs/design/topology.png` dan `docs/design/ip_plan.md`

---

## 2. Network Hardening

**Firewall (UFW) di Target Server:**

- Policy default: deny incoming, allow outgoing
- Port yang dibuka: `22/tcp` (SSH, dibatasi hanya dari IP Attacker/Admin), `80/tcp` (HTTP, untuk web app)
- Port lain: ditutup

```
# Contoh output: sudo ufw status verbose
[TEMPEL SCREENSHOT / OUTPUT DI SINI]
```

**Alasan:** (isi — misal: membatasi permukaan serangan hanya pada service yang benar-benar dibutuhkan skenario)

---

## 3. System Hardening

- [ ] Service tidak perlu dinonaktifkan (sebutkan: ..............)
- [ ] User non-root dibuat untuk operasional (`adduser namauser`, hindari login root langsung)
- [ ] Security patch diupdate (`sudo apt update && sudo apt upgrade -y`)
- [ ] SSH hardening (opsional): disable root login, ganti port default jika perlu

**Bukti (screenshot/output):**

```
[TEMPEL OUTPUT DI SINI, misal: cat /etc/ufw/user.rules]
```

---

## 4. Verifikasi Logging (Security Onion)

**Tujuan:** Membuktikan bahwa Monitoring Node berhasil merekam aktivitas jaringan sebelum fase serangan dimulai.

**Prosedur pengujian:**
1. Dari Attacker Node (192.168.16.100), lakukan ping ke Target Server (192.168.16.5)
2. Buka dashboard Sguil / Squert di Security Onion
3. Verifikasi log menunjukkan aktivitas ICMP dengan timestamp, IP asal, dan IP tujuan yang sesuai

**Bukti Screenshot:**

`![Dashboard Squert - ICMP Log](./assets/squert-icmp-log.png)`

| Timestamp | IP Asal | IP Tujuan | Jenis Aktivitas |
|---|---|---|---|
| (isi dari log) | 192.168.16.100 | 192.168.16.5 | ICMP (Ping) |

---

## 5. Kesimpulan Fase Baseline

(Ringkas 2-3 kalimat: status kesiapan sistem sebelum masuk fase simulasi serangan, apakah semua checklist sudah terpenuhi, dan catatan kendala jika ada.)
