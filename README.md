# 미니 프로젝트
<br><br>


# 1. HTTP, WP, MySQL 한 서버에서 연결
<br><br>


## 1-1. 기본 도구 설치
<br>

### 1) Apache 웹 서버(httpd) 설치
```
yum install -y httpd
```
### 2) WordPress 최신 버전 압축 파일 다운로드
curl / -o 다운도르 시 저장할 이름 / 워드프레스 최신 버전 압축파일 다운로드 url
```
curl -o wordpress.tar.gz https://wordpress.org/latest.tar.gz
```
### 3) MySQL 클라이언트 설치
데이터베이스에 접속하고 쿼리를 날릴 수 있는 도구
```
yum install -y mysql
```
### 4) MySQL-Server 설치
실제 데이터 저장과 쿼리 처리를 담당하는 DB 서버 프로그램
```
yum install -y mysql-server
```
### 5) php 설치
워드 프레스는 PHP로 작성되어있어서 php를 설치가 필수
```
yum install -y php
```
### PHP-MySQLND
MySQL Native Driver를 기반으로하는 PHP 확장도구. PHP에서 MySQL 서버에 직접 연결할 수 있게 해줌
```
yum install php-mysqlnd
```
<br><br>


## 2-1 Apache, WP 설정
<br>

### 1) Apache 웹 서버
웹 서버 시작 및 방화벽 허용
```
systemctl start httpd
```
```
firewall-cmd --add-service=http
```
### 2) 워드프레스 압축해제
```
tar xvf wordpress.tar.gz -C /var/www/html
```
```
ls /var/www/html # 확인용
```
### 3) 워드프레스 설정파일에서 db정보 입력
WordPress 압축을 해제하면 `wp-config-sample.php` 파일이 있
이를 복사해 실제 설정 파일인 `wp-config.php`로 만든 후, 아래와 같이 DB 정보를 입력
```
cp /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php
```
```
// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wp' );

/** Database username */
define( 'DB_USER', 'wp-user' );

/** Database password */
define( 'DB_PASSWORD', 'P@ssw0rd' );

/** Database hostname */
define( 'DB_HOST', '192.168.56.44' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```
### 4) 가상 호스트 설정
```
vi /etc/httpd/conf.d/wordpress.conf
```
```
<VirtualHost *:80>
        ServerName example.com
        DocumentRoot    /var/www/html/wordpress

        <Directory      "/var/www/html/wordpress">
                AllowOverride All
        </Directory>

</Virtualhost>
```
### 5) 설정 적용
```
systemctl restart httpd
```
<br><br>


## 3.1 db설정
<br>

### 1) MySQLD 시작
```
systemctl start mysqld
```
### 2) Apache의 외부 DB 접속 허용
```
setsebool -P httpd_can_network_connect_db 1
```
### 3) mysql 안에 유저 데이터 삽입
```
create database wp;
```
```
create user 'wp-user'@'192.168.56.44' identified by 'P@ssw0rd';
```
```
grant all privileges on wp.* to 'wp-user'@'192.168.56.44';
```
```
exit
```


<br><br>


# 2. MySQL을 다른 서버로 분리
<br><br>


## 2-1 원래 서버에 있던 MySQL삭제 및 새로운 db서버 설치 (웹 서버)
<br>

### 1) 새로운 db서버 init파일
```
# -*- mode: ruby -*-
# vi: set ft=ruby :
vm_image = "nobreak-labs/rocky-9"
vm_subnet = "192.168.57."

Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define "db1" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "db1"
      vb.cpus = 2
      vb.memory = 2048
    end

    node.vm.network "private_network", ip: vm_subnet + "13", nic_type: "virtio"
    node.vm.hostname = "db1"
  end

  config.vm.define "db2" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "db2"
      vb.cpus = 2
      vb.memory = 2048
    end

    node.vm.network "private_network", ip: vm_subnet + "14", nic_type: "virtio"
    node.vm.hostname = "db2"
  end
end

```
### 2) 기존 서버 db삭제
```
systemctl stop mysqld
```
```
yum remove mysql mysql-server
```
### 3) MySQL, MySQL-Server 설치
데이터베이스에 접속하고 쿼리를 날릴 수 있는 도구
```
yum install -y mysql mysql-server
```
<br><br>


## 2-2 새로운 db서버 MySQL설정 (db서버 - db1)
<br>

### 1) MySQL, MySQL-Server 설치
데이터베이스에 접속하고 쿼리를 날릴 수 있는 도구
```
yum install -y mysql mysql-server
```
### 2) MySQL 시작
```
systemctl start mysqld
```
### 3) MySQL 안에 유저 데이터 삽입
```
create database wp;
```
```
CREATE USER 'wp-user'@'192.168.57.1' IDENTIFIED BY 'P@ssw0rd';
```
```
GRANT ALL PRIVILEGES ON wp.* TO 'wp-user'@'192.168.57.1';
```
```
FLUSH PRIVILEGES;
```
```
exit
```
### 4) sql 방화벽 허용
firewall-cmd --permanent --add-service=mysql

## 2-3 웹 서버 워드프레스 설정 (웹 서버)
<br>

### 1) 워드 프레스 설정 변경
```
vi /var/www/html/wordpress/wp-config.php
```
```
// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wp' );

/** Database username */
define( 'DB_USER', 'wp-user' );

/** Database password */
define( 'DB_PASSWORD', 'P@ssw0rd' );

/** Database hostname */
define( 'DB_HOST', '192.168.57.13' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```
<br><br>


# 3. HTTP, WP, MySQL 한 서버에서 연결
<br><br>
