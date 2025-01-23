# cloud config
```
#cloud-config 
hostname: app01.lab 
runcmd: 
- date > /opt/date

#cloud-config 
hostname: app02.lab 
runcmd: 
- date > /opt/date
```
# Обновляем пакеты
```bash
sudo dnf update -y  
```

# MariaDB
## установка
https://mariadb.com/kb/en/mariadb-package-repository-setup-and-usage/
```bash
sudo su
curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | bash
dnf makecache
dnf install -y mariadb-server 
```

```bash
sudo systemctl start mariadb
mysql -uroot -e 'SELECT version();'
mysql: Deprecated program name. It will be removed in a future release, use '/usr/bin/mariadb' instead
+----------------+
| version()      |
+----------------+
| 11.6.2-MariaDB |
+----------------+

sudo mysql_secure_installation
#mkdir -p /usr/local/mysql/var 
#sudo chown -R mysql:mysql /usr/local/mysql/var 
```

## Конфиги
```bash
vi /etc/my.cnf.d/server.cnf
```
app01
```bash
#
# These groups are read by MariaDB server.
# Use it for options that only the server (but not clients) should see
#

# this is read by the standalone daemon and embedded servers
[server]

# This group is only read by MariaDB servers, not by MySQL.
# If you use the same .cnf file for MySQL and MariaDB,
# you can put MariaDB-only options here
[mariadb]

# This group is read by both MariaDB and MySQL servers
[mysqld]

#data=/usr/local/mysql/var
#language=/usr/share/mariadb/english
#log-basename=mysqld
#general-log
#log-slow-queries
#
# * Galera-related settings
#
[galera]
# Mandatory settings
wsrep_on=ON
wsrep_provider=/usr/lib64/galera-4/libgalera_smm.so
wsrep_cluster_address="gcomm://192.168.0.52,192.168.0.68,192.168.0.4"
binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
#
# Allow server to accept connections on all interfaces.
#
bind-address=0.0.0.0
#
# Optional setting
#wsrep_slave_threads=1
#innodb_flush_log_at_trx_commit=0

#my galera setting
wsrep_cluster_name="cluster"
wsrep_node_address="192.168.0.52"
wsrep_node_name="app01"

# this is only for embedded server
[embedded]

# This group is only read by MariaDB servers, not by MySQL.
# If you use the same .cnf file for MySQL and MariaDB,
# you can put MariaDB-only options here
[mariadb]

# This group is only read by MariaDB-10.11 servers.
# If you use the same .cnf file for MariaDB of different versions,
# use this group for options that older servers don't understand
[mariadb-10.11]
```
app02
```bash
#
# These groups are read by MariaDB server.
# Use it for options that only the server (but not clients) should see
#

# this is read by the standalone daemon and embedded servers
[server]

# This group is only read by MariaDB servers, not by MySQL.
# If you use the same .cnf file for MySQL and MariaDB,
# you can put MariaDB-only options here
[mariadb]

# This group is read by both MariaDB and MySQL servers
[mysqld]

#data=/usr/local/mysql/var
#language=/usr/share/mariadb/english
#log-basename=mysqld
#general-log
#log-slow-queries
#
# * Galera-related settings
#
[galera]
# Mandatory settings
wsrep_on=ON
wsrep_provider=/usr/lib64/galera-4/libgalera_smm.so
wsrep_cluster_address="gcomm://192.168.0.52,192.168.0.68,192.168.0.4"
binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
#
# Allow server to accept connections on all interfaces.
#
bind-address=0.0.0.0
#
# Optional setting
#wsrep_slave_threads=1
#innodb_flush_log_at_trx_commit=0

#my galera setting
wsrep_cluster_name="cluster"
wsrep_node_address="192.168.0.68"
wsrep_node_name="app02"

# this is only for embedded server
[embedded]

# This group is only read by MariaDB servers, not by MySQL.
# If you use the same .cnf file for MySQL and MariaDB,
# you can put MariaDB-only options here
[mariadb]

# This group is only read by MariaDB-10.11 servers.
# If you use the same .cnf file for MariaDB of different versions,
# use this group for options that older servers don't understand
[mariadb-10.11]
```
### исходник /etc/my.cnf.d/mariadb-server.cnf
```bash
#
# These groups are read by MariaDB server.
# Use it for options that only the server (but not clients) should see
#

# this is read by the standalone daemon and embedded servers
[server]

# This group is only read by MariaDB servers, not by MySQL.
# If you use the same .cnf file for MySQL and MariaDB,
# you can put MariaDB-only options here
[mariadb]

# This group is read by both MariaDB and MySQL servers
[mysqld]

#
# * Galera-related settings
#
[galera]
# Mandatory settings
#wsrep_on=ON
#wsrep_provider=
#wsrep_cluster_address=
#binlog_format=row
#default_storage_engine=InnoDB
#innodb_autoinc_lock_mode=2
#
# Allow server to accept connections on all interfaces.
#
#bind-address=0.0.0.0
#
# Optional setting
#wsrep_slave_threads=1
#innodb_flush_log_at_trx_commit=0

# this is only for embedded server
[embedded]

# This group is only read by MariaDB servers, not by MySQL.
# If you use the same .cnf file for MySQL and MariaDB,
# you can put MariaDB-only options here
[mariadb]

# This group is only read by MariaDB-10.11 servers.
# If you use the same .cnf file for MariaDB of different versions,
# use this group for options that older servers don't understand
[mariadb-10.11]
```
## Запуск
```bash
sudo galera_new_cluster  # app01
sudo systemctl start mariadb  # app02
```

