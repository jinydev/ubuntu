# 우분투 라라벨 환경설정 및 배포
> Ubuntu Server 22.04
우분투 서버에 라라벨을 설치하고, 개발된 코드를 배포하는 방법에 대해서 알아봅니다.


## step1. 준비물
> 서버 세팅을 위한 몇가지 준비사항에 대해서 알아봅니다.

### 우분투 서버 준비
라라벨을 설치할 우분투 서버를 준비합니다. 우분투는 직접설치, 가상머신, 클라우드 등 다양한 방법으로
서버를 준비 할 수 있습니다.
* 컴퓨터 직접 설치
* virtualBox 가상머신으로 설치
* 클라우드 설치 (vultr)

### 도메인
웹 서비스 접속을 위한 도메인을 준비합니다. 도메인의 `http` 통하여 웹서버의 `80`포트에 접속합니다. 
또는, `https`를 통하여 `443`번 포트에 접속을 시도 합니다.
> 가상호스트 설정을 통하여 하나의 컴퓨터에 여러개의 도메인 설정도 가능합니다.


## step2. 방화벽 설정
`HTTP` 와 `HTTPS` 접속을 위하여 '80' 포트와 `443` 포트 접속을 허용해야 합니다.
방화벽 설정 명령은 `ufw` 입니다. 다음 명령을 입력하여 방화벽을 해제 합니다.

```
$ sudo ufw allow http
$ sudo ufw allow https
```

설정이 잘되었는지 `status` 옵션을 통하여 확인을 합니다.

```
$ sudo ufw status
```

## step3. 웹서버 설치(Nginx)
Nginx 웹서버를 설치합니다. 

### 저장소 위치추가
우분투 22.04는 이전의 nginx 버젼을 탑제하고 있습니다. 최신의 버젼을 설치하기 위해서는
공식 nginx 저장소를 추가하여 최신의 버젼으로 설치할 수 있도록 추가 설정이 필요합니다.

#### 의존성
nginx는 몇개의 의존성 패키지를 먼저 설치해 주어야 합니다. 
다음과 같이 명령을 입력해서 설치합니다. 

```
$ sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring -y
```

#### Nginx's signing key 삽입.

```
$ curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
| sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
```

#### Nginx's 안정 버젼 저장소 추가.

```
$ echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg arch=amd64] \
http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
| sudo tee /etc/apt/sources.list.d/nginx.list
```

저장소를 추가한 후에는, 시스템의 패키지 목록을 갱신해 주어야 설치가 가능합니다.

```
$ sudo apt update
```

#### Nginx 설치
이제 다음 명령을 통하여 Nginx를 설치합니다.

```
$ sudo apt install nginx
```

Nginx가 잘 설치되었는지 버젼을 확인해 봅니다.

```
$ nginx -v
```


#### nginx 완전삭제
만일 Nginx를 다시 설치하거나, 삭제를 원하는 경우 다음과 같이 명령을 입력합니다.

```
apt-get remove --purge nginx nginx-full nginx-common 
```

> --purge 옵션을 사용하지 않으면, 이전에 설정한 nginx의 파일들이 계속 남아 있게 됩니다.



## step4. PHP 설치 및 설정
우분투에 최신의 PHP 버젼을 설치합니다.

### 저장소 연결
최신의 PHP 버젼을 설치하기 위해서는 추가 저장소를 등록하여, 패키지의 설치 정보를 읽어 와야 합니다.

```
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php -y
```

저장소를 추가한 후에 시스템의 패키지 목록을 갱신합니다.

```
sudo apt update
```

### PHP 설치 
이제 다음 명령을 통하여 PHP를 설치합니다. `php`뒤에 `x.x`의 버젼을 추가하여 설치합니다.

```
sudo apt install php8.1
```

### PHP-FPM 설치


Nginx를 웹서버로 사용하는 경우 추가로 `PHP-FPM`을 설치 및 실행해 주어야 합니다. 
별도 서버로 구동되어 속도가 빠른 PHP-FPM 을 설치합니다.


