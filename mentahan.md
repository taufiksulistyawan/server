1. Update repository
sudo apt update -y && sudo apt upgrade -y

2. Ubah port ssh
nano /etc/ssh/sshd_config

pada tulisan #Port 22 hilangkan pagar dan ubah port sesuai keinginan
crtl+x y enter

restart ssh: systemctl restart sshd

3. Menambahkan user
adduser (nama)

menambahkan akses root: usermod -aG sudo (nama)

untuk cek: su - (nama)
groups

4. Mematikan root akses ssh
nano /etc/ssh/sshd_config

ubah: PermitRootLogin yes menjadi no
restart ssh: systemctl restart sshd

5. Akses ssh
ssh nama@ipaddress -p port

6. Atur Firewall
ufw allow port-ssh/tcp

ufw default allow outgoing
ufw default deny incoming

ufw enable

7. setup fail2ban
sudo apt install fail2ban
sudo systemctl status fail2ban
sudo systemctl enable --now fail2ban

lokasi konfigurasi: ls /etc/fail2ban/
membuat konfigurasi: sudo nano /etc/fail2ban/jail.local

[sshd]
enabled = true
port = ...
filter = sshd
logpath = /var/log/auth.log
maxretry = 5 (maksimal percobaan)
bantime = 1h (waktu banned)
findtime = 600 (waktu percobaan)

testing: sudo fail2ban-client -t
restart: systemctl restart fail2ban
cek status: sudo fail2ban-client status
sudo fail2ban-client status sshd

buka ban: sudo fail2ban-client set sshd unbanip IP_ADDRESS
