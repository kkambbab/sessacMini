# 미니 프로젝트
<br><br>


## 초기 vagrant 파일
```
# -*- mode: ruby -*-
# vi: set ft=ruby :
vm_image = "nobreak-labs/rocky-9"
vm_subnet = "192.168."

Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define "web1" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "web1"
      vb.cpus = 2
      vb.memory = 2048
    end

    node.vm.network "private_network", ip: vm_subnet + "56.44", nic_type: "virtio"
    node.vm.hostname = "web1"
  end

  config.vm.define "web2" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "web2"
      vb.cpus = 2
      vb.memory = 2048
    end

    node.vm.network "private_network", ip: vm_subnet + "56.45", nic_type: "virtio"
    node.vm.hostname = "web2"
  end
end
```

<br><br>


---
# 1. HTTP, WP, MySQL 한 서버에서 연결
<br><br>


## 1-1. 기본 도구 설치
<br>

### 1) Apache 웹 서버(httpd) 설치
```
sudo yum install -y httpd
```
### 2) WordPress 최신 버전 압축 파일 다운로드
curl / -o 다운도르 시 저장할 이름 / 워드프레스 최신 버전 압축파일 다운로드 url
```
curl -o wordpress.tar.gz https://wordpress.org/latest.tar.gz
```
### 3) MySQL 클라이언트 설치
데이터베이스에 접속하고 쿼리를 날릴 수 있는 도구
```
sudo yum install -y mysql
```
### 4) MySQL-Server 설치
실제 데이터 저장과 쿼리 처리를 담당하는 DB 서버 프로그램
```
sudo yum install -y mysql-server
```
### 5) php 설치
워드 프레스는 PHP로 작성되어있어서 php를 설치가 필수
```
sudo yum install -y php
```
### 6) PHP-MySQLND
MySQL Native Driver를 기반으로하는 PHP 확장도구. PHP에서 MySQL 서버에 직접 연결할 수 있게 해줌
```
sudo yum install -y php-mysqlnd
```
<br><br>


## 1-2 Apache, WP 설정
<br>

### 1) Apache 웹 서버
웹 서버 시작 및 방화벽 허용
```
sudo systemctl start httpd
```
```
sudo firewall-cmd --add-service=http
```
### 2) 워드프레스 압축해제
```
sudo tar xvf wordpress.tar.gz -C /var/www/html
```
```
ls /var/www/html # 확인용
```
### 3) 워드프레스 설정파일에서 db정보 입력
WordPress 압축을 해제하면 `wp-config-sample.php` 파일이 있
이를 복사해 실제 설정 파일인 `wp-config.php`로 만든 후, 아래와 같이 DB 정보를 입력
```
sudo cp /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php
```
```
sudo vi /var/www/html/wordpress/wp-config.php
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
sudo vi /etc/httpd/conf.d/wordpress.conf
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
sudo systemctl restart httpd
```
### 6) sebool http와 db 연결 설정
```
sudo setsebool -P httpd_can_network_connect_db 1
```
<br><br>


## 1-3 db설정
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


---
# 2. MySQL을 다른 서버로 분리
<br><br>


## 2-1 원래 서버에 있던 MySQL삭제 및 새로운 db서버 설치 (웹 서버)
<br>

### 1) 새로운 db서버 init파일
```
# -*- mode: ruby -*-
# vi: set ft=ruby :
vm_image = "nobreak-labs/rocky-9"
vm_subnet = "192.168."

Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define "db1" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "db1"
      vb.cpus = 2
      vb.memory = 2048
    end

    node.vm.network "private_network", ip: vm_subnet + "57.13", nic_type: "virtio"
    node.vm.hostname = "db1"
  end

  config.vm.define "db2" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "db2"
      vb.cpus = 2
      vb.memory = 2048
    end

    node.vm.network "private_network", ip: vm_subnet + "57.14", nic_type: "virtio"
    node.vm.hostname = "db2"
  end
end

```
### 2) 기존 서버 db삭제 (웹 서버)
```
sudo systemctl stop mysqld
```
```
sudo yum remove mysql mysql-server
```
<br><br>


## 2-2 새로운 db서버 MySQL설정 (db서버)
<br>

