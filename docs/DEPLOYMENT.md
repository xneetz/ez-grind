# Deployment EZ GRIND ke VPS

Panduan ini menggunakan **Ubuntu 24.04**, **Node.js 22**, **systemd**, **Nginx**, dan **Let's Encrypt**. Ganti `ezgrind.example.com` dengan domain sendiri.

## 1. Persiapan server

```bash
ssh root@IP_VPS
apt update && apt upgrade -y
apt install -y git nginx curl ca-certificates ufw apache2-utils
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs
node --version
```

Node.js minimal versi 22.5 diperlukan karena EZ GRIND memakai `node:sqlite`.

## 2. Buat user service

```bash
adduser --system --group --home /opt/ez-grind ezgrind
```

## 3. Clone private repository

Gunakan SSH deploy key read-only:

```bash
sudo -u ezgrind mkdir -p /opt/ez-grind/.ssh
sudo -u ezgrind ssh-keygen -t ed25519 -C "ez-grind-vps" -f /opt/ez-grind/.ssh/id_ed25519 -N ""
cat /opt/ez-grind/.ssh/id_ed25519.pub
```

Tambahkan public key di **Repository → Settings → Deploy keys**, lalu:

```bash
sudo -u ezgrind ssh-keyscan github.com >> /opt/ez-grind/.ssh/known_hosts
sudo -u ezgrind git clone git@github.com:xneetz/ez-grind.git /opt/ez-grind/app
```

## 4. Siapkan data

```bash
mkdir -p /opt/ez-grind/app/data/attachments
mkdir -p /opt/ez-grind/app/data/safety-backups
chown -R ezgrind:ezgrind /opt/ez-grind
chmod 750 /opt/ez-grind /opt/ez-grind/app/data
```

Data persisten berada di `data/ezdeck.db`, `data/attachments/`, dan `data/safety-backups/`. Jangan mengganti direktori ini saat update.

## 5. Systemd service

Buat `/etc/systemd/system/ez-grind.service`:

```ini
[Unit]
Description=EZ GRIND productivity planner
After=network.target

[Service]
Type=simple
User=ezgrind
Group=ezgrind
WorkingDirectory=/opt/ez-grind/app
Environment=NODE_ENV=production
Environment=PORT=3000
ExecStart=/usr/bin/node /opt/ez-grind/app/server.js
Restart=on-failure
RestartSec=5
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=full
ReadWritePaths=/opt/ez-grind/app/data

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable --now ez-grind
systemctl status ez-grind
curl http://127.0.0.1:3000/api/health
```

## 6. Tambahkan autentikasi

EZ GRIND belum memiliki login bawaan. Buat Basic Auth:

```bash
htpasswd -c /etc/nginx/.htpasswd ezgrind
```

Alternatif yang lebih kuat adalah Cloudflare Access, VPN, atau Tailscale.

## 7. Nginx

Buat `/etc/nginx/sites-available/ez-grind`:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name ezgrind.example.com;
    client_max_body_size 15M;

    auth_basic "EZ GRIND";
    auth_basic_user_file /etc/nginx/.htpasswd;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 120s;
    }
}
```

```bash
ln -s /etc/nginx/sites-available/ez-grind /etc/nginx/sites-enabled/ez-grind
nginx -t
systemctl reload nginx
```

## 8. HTTPS

```bash
apt install -y certbot python3-certbot-nginx
certbot --nginx -d ezgrind.example.com
certbot renew --dry-run
```

## 9. Firewall

```bash
ufw allow OpenSSH
ufw allow 'Nginx Full'
ufw enable
```

Port 3000 tidak perlu dibuka ke publik.

## 10. Backup

Gunakan **Settings → Full Backup** atau:

```bash
mkdir -p /var/backups/ez-grind
systemctl stop ez-grind
tar -czf "/var/backups/ez-grind/ez-grind-$(date +%F-%H%M%S).tar.gz" -C /opt/ez-grind/app data
systemctl start ez-grind
```

## 11. Update

```bash
cd /opt/ez-grind/app
systemctl stop ez-grind
sudo -u ezgrind cp -a data "data-backup-$(date +%F-%H%M%S)"
sudo -u ezgrind git pull --ff-only origin main
systemctl start ez-grind
systemctl status ez-grind
curl http://127.0.0.1:3000/api/health
```

## 12. Google Drive backup

Aktifkan Google Drive API, buat OAuth 2.0 Client ID bertipe Web application, lalu tambahkan origin produksi seperti `https://ezgrind.example.com` ke Authorized JavaScript origins. Masukkan Client ID di Settings. Jangan menaruh Client Secret di frontend atau repository.

## Checklist produksi

- [ ] Node.js 22.5+
- [ ] Service berjalan sebagai user non-root
- [ ] Direktori data dapat ditulis user `ezgrind`
- [ ] Nginx dan HTTPS aktif
- [ ] Autentikasi tambahan aktif
- [ ] Port 3000 tidak terbuka ke publik
- [ ] Backup terjadwal dan restore pernah diuji
- [ ] Health check mengembalikan `ok: true`
