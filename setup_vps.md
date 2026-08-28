# Basic Server Setup & Security

Panduan dasar setup dan hardening server Linux berbasis Ubuntu/Debian.

> **Peringatan:** Sebelum mengubah konfigurasi SSH, pastikan Anda memiliki akses console/VNC dari provider VPS sebagai akses cadangan. Kesalahan konfigurasi SSH dapat menyebabkan server tidak bisa diakses melalui SSH.

---

## 1. Update Repository & System

Update repository dan upgrade package yang tersedia:

```bash
sudo apt update -y && sudo apt upgrade -y
```

---

## 2. Mengubah Port SSH

Edit konfigurasi SSH:

```bash
sudo nano /etc/ssh/sshd_config
```

Cari:

```text
#Port 22
```

Hilangkan tanda `#` dan ubah port sesuai kebutuhan, contoh:

```text
Port 2222
```

> Gunakan port yang tidak sedang digunakan oleh service lain.

Simpan konfigurasi:

```text
Ctrl + X
Y
Enter
```

### Cek konfigurasi SSH

Sebelum melakukan restart, sebaiknya cek terlebih dahulu apakah konfigurasi valid:

```bash
sudo sshd -t
```

Jika tidak ada output, berarti konfigurasi tidak memiliki error syntax.

Kemudian restart SSH:

```bash
sudo systemctl restart ssh
```

Cek status:

```bash
sudo systemctl status ssh
```

> **Penting:** Jangan langsung menutup session SSH yang sedang aktif. Buka terminal/session baru dan pastikan port SSH baru dapat digunakan terlebih dahulu.

---

## 3. Membuat User Baru

Buat user baru:

```bash
sudo adduser nama
```

Ganti `nama` dengan username yang diinginkan.

### Memberikan akses sudo

Tambahkan user ke grup `sudo`:

```bash
sudo usermod -aG sudo nama
```

### Mengecek akses user

Login sebagai user tersebut:

```bash
su - nama
```

Kemudian cek grup:

```bash
groups
```

Untuk memastikan user memiliki akses sudo:

```bash
sudo whoami
```

Jika berhasil, output:

```text
root
```

---

## 4. Menonaktifkan Root Login melalui SSH

Edit konfigurasi SSH:

```bash
sudo nano /etc/ssh/sshd_config
```

Cari:

```text
PermitRootLogin yes
```

Ubah menjadi:

```text
PermitRootLogin no
```

Jika konfigurasi masih berupa komentar:

```text
#PermitRootLogin prohibit-password
```

tambahkan konfigurasi aktif:

```text
PermitRootLogin no
```

### Cek konfigurasi

```bash
sudo sshd -t
```

Jika tidak ada error, restart SSH:

```bash
sudo systemctl restart ssh
```

> Pastikan user baru sudah dapat login dan menggunakan `sudo` sebelum menonaktifkan root login.

---

## 5. Login melalui SSH

Format:

```bash
ssh nama@IP_ADDRESS -p PORT
```

Contoh:

```bash
ssh admin@192.168.1.100 -p 2222
```

---

## 6. Konfigurasi Firewall (UFW)

Install UFW jika belum tersedia:

```bash
sudo apt install ufw -y
```

### Izinkan port SSH

Jika menggunakan port SSH `2222`:

```bash
sudo ufw allow 2222/tcp
```

> **PENTING:** Izinkan port SSH terlebih dahulu sebelum mengaktifkan UFW agar koneksi SSH tidak terblokir.

Atur default policy:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Aktifkan firewall:

```bash
sudo ufw enable
```

Cek status:

```bash
sudo ufw status verbose
```

Contoh output:

```text
Status: active

To                         Action      From
--                         ------      ----
2222/tcp                   ALLOW       Anywhere
```

Jika menggunakan service lain, misalnya HTTP/HTTPS:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

---

## 7. Install & Setup Fail2Ban

Install Fail2Ban:

```bash
sudo apt install fail2ban -y
```

Aktifkan service:

```bash
sudo systemctl enable --now fail2ban
```

Cek status:

