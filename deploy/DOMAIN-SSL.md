# panel.hypevisionlab.com + HTTPS

## 1. DNS (TurkTicaret)

| Alan | Değer |
|------|--------|
| Tip | A |
| Host | `panel` |
| IP | Sunucunun **gerçek** IPv4 adresi (`curl -4 ifconfig.me` ile kontrol) |
| TTL | 3600 |

Yayılma: 5–60 dakika. Kontrol:

```bash
dig +short panel.hypevisionlab.com
curl -4 ifconfig.me   # sunucuda — ikisi aynı IP olmalı
```

## 2. Sunucuda (tek komut)

```bash
cd ~/hp-pan
git pull
chmod +x deploy/setup-domain-ssl.sh
sudo ./deploy/setup-domain-ssl.sh
```

Script: nginx `server_name panel.hypevisionlab.com`, certbot Let's Encrypt, HTTP→HTTPS yönlendirme.

Farklı e-posta:

```bash
sudo CERTBOT_EMAIL=sizin@mail.com ./deploy/setup-domain-ssl.sh
```

## 3. Manuel (script kullanmazsan)

```bash
cd ~/hp-pan
sudo cp deploy/nginx-hypevision.conf /etc/nginx/sites-available/hypevision
sudo sed -i "s|/root/hp-pan|$(pwd)|g" /etc/nginx/sites-available/hypevision
sudo ln -sf /etc/nginx/sites-available/hypevision /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default
sudo chmod 755 /root && sudo chmod -R 755 dist
sudo nginx -t && sudo systemctl reload nginx

sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d panel.hypevisionlab.com --redirect
```

## Sorun giderme

| Sorun | Çözüm |
|--------|--------|
| certbot DNS hatası | A kaydı doğru IP mi, `dig @8.8.8.8 panel.hypevisionlab.com` |
| `X509_V_FLAG_NOTIFY_POLICY` / certbot çöküyor | **venv kapat:** `deactivate`, sonra snap certbot (aşağı) |
| 403 Forbidden | `chmod 755 /root` ve `chmod -R 755 ~/hp-pan/dist` |
| SSL yok | `sudo certbot certificates` |
| API 502 | `sudo systemctl status hypevision-api` |

### certbot OpenSSL hatası (venv çakışması)

```bash
deactivate
sudo snap install core && sudo snap refresh core
sudo snap install --classic certbot
sudo ln -sf /snap/bin/certbot /usr/bin/certbot
sudo env -i PATH="/snap/bin:/usr/sbin:/usr/bin" HOME=/root \
  certbot --nginx -d panel.hypevisionlab.com --redirect \
  --non-interactive --agree-tos -m admin@hypevisionlab.com
sudo nginx -t && sudo systemctl reload nginx
```

Panel: **https://panel.hypevisionlab.com**
