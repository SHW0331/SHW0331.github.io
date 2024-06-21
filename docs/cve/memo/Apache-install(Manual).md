

# Apache install
Apache 2.4.58v 수동 설치
{: .fs-6 .fw-300 }

- Ubuntu Version : 22.0.4

## Apache 수동 설치 (*관리자 권한)
## 관련 Apache 모듈
- mod_http2
- nghttp2 lib
- mod_ssl


## 필수 라이브러리 설치
apt update
apt upgrade

apt install -y build-essential libpcre3 libpcre3-dev libssl-dev libexpat1-dev libtool binutils libnghttp2-dev libapr1-dev libaprutil1-dev

# 환경 변수
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
export CPPFLAGS="-I/usr/local/include"
export LDFLAGS="-L/usr/local/lib"

## Apache 2.4.58 다운로드
cd /usr/local/src
wget https://archive.apache.org/dist/httpd/httpd-2.4.58.tar.gz
tar -xvf httpd-2.4.58.tar.gz
cd httpd-2.4.58


## APR 모듈 다운로드
cd /usr/local/src
wget https://dlcdn.apache.org//apr/apr-1.7.4.tar.gz
wget https://dlcdn.apache.org//apr/apr-util-1.6.3.tar.gz
tar -xvf apr-1.7.4.tar.gz
tar -xvf apr-util-1.6.3.tar.gz
mv apr-1.7.4 /usr/local/src/httpd-2.4.58/srclib/apr
mv apr-util-1.6.3 /usr/local/src/httpd-2.4.58/srclib/apr-util

## nghttp2 모듈 다운로드
apt install -y python3-dev
wget https://github.com/nghttp2/nghttp2/releases/download/v1.47.0/nghttp2-1.47.0.tar.gz
tar -xzvf nghttp2-1.47.0.tar.gz
cd nghttp2-1.47.0
./configure
make
make install
<!-- mv nghttp2-1.47.0 /usr/local/src/httpd-2.4.58/srclib/nghttp2 -->

## Apache 컴파일 및 설치
cd /usr/local/src/httpd-2.4.58
./configure --enable-so --enable-ssl --with-ssl --enable-http2 --with-included-apr --with-mpm=event --with-pcre --with-included-nghttp2
make
make install

/usr/local/apache2/bin/httpd -v
/usr/local/apache2/bin/apachectl start
/usr/local/apache2/bin/apachectl status

## Service 설정
vi /etc/systemd/system/httpd.service

```ini
[Unit]
Description=Apache HTTP Server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/apache2/bin/apachectl start
ExecStop=/usr/local/apache2/bin/apachectl stop
ExecReload=/usr/local/apache2/bin/apachectl graceful

[Install]
WantedBy=multi-user.target
```

systemctl daemon-reload
systemctl enable httpd
systemctl start httpd

## httpd.conf 설정
vi /usr/local/apache2/conf/httpd.conf
- 추가
LoadModule mpm_event_module modules/mod_mpm_event.so
LoadModule ssl_module modules/mod_ssl.so
LoadModule http2_module modules/mod_http2.so
LoadModule socache_shmcb_module modules/mod_socache_shmcb.so

LoadModule mpm_event_module modules/mod_mpm_event.so
LoadModule authn_core_module modules/mod_authn_core.so
LoadModule authz_core_module modules/mod_authz_core.so
LoadModule authz_host_module modules/mod_authz_host.so
LoadModule authz_user_module modules/mod_authz_user.so
LoadModule access_compat_module modules/mod_access_compat.so
LoadModule auth_basic_module modules/mod_auth_basic.so
LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
LoadModule reqtimeout_module modules/mod_reqtimeout.so
LoadModule filter_module modules/mod_filter.so
LoadModule mime_module modules/mod_mime.so
LoadModule log_config_module modules/mod_log_config.so
LoadModule env_module modules/mod_env.so
LoadModule headers_module modules/mod_headers.so
LoadModule setenvif_module modules/mod_setenvif.so
LoadModule ssl_module modules/mod_ssl.so
LoadModule http2_module modules/mod_http2.so
LoadModule unixd_module modules/mod_unixd.so
LoadModule status_module modules/mod_status.so
LoadModule autoindex_module modules/mod_autoindex.so


- 추가 : Inlcude conf/extra/httpd-ssl.conf (주석 제거)

## httpd-ssl.conf find
sudo find /usr/local/apache2 -name httpd-ssl.conf

## 방화벽 open
ufw allow 443/tcp
ufw allow 80/tcp

apt install nmap
nmap -p 443 your_server_ip


## 확인
sudo /usr/local/apache2/bin/apachectl configtest

## ssl 인증서 (localhost SSL 인증서)
mkdir /usr/local/apache2/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /usr/local/apache2/ssl/localhost.key -out /usr/local/apache2/ssl/localhost.crt

vi /usr/local/apache2/conf/extra/httpd-ssl.conf
SSLCertificateFile "/usr/local/apache2/ssl/localhost.crt"
SSLCertificateKeyFile "/usr/local/apache2/ssl/localhost.key"

## port forwarding
80, 443 open

## 모니터링
apt install htop
watch -n 0.5 'ps -C httpd -o pid,%cpu,%mem,cmd'

.\cve-2024-27316 -t 192.168.56.1:3395 -p https -i 8192