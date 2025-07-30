# 미니 프로젝트
# 1. HTTP, WP, MySQL 한 서버에서 연결
## 1-1. HTTP, WP, MySQL설치
### httpd 설치
```
yum install -y httpd
```
### 워드 프레스 압축파일 가져오기
curl / 다운도르 시 저장할 이름 / 워드프레스 최신 버전 압축파일 다운로드 url
```
curl -o wordpress.tar.gz https://wordpress.org/latest.tar.gz
```
### MySQL 설치
클라이언트 도구
```
yum install -y mysql
```
### MySQL-Server 설치
MySQL 서버 데몬
```
yum install -y mysql-server
```
