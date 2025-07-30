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
curl / 다운도르 시 저장할 이름 / 워드프레스 최신 버전 압축파일 다운로드 url
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
