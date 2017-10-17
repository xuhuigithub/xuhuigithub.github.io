---
layout: post
title: Centos 安装bind9.3.1使用mariadb存储域名
date: 2017-10-17 17:45:56 +0800
description: bind使用mysql作为后端存储域名
img: i-rest.jpg # Add image post (optional)
tags: [bind,mysql]
---
## 环境:
    - CentOS Linux release 7.2.1511 (Core)
    - 3.10.0-327.el7.x86_64

    - BIND 9.3.1

    - mysql  Ver 15.1 Distrib 5.5.56-MariaDB, for Linux (x86_64) using readline 5.1(yum install)

    -  mysql-bind-0-1.tgz 附加软件包，bind连接mysql的SDB驱动，需要跟随bind一起编译，下载地址： https://sourceforge.net/projects/mysql-bind/
## 
```bash
wget  https://ftp.isc.org/isc/bind/9.3.1/bind-9.3.1.tar.gz
tar zxf bind-9.3.1.tar.gz
chown -R root:root bind-9.3.1

tar -xzf mysql-bind-0-1.tgz

cd mysql-bind-0.1/

cp -a mysqldb.c ../bind-9.3.1/bin/named/

cp -a mysqldb.h ../bind-9.3.1/bin/named/include/
cd ../bind-9.3.1/

yum install mysql-devel -y #安装mysql lib include库

vim bin/named/Makefile.in
#将
DBDRIVER_OBJS =

DBDRIVER_SRCS =

DBDRIVER_INCLUDES =

DBDRIVER_LIBS =

#修改为
DBDRIVER_OBJS = mysqldb.@O@

DBDRIVER_SRCS = mysqldb.c

DBDRIVER_INCLUDES = -I'/usr/include/mysql'

DBDRIVER_LIBS = -L'/usr/lib64/mysql/' -lmysqlclient -lz -lcrypt -lnsl -lm -lc -lnss_files -lnss_dns -lresolv -lc -lnss_files -lnss_dns - lresolv


vim bind-9.9.11/bin/named
#添加
#include "mysqldb.h"

找到/*
* Add calls to register sdb drivers here.
*/
/* xxdb_init(); */
#添加
mysqldb_init();
找到
/*
* Add calls to unregister sdb drivers here.
*/
/* xxdb_clear(); */
#添加 
mysqldb_clear();



#开始编译
./configure --prefix=/usr/local/named --sysconfdir=/etc/named/ --disable-ipv6 --disable-chroot --enable-threads --libdir=/var/lib/named

make
make install
```
`如果make发现问题，先检查上边的步骤然后再重新 configure再make试试 `


## 生成named 配置文件
```bash
cd /usr/local/named

mkdir etc

cd etc/
../sbin/rndc-confgen > rndc.conf

tail -n 10 rndc.conf | head -9 | sed 's/#/ /g' > named.conf
cat named.conf 
#建立localhost.zone文件

vi localhost.zone

$TTL    86400

$ORIGIN localhost.

@                       1D IN SOA       @ root (

                                       42              ; serial (d. adams)

                                       3H              ; refresh

                                       15M             ; retry

                                       1W              ; expiry

                                       1D )            ; minimum



                       1D IN NS        @

                       1D IN A         127.0.0.1

#建立named.local文件
vi named.local

$TTL    86400

@       IN      SOA     localhost. root.localhost.  (

                                     1997022700 ; Serial

                                     28800      ; Refresh

                                     14400      ; Retry

                                     3600000    ; Expire

                                     86400 )    ; Minimum

             IN      NS      localhost.



1       IN      PTR     localhost.

```



```bash
#建立根域文件named.ca  


; <<>> DiG 9.9.2-P1-RedHat-9.9.2-6.P1.fc18 <<>> +bufsize=1200 +norec @a.root-servers.net

; (2 servers found)

;; global options: +cmd

;; Got answer:

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25828

;; flags: qr aa; QUERY: 1, ANSWER: 13, AUTHORITY: 0, ADDITIONAL: 23



;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 512

;; QUESTION SECTION:

;.                IN    NS



;; ANSWER SECTION:

.            518400    IN    NS    a.root-servers.net.

.            518400    IN    NS    b.root-servers.net.

.            518400    IN    NS    c.root-servers.net.

.            518400    IN    NS    d.root-servers.net.

.            518400    IN    NS    e.root-servers.net.

.            518400    IN    NS    f.root-servers.net.

.            518400    IN    NS    g.root-servers.net.

.            518400    IN    NS    h.root-servers.net.

.            518400    IN    NS    i.root-servers.net.

.            518400    IN    NS    j.root-servers.net.

.            518400    IN    NS    k.root-servers.net.

.            518400    IN    NS    l.root-servers.net.

.            518400    IN    NS    m.root-servers.net.



;; ADDITIONAL SECTION:

a.root-servers.net.    3600000    IN    A    198.41.0.4

a.root-servers.net.    3600000    IN    AAAA    2001:503:ba3e::2:30

b.root-servers.net.    3600000    IN    A    192.228.79.201

c.root-servers.net.    3600000    IN    A    192.33.4.12

d.root-servers.net.    3600000    IN    A    199.7.91.13

d.root-servers.net.    3600000    IN    AAAA    2001:500:2d::d

e.root-servers.net.    3600000    IN    A    192.203.230.10

f.root-servers.net.    3600000    IN    A    192.5.5.241

f.root-servers.net.    3600000    IN    AAAA    2001:500:2f::f

g.root-servers.net.    3600000    IN    A    192.112.36.4

h.root-servers.net.    3600000    IN    A    128.63.2.53

h.root-servers.net.    3600000    IN    AAAA    2001:500:1::803f:235

i.root-servers.net.    3600000    IN    A    192.36.148.17

i.root-servers.net.    3600000    IN    AAAA    2001:7fe::53

j.root-servers.net.    3600000    IN    A    192.58.128.30

j.root-servers.net.    3600000    IN    AAAA    2001:503:c27::2:30

k.root-servers.net.    3600000    IN    A    193.0.14.129

k.root-servers.net.    3600000    IN    AAAA    2001:7fd::1

l.root-servers.net.    3600000    IN    A    199.7.83.42

l.root-servers.net.    3600000    IN    AAAA    2001:500:3::42

m.root-servers.net.    3600000    IN    A    202.12.27.33

m.root-servers.net.    3600000    IN    AAAA    2001:dc3::35



;; Query time: 78 msec

;; SERVER: 198.41.0.4#53(198.41.0.4)

;; WHEN: Mon Jan 28 15:33:31 2013

;; MSG SIZE  rcvd: 699
```