```
sudo apt install php8.1-fpm
```
> 설치된 PHP 버젼과 맞는 fpm을 선택하여 설치합니다.


### PHP 확장 패키지 설치
추가적으로 PHP 확장 패키지를 같이 설치해 줍니다.

```
sudo apt install php8.1-common php8.1-curl php8.1-gd php8.1-imagick php8.1-cli php8.1-dev php8.1-imap php8.1-mbstring php8.1-opcache php8.1-soap php8.1-zip php8.1-redis php8.1-intl php8.1-xmlrpc php8.1-xml php8.1-mysql php8.1-ldap php8.1-pspell -y
```


```
$ sudo apt install graphviz aspell ghostscript clamav          
```

### PHP 8.1 설정
`php.ini`의 설정을 변경합니다.

```
sudo nano /etc/php/8.1/fpm/php.ini
```

> php.ini 의 설정을 운영 환경에 맞게 수정하며 대상 파일은 php-fpm 이 사용하는 /etc/php/7.0/fpm/php.ini 과 명령행에서 구동할 경우 사용하는 /etc/php/7.0/cli/php.ini 2개입니다.


Open the file /etc/php/8.0/fpm/pool.d/www.conf.

```
$ sudo nano /etc/php/8.0/fpm/pool.d/www.conf
```

서버에서 php를 실행할 수 있는 사용자를 변경합니다.
설정파일에서 `user=apache` 와 `group=apache` 가 있는 라인을 찾아 다음과 같이 변경합니다.

```ini
...
; Unix user/group of processes
; Note: The user is mandatory. If the group is not set, the default user's group
;       will be used.
user = nginx
group = nginx
...
```


또한, `listen.owner=www-data` 와 `listen.group=www-data` 부분을 찾아 nginx가 실행될 수 있도록 변경합니다.

```ini
...
listen.owner = nginx
listen.group = nginx
...
```

설정한 파일을 저장합니다. 
> nano 에디터에서 `CTRL+X`를 누른후 `Y`를 선택 합니다..

설정 파일을 수정후에 `PHP-FPM` 서비스를 재시작 합니다.

```
$ sudo systemctl restart php8.1-fpm
```

### nginx 실행하기
`systemctl` 명령을 통하여 nginx를 실행합니다. 

```
$ sudo systemctl start nginx
```

> 만일, 실행에 오류가 있는 경우 서버를 재시작한 후에 다시 명령을 입력합니다.


## step5. MySQL 설치 및 설정
라라벨은 관계형 데이터베이스와 연동하여 동작합니다. 이를 위해서 오픈소스인 MySQL을 같이 설치해 줍니다.

### 서버 설치
다음 명령을 입력하여 mysql server를 설치합니다.

```
$ sudo apt install mysql-server
```

> The following step is necessary for MySQL versions 8.0.28 and above. Enter the MySQL Shell.

### mysql 실행
`mysql` 명령을 입력하여 서버에 접속합니다.

```
$ sudo mysql
```

### 사용자 추가
MySQL을 설치한 후에 제일 먼저 해주어야 하는 것은 DB 사용자를 추가하는 것입니다.
먼저 `root` 사용자의 패스워드를 다음과 같이 추가합니다.
```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourrootpassword';
```
> 패스워드에는 숫자, 대문자, 소문자, 특수문자가 같이 포함되어 있어야 합니다.

설정을 완료 했으면, mysql 클라이언트를 종료합니다.

```
mysql> exit
```

### 보완 설정
Secure installation 스크립트를 실행합니다. 

```
$ sudo mysql_secure_installation
```

MySQL 보완을 위하여 다음 질문들 몇개를 설정합니다.

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

설정후 다시 `root` 계정으로 접속을 하여 확인합니다.

```
$ sudo mysql -u root -p
```