### 1) MySQL, MySQL-Server 설치
데이터베이스에 접속하고 쿼리를 날릴 수 있는 도구
```
sudo yum install -y mysql mysql-server
```
### 2) MySQL 시작
```
sudo systemctl start mysqld
```
```
sudo systemctl enable mysqld
```
### 3) MySQL 안에 유저 데이터 삽입
```
sudo mysql
```
```
create database wp;
```
```
CREATE USER 'wp-user'@'192.168.57.44' IDENTIFIED BY 'P@ssw0rd';
```
```
GRANT ALL PRIVILEGES ON wp.* TO 'wp-user'@'192.168.57.44';
```

[왜 57.1로 나가는가?](#왜-571인가)

```
FLUSH PRIVILEGES;
```
```
exit
```
### 4) sql 방화벽 허용
```
sudo firewall-cmd --permanent --add-service=mysql
```
```
sudo firewall-cmd --reload
```
<br><br>


## 2-3 웹 서버 워드프레스 설정 (웹 서버)
<br>

### 1) 워드 프레스 설정 변경
```
sudo vi /var/www/html/wordpress/wp-config.php
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


---
# 3. LoadBalancer 구현
vagrant파일
```
# -*- mode: ruby -*-
# vi: set ft=ruby :
vm_image = "nobreak-labs/rocky-9"
vm_subnet = "192.168."

Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define "lb" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "lb"
      vb.cpus = 2
      vb.memory = 2048
    end

    node.vm.network "private_network", ip: vm_subnet + "56.10", nic_type: "virtio"
    node.vm.hostname = "lb"
  end
end
```
<br><br>


## 3-1. 로드밸런서 설치
<br>

HAProxy와 Nginx가 있다.
HAProxy로 할 예정
```
sudo yum install -y haproxy
```
<br><br>


## 3-2. 로드밸런서 설정
<br>

```
sudo vi /etc/haproxy/haproxy.cfg
```
밑 내용 추가
```
frontend http_front
    bind *:80
    default_backend http_back

backend http_back
    balance roundrobin
    server web1 192.168.56.44:80 check
    server web2 192.168.56.45:80 check
```
<br><br>


## 3-3. 로드밸런서 서비스 시작
<br>

```
sudo systemctl enable haproxy
```
```
sudo systemctl start haproxy
```
<br><br>


## 3-4. 로그 설정
<br>

```
sudo vi /etc/rsyslog.d/haproxy.conf
```
```
# HAProxy log 수신 허용
module(load="imudp")
input(type="imudp" port="514")

# HAProxy 로그를 별도 파일로 저장
local0.*   /var/log/haproxy.log
```
rsyslog 재시작
```
sudo systemctl restart rsyslog
```
```
sudo vi /etc/haproxy/haproxy.cfg
```
```
global
    log 127.0.0.1 local0
    log 127.0.0.1 local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000
```
option httplog 이 없으면 로깅이 작동하지 않음
HAProxy 재시작
```
sudo systemctl restart haproxy
```
로그확인
```
sudo tail -f /var/log/haproxy.log
```
<br><br>


## 3-4. 포트 80으로 방화벽 열어주기
<br>

```
sudo firewall-cmd --permanent --add-port=80/tcp
```
```
sudo firewall-cmd --reload
```
<br><br>


## 3-4. 웹서버 (web2 설정)
<br>

### 1) Apache 웹 서버(httpd) 설치
```
sudo yum install -y httpd
```
### 2) WordPress 최신 버전 압축 파일 다운로드
curl / -o 다운도르 시 저장할 이름 / 워드프레스 최신 버전 압축파일 다운로드 url
```
curl -o wordpress.tar.gz https://wordpress.org/latest.tar.gz
```
### 3) php 설치
워드 프레스는 PHP로 작성되어있어서 php를 설치가 필수
```
sudo yum install -y php
```
### 4) PHP-MySQLND
MySQL Native Driver를 기반으로하는 PHP 확장도구. PHP에서 MySQL 서버에 직접 연결할 수 있게 해줌
```
sudo yum install -y php-mysqlnd
```
<br><br>


## 3-5 Apache, WP 설정
<br>

### 1) Apache 웹 서버
웹 서버 시작 및 방화벽 허용
```
sudo systemctl start httpd
```
```
sudo firewall-cmd --add-service=http
```
### 2) 워드프레스 압축해제
```
sudo tar xvf wordpress.tar.gz -C /var/www/html
```
```
ls /var/www/html # 확인용
```
### 3) 워드프레스 설정파일에서 db정보 입력
WordPress 압축을 해제하면 `wp-config-sample.php` 파일이 있
이를 복사해 실제 설정 파일인 `wp-config.php`로 만든 후, 아래와 같이 DB 정보를 입력
```
sudo cp /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php
```
```
sudo vi /var/www/html/wordpress/wp-config.php
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
### 4) 가상 호스트 설정
```
sudo vi /etc/httpd/conf.d/wordpress.conf
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
sudo systemctl restart httpd
```
### 6) sebool http와 db 연결 설정
```
sudo setsebool -P httpd_can_network_connect_db 1
```
<br><br>


## 3-6 Apache, WP 설정
<br>



<br><br>























































## 왜 57.1인가?
[root@server ~]# telnet 192.168.57.13 3306
Connected to 192.168.57.13.
Escape character is '^]'.
EHost '192.168.57.1' is not allowed to connect to this MySQL serverConnection closed by foreign host.

웹 서버는 192.168.57.1로 접속을 시도하고있다.

[root@server ~]# ip route get 192.168.57.13
192.168.57.13 via 10.0.2.2 dev enp0s3 src 10.0.2.15 uid 0
    cache

ip route get 192.168.57.13 결과를 보면,

192.168.57.13 via 10.0.2.2 dev enp0s3 src 10.0.2.15 uid 0
이 뜻은:

192.168.57.13 (DB서버)로 가는 경로가

게이트웨이 10.0.2.2를 통해

enp0s3 인터페이스로 나가고

출발지 IP는 10.0.2.15로 설정된다는 것

즉, 웹서버에서 DB서버로 가는 트래픽이 **10.0.2.0/24 네트워크(호스트 전용 네트워크)**를 거치고 있습니다.

이게 의미하는 점
웹서버에서 DB서버로 연결 시도할 때 실제 출발 IP가 192.168.56.44가 아니라
10.0.2.15 혹은 192.168.57.1 같은 가상 네트워크 IP로 변환되어 MySQL에 도달합니다.

따라서 MySQL에서 허용하는 IP도 10.0.2.15 또는 192.168.57.1 같은 IP로 등록해야 정상 연결 가능.

즉, **enp0s3 (NAT 네트워크)** 를 통해 나가므로 출발 IP는 10.0.2.15

그런데 접속 로그엔 왜 192.168.57.1이라고 보일까?
그건 다음과 같은 경우입니다:

🔍 현재 상황의 원인 요약
1. 웹서버 IP: 192.168.56.44
이는 enp0s8 등 호스트 전용 네트워크 어댑터에 연결된 주소입니다.

이 NIC는 56대역 (예: 192.168.56.0/24)에 속해 있음.

2. DB서버 IP: 192.168.57.13
enp0s8 또는 enp0s9에 설정된 주소이며, 57대역 (192.168.57.0/24)에 있음.

DB서버 입장에서 보면 56대역 NIC가 없음!

그래서 56 네트워크로부터 직접 패킷을 받을 방법이 없음.

📌 그래서 어떤 일이 일어나냐면?
🛣 경로 흐름 (추론):
웹서버 (192.168.56.44) → DB서버 (192.168.57.13)로 통신 시도

VirtualBox의 호스트 머신(Windows)이 두 네트워크 모두에 속함

56대역: 192.168.56.1

57대역: 192.168.57.1

그래서 실제 경로는 이렇게 됨:

scss
복사
편집
웹서버 → VirtualBox 호스트 (56.1) → VirtualBox 호스트 (57.1) → DB서버 (57.13)
이 때 DB서버에서 보면 접속 출발지는 192.168.57.1 (호스트) 으로 보임

즉, 웹서버 IP가 아니고, 중간 릴레이인 호스트의 IP

🎯 결론: DB서버에 56대역 NIC가 없다 보니
VirtualBox가 알아서 호스트 머신을 릴레이처럼 사용함

그 결과 DB서버에서 **실제 요청 IP는 192.168.57.1**로 보임

이건 NAT처럼 동작하는 셈
