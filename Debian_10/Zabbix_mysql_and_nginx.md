# Zabbix server 5.2 install with Nginx and MariaDB on Debian 10

### 1. Update the system

```bash
sudo apt update

sudo apt upgrade
```
------
### 2. Install nginx

```bash
sudo apt install nginx
```
------
### 3. Install mariaDB

```bash
sudo apt install mariadb-server

sudo systemctl enable --now mariadb

sudo systemctl status mariadb
```
------
### 4. Secure mariaDB

```bash
mysql_secure_installation
```

```
Enter current password for root (enter for none): Press the Enter
Set root password? [Y/n]: Y
New password: <Enter root DB password>
Re-enter new password: <Repeat root DB password>
Remove anonymous users? [Y/n]: Y
Disallow root login remotely? [Y/n]: Y
Remove test database and access to it? [Y/n]: Y
Reload privilege tables now? [Y/n]: Y
```
------
### 5. Create initial database

```bash
mysql -uroot -p
```

```sql
mariadb> create database zabbix character set utf8 collate utf8_bin;
mariadb> create user zabbix@localhost identified by 'password';
mariadb> grant all privileges on zabbix.* to zabbix@localhost;
mariadb> quit;
```
------
### 6. Install Zabbix repository

```bash
wget https://repo.zabbix.com/zabbix/5.2/debian/pool/main/z/zabbix-release/zabbix-release_5.2-1+debian10_all.deb

sudo dpkg -i zabbix-release_5.2-1+debian10_all.deb

sudo apt update
```
------
### 7. Install Zabbix Server, Zabbix Agent and Zabbix frontend

```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-agent
```
------
### 8. import initial schema and data to the database.

```bash
sudo zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```
------
### 9. Configure the database for Zabbix server

Edit file /etc/zabbix/zabbix_server.conf

Add the password you created for the [Zabbix database user](#5-create-initial-database).

```
DBPassword=password
```
------

### 10. Set timezone

Edit file /etc/zabbix/nginx.conf, uncomment and set 'listen' and 'server_name' directives.

```bash
# listen 80;
# server_name example.com;
```

------

Edit the /etc/nginx/sites-available/default

```bash
server {
listen 8080 default_server;        ## Change from 80 to 8080
listen [::]:8080 default_server;   ## Change from 80 to 8080 as well
```

------

Edit file /etc/zabbix/php-fpm.conf, uncomment and set the right timezone for you.

```php
; php_value[date.timezone] = Europe/Riga
```
-------

### 11. Start Zabbix server and agent processes

```bash
# Restarts the zabbix-server, zabbix-agent, nginx and php service.
sudo systemctl restart zabbix-server zabbix-agent nginx php7.3-fpm

# Enables the zabbix-server, zabbix-agent, nginx and php service to start automatically after a reboot.
sudo systemctl enable zabbix-server zabbix-agent nginx php7.3-fpm

# Gets the status of zabbix-server, zabbix-agent, apache and php service.
sudo systemctl status zabbix-server zabbix-agent nginx php7.3-fpm
```

--------

### 12. Configure Zabbix frontend

Connect to your newly installed Zabbix frontend: http://server_ip_or_name

