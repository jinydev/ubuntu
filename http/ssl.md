# ssl을 통한 https 연결

## SSL 설치하기
You need the Certbot tool that uses Let's Encrypt API, to install SSL certificates. The latest version of Certbot is available via the Snap store.

Issue the following commands to ensure that you have the latest version of snapd.

```
$ sudo snap install core
$ sudo snap refresh core
```

## Certbot 설치

```
$ sudo snap install --classic certbot
```

Create a symlink for Certbot to the /usr/bin directory.

```
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

## Issue the SSL Certificate.
ssl 인증서를 생성합니다.

```
$ sudo certbot certonly --standalone --agree-tos --no-eff-email --staple-ocsp --preferred-challenges http -m name@example.com -d moodle.example.com
```

Generate a Diffie-Hellman group certificate.

```
$ sudo openssl dhparam -dsaparam -out /etc/ssl/certs/dhparam.pem 4096
```

Open the file /etc/letsencrypt/renewal/moodle.example.com.conf for editing.

```
$ sudo nano /etc/letsencrypt/renewal/moodle.example.com.conf
```

Paste the following code at the bottom.

```
pre_hook = systemctl stop nginx
post_hook = systemctl start nginx
```

Save the file by pressing Ctrl + X and entering Y when prompted.

The standalone option of Certbot uses its web server to create the certificate that doesn't work with Nginx. The pre_hook and post_hook commands run before and after the renewal to automatically shut and restart the Nginx server without manual intervention.

## 연장
Do a dry run of the SSL renewal process to ensure it works.

$ sudo certbot renew --dry-run