## тест репликации
```bash
mysql -uroot
```
```sql
USE test;
SHOW TABLES;
INSERT INTO test (id, name) VALUES ('2', 'second name');
SELECT * FROM test;
+----+-------------+
| id | name        |
+----+-------------+
|  1 | name        |
|  2 | second name |
+----+-------------+
```
Проверяем SELECT на втором сервере, что данные в таблицу сохранились 
Теперь проверяем работу кворума, принудительно вырубаем один сервак и снова обновляем таблицу
```sql
INSERT INTO test (id, name) VALUES ('3', 'qwerty');
```
включаем второй сервер и проверяем таблицу
```sql
USE test;
SELECT * FROM test;
+----+-------------+
| id | name        |
+----+-------------+
|  1 | name        |
|  2 | second name |
|  3 | qwerty      |
+----+-------------+
```

# Nextcloud 28
## подготовка среды
## установка php
```bash
app01
sudo dnf install epel-release -y
dnf -y install https://rpms.remirepo.net/enterprise/remi-release-9.2.rpm
dnf module list php
dnf module reset php
dnf module enable php:remi-8.3
dnf install php php-{bz2,ctype,curl,fpm,gd,imagick,intl,json,fileinfo,libxml,mbstring,mysqlnd,openssl,posix,session,simplexml,xmlreader,xmlwriter,zip,zlib}

php -v
PHP 8.3.16 (cli) (built: Jan 14 2025 18:25:29) (NTS gcc x86_64)
Copyright (c) The PHP Group
Zend Engine v4.3.16, Copyright (c) Zend Technologies
    with Zend OPcache v8.3.16, Copyright (c), by Zend Technologies
```
## PHP-FPM
```bash
vi /etc/php-fpm.d/www.conf
ставим nginx вместо apache 
user = nginx  
group = nginx

vi /etc/php.ini
memory_limit = 768M
date.timezone = Europe/Moscow
cgi.fixpathinfo = 0

chown -R root.nginx /var/lib/php/opcache/
chown -R root.nginx /var/lib/php/session/

systemctl restart php-fpm
systemctl enable php-fpm
```

## Nginx
```bash
dnf install nginx -y
systemctl enable --now nginx
```

```bash
vi /etc/nginx/conf.d/cloud.example.com.conf
```