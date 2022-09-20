# How to Install Laravel on Ubuntu Server 22.04

## ubuntu 22.04 서버를 준비합니다.
* 우분투 서버 준비
* 도메인 준비
> vultr 가상서버 생성


## step1. 방화벽 설정
`HTTP` 와 `HTTPS` 접속을 위하여 포트를 활성화 합니다.

Open them using the Uncomplicated Firewall (UFW).

```
$ sudo ufw allow http
$ sudo ufw allow https
```
Check the status of the firewall.

```
$ sudo ufw status
```

## 2. Install Nginx

Ubuntu 22.04 ships with an older version of Nginx. This article prefers using the official Nginx repository to install the latest version.

Install dependencies required for the Nginx install.

```
$ sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring -y
```

Import Nginx's signing key.

```
$ curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
| sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
```

Add the repository for Nginx's stable version.

```
$ echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg arch=amd64] \
http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
| sudo tee /etc/apt/sources.list.d/nginx.list
```

Update the system repository list.

```
$ sudo apt update
```

Install the Nginx server.

```
$ sudo apt install nginx
```

Verify the installation.

```
$ nginx -v
```


### nginx 완전삭제

```
apt-get remove --purge nginx nginx-full nginx-common 
```


## 3. PHP 설치 및 설정

The latest version of Moodle (v4.0.2) requires PHP 8.0. 
Add Ondrej's PHP repository to add support for PHP 8.0.

저장소 연결
```
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
```

```
$ sudo add-apt-repository ppa:ondrej/php -y
```

Install PHP 

```
sudo apt install php8.1
```

Install PHP 8.1 FPM for Nginx

```
sudo apt install php8.1-fpm
```

and the required extensions.
### PHP 확장 패키지 설치
```
sudo apt install php8.1-common php8.1-mysql php8.1-xml php8.1-xmlrpc php8.1-curl php8.1-gd php8.1-imagick php8.1-cli php8.1-dev php8.1-imap php8.1-mbstring php8.1-opcache php8.1-soap php8.1-zip php8.1-redis php8.1-intl -y
```

```
$ sudo apt install graphviz aspell ghostscript clamav php8.1-cli php8.1-pspell php8.1-curl php8.1-gd php8.1-intl php8.1-mysql php8.1-xml php8.1-xmlrpc php8.1-ldap php8.1-zip php8.1-soap php8.1-mbstring
```

### Configure PHP 8.1

```
sudo nano /etc/php/8.1/fpm/php.ini
```

Open the file /etc/php/8.0/fpm/pool.d/www.conf.

```
$ sudo nano /etc/php/8.0/fpm/pool.d/www.conf
```

Find the user=apache and group=apache lines in the file and change them as follows.

...
; Unix user/group of processes
; Note: The user is mandatory. If the group is not set, the default user's group
;       will be used.
user = nginx
group = nginx
...
Also, find the lines listen.owner=www-data and listen.group=www-data in the file and change them to nginx.

...
listen.owner = nginx
listen.group = nginx
...
Save the file by pressing CTRL+X, then Y.

Restart the PHP-FPM service.

```
$ sudo systemctl restart php8.1-fpm
```

## 4. Install and Configure MySQL
Install MySQL server.

```
$ sudo apt install mysql-server
```

The following step is necessary for MySQL versions 8.0.28 and above. Enter the MySQL Shell.

```
$ sudo mysql
```

Run the following command to set the password for your root user. Make sure it has a mix of numbers, uppercase, lowercase, and special characters.

```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourrootpassword';
```

Exit the shell.

```
mysql> exit
```
Run the Secure installation script.

```
$ sudo mysql_secure_installation
```

Answer the questions as follows to secure MySQL.

```
Would you like to setup VALIDATE PASSWORD component?
Press y|Y for Yes, any other key for No: (Enter Y)

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: (Enter 2)
Change the password for root ? ((Press y|Y for Yes, any other key for No) : (Enter N)
Remove anonymous users? (Press y|Y for Yes, any other key for No) : (Enter Y)
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : (Enter Y)
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : Y (Enter Y)
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : (Enter Y)
Log in to the MySQL shell.

```

```
$ sudo mysql -u root -p
```

Create a database for Moodle.