## 最终named.conf，更多参数请参考yum install bind后生成的/etc/named.conf
```bash
key "rndc-key" {

      algorithm hmac-md5;

      secret "GBO+RH2CgwAjNpucN6fCiw==";

};

controls {

      inet 127.0.0.1 port 953

          allow { 127.0.0.1; } keys { "rndc-key"; };

  };

options {
        listen-on port 53 { any; };

        listen-on-v6 port 53 { ::1; };

        allow-query     { any; };

        allow-transfer  { none; };

        recursion yes;

        allow-recursion  { any; };    //允许递归查询

};



zone "." IN {

        type hint;

        file "/usr/local/named/etc/named.ca";

};



zone "localhost" IN {

        type master;

        file "/usr/local/named/etc/localhost.zone";

        allow-update { none; };

};



zone "0.0.127.in-addr.arpa" IN {

        type master;

        file "/usr/local/named/etc/named.local";

        allow-update { none; };

};



zone "mydomain.com" {

  type master;

  database "mysqldb dnsdb mydomain localhost root 123456";

    //database 参数解释：
    // 第一个mysqldb写死应该是指的驱动，第二个谢数据库名，第三个写表名，第四个MYSQL主机地址，第五个MYSQL用户，第六个MYSQL用户密码

};


zone "19.202.220.in-addr.arpa" {

  type master;

  database "mysqldb dnsdb ptr localhost root 123456";

};



```


## 建立数据库表结构
### 正向解析
```sql
CREATE TABLE `xuhui` (

  name varchar(255) default NULL,

  ttl int(11) default NULL,

  rdtype varchar(255) default NULL,

  rdata varchar(255) default NULL

) ;

```
插入测试数据
```sql
INSERT INTO xuhui VALUES ('xuhui.local', 259200, 'SOA', 'xuhui.local. xuhui.xuhui.local. 201710131 28800 7200 86400 28800');

INSERT INTO xuhui VALUES ('xuhui.local', 259200, 'NS', 'dns1.xuhui.local.');

INSERT INTO xuhui VALUES ('xuhui.local', 259200, 'NS', 'dns2.xuhui.local.');

INSERT INTO xuhui VALUES ('xuhui.local', 259200, 'MX', '10 mail.xuhui.local.');

INSERT INTO xuhui VALUES ('dns1.xuhui.local', 259200, 'A', '192.168.61.188');

INSERT INTO xuhui VALUES ('dns2.xuhui.local', 259200, 'A', '192.168.61.9');

INSERT INTO xuhui VALUES ('dashboard.xuhui.local', 259200, 'A', '192.168.61.138');

INSERT INTO xuhui VALUES ('dashboard.xuhui.local', 259200, 'A', '192.168.61.139');

```
### 反向解析
```sql
CREATE TABLE ptr (

  name varchar(255) default NULL,

  ttl int(11) default NULL,

  rdtype varchar(255) default NULL,

  rdata varchar(255) default NULL

) ;

```
插入测试数据
```sql
INSERT INTO `ptr` VALUES ('168.192.in-addr.arpa', 17600, 'SOA', 'xuhui.local. xuhui.xuhui.local. 201710131 28800 7200 86400 28800');

INSERT INTO `ptr` VALUES ('168.192.in-addr.arpa', 17600, 'NS', 'dns1.xuhui.local.');

INSERT INTO `ptr` VALUES ('68.192.in-addr.arpa', 17600, 'NS', 'dns2.xuhui.local.');

INSERT INTO `ptr` VALUES ('138.61.168.162.in-addr.arpa', 17600, 'PTR', 'dashboard.xuhui.local.');

INSERT INTO `ptr` VALUES ('188.61.168.192.in-addr.arpa', 17600, 'NS', 'dns1.xuhui.local.');

```
## 测试
启动bind
```
/usr/local/named/sbin/named -c /usr/local/named/etc/named.conf

```
测试解析
```bash
nslookup  www.mydomain.com  192.168.61.9
#192.168.61.9是你的dns地址
```