### 사용자계정 및 스키마 생성
새로운 사용자와 스키마를 생성하고 권한을 부여합니다.

#### 스키마 생성
```
mysql > CREATE DATABASE laravel DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

#### 사용자생성
데이터베이스에 접속 가능한 새로운 사용자를 추가합니다.
```
mysql > CREATE USER 'jiny'@'localhost' IDENTIFIED BY 'yourpassword';
```

> Replace yourpassword with a strong password of your choice.


그리고 생성한 사용자계정이 스키마에 접근이 가능하도록 권한을 설정합니다.

```
mysql > GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,CREATE TEMPORARY TABLES,DROP,INDEX,ALTER ON moodledb.* TO jiny@localhost;
```

사용자와 권한이 적용하도록 테이블을 다시 로드합니다.

```
mysql > FLUSH PRIVILEGES;
```

shell을 종료하고 나갑니다.

```
mysql > exit
```



## step6. SSL 설치
보안 웹페이지 접근을 위한 `https`를 사용하기 위해서는 인증서를 같이 설치해 주어야 합니다.
`Let's Encrypt`를 이용하면 무료로 인증서를 발급받을 수 있습니다.

이를 편리하게 발급 및 갱신을 위한 `Certbot` 도구를 같이 설치합니다.

### Snap store 갱신
Snap Store를 통하여 최신 버젼의 `Cerbot`을 설치합니다.
이를 위하여 먼저 snap 을 최신 상태로 갱신을 합니다.

```
$ sudo snap install core
$ sudo snap refresh core
```

### Certbot 설치
Certbot을 설치합니다.

```
$ sudo snap install --classic certbot
```

Certbot 명령을 어디서든지 호출할 수 있도록 `/usr/bin` 디랙터리에 심볼 링크를 생성합니다.

```
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### SSL 인증서 생성
도메인에 대한 인증서를 생성합니다. 인증서를 생성하기 위해서는 먼저 도메인을 준비되어 있어야 합니다.

```
$ sudo certbot certonly --standalone --agree-tos --no-eff-email --staple-ocsp --preferred-challenges http -m name@example.com -d jiny.example.com
```

Generate a Diffie-Hellman group certificate.

```
$ sudo openssl dhparam -dsaparam -out /etc/ssl/certs/dhparam.pem 4096
```

선택한 도메인의 인증서 설정파일이 같이 생성됩니다.
`/etc/letsencrypt/renewal/jiny.example.com.conf`을 열어 내용을 수정합니다.

```
$ sudo nano /etc/letsencrypt/renewal/jiny.example.com.conf
```

파일 하단에 아래 내용을 추가 합니다.

```
pre_hook = systemctl stop nginx
post_hook = systemctl start nginx
```

Save the file by pressing Ctrl + X and entering Y when prompted.

The standalone option of Certbot uses its web server to create the certificate that doesn't work with Nginx. The pre_hook and post_hook commands run before and after the renewal to automatically shut and restart the Nginx server without manual intervention.

#### 인증서 연장하기

Do a dry run of the SSL renewal process to ensure it works.

```
$ sudo certbot renew --dry-run
```


## step7. Nginx 설정 변경

### nginx.conf 수정
Open the file nginx.conf for editing.

```
$ sudo nano /etc/nginx/nginx.conf
```

Find the line include /etc/nginx/conf.d/*.conf; and paste the following code below it.

```
server_names_hash_bucket_size  64;
```

Save the file by pressing CTRL+X, then Y.

### 가상도메인
Create the Moodle configuration file for Nginx and open it for editing.

```
$ sudo nano /etc/nginx/conf.d/moodle.conf
```

Paste the following code in it.

#### Redirect HTTP to HTTPS
```
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
```

Save the file by pressing CTRL+X, then Y.

### 설정확인 및 실행하기
Verify Nginx configuration syntax.

```
$ sudo nginx -t
```

Restart the Nginx service.

```
$ sudo systemctl restart nginx
```