```
mysql > CREATE DATABASE laravel DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

Create an SQL user to access the database. Replace yourpassword with a strong password of your choice.

```
mysql > CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'yourpassword';
```

Grant moodleuser access to the database.

```
mysql > GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,CREATE TEMPORARY TABLES,DROP,INDEX,ALTER ON moodledb.* TO moodleuser@localhost;
```

Reload the privilege table.
```
mysql > FLUSH PRIVILEGES;
```

Exit the shell.

```
mysql > exit
```



## 7. Install SSL
You need the Certbot tool that uses Let's Encrypt API, to install SSL certificates. The latest version of Certbot is available via the Snap store.

Issue the following commands to ensure that you have the latest version of snapd.

```
$ sudo snap install core
$ sudo snap refresh core
```

Install Certbot.

```
$ sudo snap install --classic certbot
```

Create a symlink for Certbot to the /usr/bin directory.

```
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Issue the SSL Certificate.

```
$ sudo certbot certonly --standalone --agree-tos --no-eff-email --staple-ocsp --preferred-challenges http -m name@example.com -d tms.jinyerp.com
```

Generate a Diffie-Hellman group certificate.

$ sudo openssl dhparam -dsaparam -out /etc/ssl/certs/dhparam.pem 4096
Open the file /etc/letsencrypt/renewal/moodle.example.com.conf for editing.

$ sudo nano /etc/letsencrypt/renewal/moodle.example.com.conf
Paste the following code at the bottom.

pre_hook = systemctl stop nginx
post_hook = systemctl start nginx
Save the file by pressing Ctrl + X and entering Y when prompted.

The standalone option of Certbot uses its web server to create the certificate that doesn't work with Nginx. The pre_hook and post_hook commands run before and after the renewal to automatically shut and restart the Nginx server without manual intervention.

Do a dry run of the SSL renewal process to ensure it works.

$ sudo certbot renew --dry-run
8. Configure Nginx
Open the file nginx.conf for editing.

$ sudo nano /etc/nginx/nginx.conf
Find the line include /etc/nginx/conf.d/*.conf; and paste the following code below it.

server_names_hash_bucket_size  64;
Save the file by pressing CTRL+X, then Y.

Create the Moodle configuration file for Nginx and open it for editing.

$ sudo nano /etc/nginx/conf.d/moodle.conf
Paste the following code in it.

# Redirect HTTP to HTTPS
server {
    listen 80;  listen [::]:80;
    server_name moodle.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    server_name moodle.example.com;
    root   /var/www/html/moodle;
    index  index.php;

    ssl_certificate     /etc/letsencrypt/live/moodle.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/moodle.example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/moodle.example.com/chain.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;

    access_log /var/log/nginx/moodle.access.log main;
    error_log  /var/log/nginx/moodle.error.log;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ ^(.+\.php)(.*)$ {
        fastcgi_split_path_info ^(.+\.php)(.*)$;
        fastcgi_index index.php;
        fastcgi_pass unix:/run/php/php8.0-fpm.sock;
        include /etc/nginx/mime.types;
        include fastcgi_params;
        fastcgi_param  PATH_INFO  $fastcgi_path_info;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    # Hide all dot files but allow "Well-Known URIs" as per RFC 5785
    location ~ /\.(?!well-known).* {
        return 404;
    }

    # This should be after the php fpm rule and very close to the last nginx ruleset.
    # Don't allow direct access to various internal files. See MDL-69333
    location ~ (/vendor/|/node_modules/|composer\.json|/readme|/README|readme\.txt|/upgrade\.txt|db/install\.xml|/fixtures/|/behat/|phpunit\.xml|\.lock|environment\.xml) {
        deny all;
        return 404;
    }
}
Save the file by pressing CTRL+X, then Y.

Verify Nginx configuration syntax.

$ sudo nginx -t
Restart the Nginx service.

$ sudo systemctl restart nginx
9. Complete Moodle Install
Open the URL https://moodle.example.com in your browser to open the welcome screen.

Press the Continue button to proceed. The following page checks for system requirements. If everything is okay, move to the next screen.

The next page sets up the database and files required by Moodle. Continue to create your administrator account and fill in other details on the next screen.

The final step is to set up Moodle's front page. Save your changes to proceed to the Moodle dashboard. You can start using the application to create your learning platform.