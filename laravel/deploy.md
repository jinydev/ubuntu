# 라라벨 애플리케이션 배포
라라벨을 배포하기 위한 몇가지 단계를 설정합니다.

## 컴포저 설치
PHP언어로 개발된 라라벨은 composer를 통하여 패키지를 관리합니다. 이를 위해서 먼저 컴포저를 설치합니다.
컴포저를 루트로 설치후, 실행할 수 있습니다. 하지만, 컴포저를 셀프로 업데이트 할때 루트 권한이 같이 필요하기 때문에 불편합니다.
이를 개선하기 위하여 사용자 계정으로 컴포저를 설치하여 사용합니다.


### 사용자 계정에 컴포저 설치하기

애플리케이션을 변경 및 배포하는 `jiny` 계정을 생성합니다.

```
# adduser jiny
```
생성된 jiny 계정을 웹 서버의 계정 그룹인 `www-data` 그룹에 추가 합니다.

```
# usermod -a -G www-data jiny
```

이제 애플리케이션을 배포할 디렉토리를 생성합니다.

```
# mkdir /var/www/laravel
```

이제 jiny 계정이 생성한 폴더에 애플리케이션을 배포할 수 있도록 소유자를 변경합니다.

```
# chown -R jiny:jiny /var/www/jiny/laravel/
```

우이제 루트로 실행할 작업은 완료되었고 애플리케이션을 배포하기 위해 jiny 계정으로 로그인합니다.



사용자 실행파일을 `bin` 폴더 안에 넣어 사용할 예정입니다.
컴포저를 설치할 사용자 계정 밑에 폴더를 생성합니다. 

```
$ mkdir ~/bin
```

`bin`파일에 있는 파일이 실행될 수 있도록 path 설정을 추가합니다.
 

```
$ nano ~/.profile
```

`~/.profile` 파일 맨 아래에 다음 내용을 추가줍니다.

```
export PATH=$PATH:$HOME/bin
```

추가한 설정 내용이 현재 세션에 반영 되도록 `source` 명령어을 실행합니다.

```
$ source ~/.profile
```
 
### 컴포저 설치
사용자 계정에 컴포저를 설치합니다.

```
$ curl -sS https://getcomposer.org/installer | php -- --install-dir=$HOME/bin/
```

컴포저 공식사이트에서 `composer.phar` 파일을 다운로드 받습니다.
`bin` 폴더 안에 파일이 다운로드 되었는지 확인합니다.
```
jinyerp@jinytms:~$ ls ./bin/
composer.phar
```

사용이 편리하도록 composer 라는 이름으로 심볼릭 링크를 겁니다.

```
$ ln -s $HOME/bin/composer.phar $HOME/bin/composer
```

컴포저가 잘 설치되었는지 확인해 봅니다.

```
jinyerp@jinytms:~$ composer --version
Composer version 2.4.2 2022-09-14 16:11:15
```
 

## 라라벨 애플리케이션 설치
라라벨 코드를 설치합니다. 
애플리케이션을 설치할 폴더로 이동합니다.

```
$ cd /var/www/laravel/
```

깃을 통하여 개발된 소스코드를 복제합니다.

```
$ git clone git@github.com:jinyerp-src/base.git demo
```

프로젝트 폴더로 이동합니다.
```
cd demo
```

 
### 의존 패키지 설치
라라벨 코드를 복제할 경우, 의존성 있는 패키지들은 같이 설치되지 않습니다. 컴포저를 통하여 의존된 패키지를 추가로 다운로드 받아 설치해 주어야 합니다.

> 주의할 점은 이제 운영 환경이므로 개발과 다르게 접근해야 하며 개발시에만 사용되는 패키지를 설치하지 않고 오토로더를 최적화하도록 아래의 옵션을 사용합니다.

```
$ composer install --no-dev -o --prefer-dist
```


## 환경 구성
라라벨을 설치가 끝났습니다. 하지만, 이를 바로 웹서비스로 구동할 수 없습니다. 
몇개의 추가 설정이 필요합니다.

### 환경설정파일
라라벨은 설치 루트에 `.env` 설정파일이 필요합니다. 하지만, 단순한 복제로 코드를 등록한 경우 이파일은 생성되지 않습니다.
예제코드를 통하여 `.env` 파일을 생성합니다.

```
$ cp .env.example .env
```

### 키생성
세션과 데이타 암호화에 사용하는 대칭키를 생성합니다. 

```
$ php artisan key:gen
```

### 추가설정
에디터로 .env 를 열어서 APP_DEBUG 와 APP_ENV 를 설정하며 그외 DB 연결 정보와 mailgun, github 인증 정보등을 사용자의 인증 정보에 맞게 수정합니다.

```
APP_ENV=production
APP_DEBUG=false
 
SESSION_DRIVER=redis
 
MAIL_DRIVER=mailgun
MAILGUN_DOMAIN=todolog.mailgun.org
MAILGUN_SECRET=key-mailgun-secret
 
GITHUB_ID=github-app-id
GITHUB_SECRET=github-secret
GITHUB_URL=http://todolog.myhost.com/auth/github/callback
```


## DB 마이그레이션

이제 데이타베이스 마이그레이션을 실행합니다. 
`APP_ENv=production` 이므로 진짜로 실행할지를 물어보는 프롬프트가 뜨는데 yes 를 입력합니다.

```
$ php artisan migrate
```
 

이제 초기 데이타를 시딩합니다. --class 옵션으로 시딩 클래스를 명시한 이유는 프로덕션 환경이라 개발 패키지를 설치하지 않아서(컴포저의 --no-dev) faker 패키지가 없으므로 TaskTableSeeder 는 작동하지 않기 때문입니다.

```
$ php artisan db:seed --class=UserTableSeeder
```

## 스케줄러 등록
스케줄러가 구동되도록 crontab -e 로 작업 등록 기능을 실행한 후에 다음 내용을 등록합니다.

```
* * * * * php /var/www/laravel/todolog/artisan schedule:run 1>> /dev/null 2> &1
```
 

## php-fpm 디렉터리 권한 설정

php-fpm 은 www-data 라는 계정으로 구동되지만 우리는 todolog 라는 계정으로 애플리케이션을 만들었기때문에 php-fpm 이 애플리케이션 폴더에 파일 쓰기가 안 됩니다. storage 폴더와 bootstrap/cache 폴더는 php-fpm 가 쓰기 권한이 있어야 제대로 동작하므로 소유자의 그룹을 www-data 로 변경해 줍니다.

```
$ sudo chown -R www-data:www-data storage/ bootstrap/cache/
```

소유자와 그룹은 읽고 쓰기가 가능하도록 other 는 안 되도록 모드를 수정합니다.

```
$ sudo chmod -R 775 storage/* bootstrap/cache/
```

```
$ sudo usermod -a -G www-data todolog
```


모두가 읽고 쓰기가 되도록 777 로 설정하는 것은 보안에 취약하므로 권장하지 않습니다.

 

애플리케이션 구동을 위한 모든 준비가 완료되었고 브라우저로 연결하여 정상 동작하는지 확인해 봅시다.