```bash
sudo systemctl status fail2ban
```

Lokasi konfigurasi:

```bash
ls /etc/fail2ban/
```

---

### Membuat konfigurasi `jail.local`

Jangan mengubah `jail.conf` secara langsung. Buat konfigurasi lokal:

```bash
sudo nano /etc/fail2ban/jail.local
```

Tambahkan:

```ini
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
findtime = 600
bantime = 1h
```

Sesuaikan:

```ini
port = 2222
```

dengan port SSH yang digunakan server.

### Penjelasan konfigurasi

| Parameter  | Fungsi                                                |
| ---------- | ----------------------------------------------------- |
| `enabled`  | Mengaktifkan jail SSH                                 |
| `port`     | Port SSH yang akan dipantau                           |
| `filter`   | Filter log untuk mendeteksi percobaan login SSH gagal |
| `logpath`  | Lokasi log autentikasi                                |
| `maxretry` | Jumlah percobaan gagal sebelum IP diblokir            |
| `findtime` | Waktu penghitungan percobaan gagal                    |
| `bantime`  | Lama IP diblokir                                      |

Dengan konfigurasi di atas:

> IP akan diblokir selama **1 jam** apabila melakukan **5 kali percobaan gagal dalam 10 menit**.

---

## 8. Testing Konfigurasi Fail2Ban

Sebelum restart, lakukan pengecekan konfigurasi:

```bash
sudo fail2ban-client -t
```

Jika konfigurasi valid, akan muncul pesan bahwa konfigurasi berhasil.

Restart Fail2Ban:

```bash
sudo systemctl restart fail2ban
```

Cek status service:

```bash
sudo systemctl status fail2ban
```

Cek semua jail:

```bash
sudo fail2ban-client status
```

Cek jail SSH:

```bash
sudo fail2ban-client status sshd
```

Contoh informasi yang dapat ditampilkan:

```text
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  `- Total failed: 5
`- Actions
   |- Currently banned: 1
   `- Total banned: 1
```

---

## 9. Unban IP Address

Jika IP tertentu terblokir oleh Fail2Ban dan ingin dibuka kembali:

```bash
sudo fail2ban-client set sshd unbanip IP_ADDRESS
```

Contoh:

```bash
sudo fail2ban-client set sshd unbanip 192.168.1.50
```

Cek kembali daftar IP yang diblokir:

```bash
sudo fail2ban-client status sshd
```

---

## 10. Checklist Setelah Setup

Pastikan beberapa hal berikut sudah berhasil:

* [ ] System sudah di-update
* [ ] Port SSH sudah diubah
* [ ] User baru sudah dibuat
* [ ] User baru dapat menggunakan `sudo`
* [ ] Root login melalui SSH sudah dinonaktifkan
* [ ] SSH dapat diakses menggunakan port baru
* [ ] UFW sudah aktif
* [ ] Port SSH sudah diizinkan di UFW
* [ ] Fail2Ban sudah aktif
* [ ] Jail `sshd` sudah aktif
* [ ] Konfigurasi Fail2Ban sudah berhasil ditest
* [ ] IP yang diblokir dapat di-unban jika diperlukan

---
## 11. Install Speedtest CLI

Install jika belum tersedia:

```bash
sudo apt update
sudo apt install speedtest-cli -y
```
Kemudian jalankan:
```bash
speedtest-cli
```

## Catatan Keamanan

Sebelum mengubah konfigurasi SSH atau Firewall pada server remote:

1. **Jangan langsung menutup koneksi SSH yang sedang aktif.**
2. Pastikan konfigurasi SSH valid dengan:

   ```bash
   sudo sshd -t
   ```
3. Pastikan port SSH sudah diizinkan di firewall.
4. Buka koneksi SSH baru untuk melakukan pengujian.
5. Pastikan user baru dapat login dan menggunakan `sudo`.
6. Setelah semuanya dipastikan berjalan, barulah tutup session lama.

Dengan cara ini, risiko **server terkunci dari akses SSH akibat salah konfigurasi** dapat diminimalkan.
