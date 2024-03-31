# Deployment of platform (M2PLab) for running PoT on a Raspberry Pi 

# Hardware
* Raspberry Pi 4 Model B 4GB
* Bipolar High Precision Analog-to-Digital Expansion Board (AD1263)
  
![AD1263](https://raw.githubusercontent.com/blockchainer01/Software_platform_PoT/main/Figures/ExpansionBoard.jpg)
# Software
# I. Preparatory Work

## 1. **Modify the image source:**

sudo nano /etc/apt/sources.list

deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib

deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib

sudo nano /etc/apt/sources.list.d/raspi.list

deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui

## 2. **Fix the Raspberry Pi IP address**

sudo nano /etc/dhcpcd.conf

Comment out #option ntp_servers

Finally add:

interface eth0

static ip_address=192.168.46.104/24

static routers=192.168.46.1

static domain_name_servers=192.168.0.1 8.8.8.8

# **II. Installation Instructions**

## 3. **jdk installation**

Installation package: jdk-8u301-linux-arm32-vfp-hflt.tar.gz

Unzip it: sudo tar -zxvf jdk-8u301-linux-arm32-vfp-hflt.tar.gz

Edit the configuration file and configure environment variables: sudo nano /etc/profile

export JAVA_HOME=/home/pi/NetConTop/jdk/jdk1.8.0_301

export JRE_HOME=${JAVA_HOME}/jre

export CLASSPATH=.:${JAVA_HOME}lib:${JRE_HOME}/lib:$CLASSPATH

export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin

export PATH=$PATH:${JAVA_PATH}

## 4. **php environment configuration**

install  PHP7.3：（https://blog.csdn.net/jdyanghang/article/details/102782780）

sudo apt install -y -t buster php7.3-fpm php7.3-curl php7.3-gd php7.3-intl php7.3-mbstring php7.3-mysql php7.3-imap php7.3-opcache php7.3-sqlite3 php7.3-xml php7.3-xmlrpc php7.3-zip

Configure /etc/php/7.3/fpm/pool.d, add the conf configuration file, and configure it

sudo chmod 777 /etc/php/7.3/fpm/pool.d/www.conf

Monitor changes:

Uncomment: listen.allowed_clients = 127.0.0.1

## 5. **nginx configuration**

Install nginx: sudo apt-get install nginx

nginx configuration file modification: sudo chmod 777 /etc/nginx/sites-available/default

Replace the contents

## 6. **Tomcat installation**

Installation package: apache-tomcat-8.5.32.tar.gz

Unzip the installation package: sudo tar -zxvf apache-tomcat-8.5.32.tar.gz

There will be problems when starting directly: The Apache Tomcat Native library which allows using OpenSSL was not found on the java.library.path

Cause of the problem: Missing tomcat-native library

Install tomcat-native and apr, apr-iconv, apr-util

First sudo chmod -R 777 /usr/local/src, then apr-1.6.5.tar.gz, apr-iconv-1.2.2.tar.gz, apr-util-1.6.1.tar.gz Copy the installation package to the /usr/local/src folder and decompress it first sudo tar xf apr-1.6.5.tar.gz; sudo tar xf apr-iconv-1.2.2.tar.gz; sudo tar xf apr-util-1.6.1.tar.gz

cd apr-1.6.5 && sudo ./configure --prefix=/usr/local/apr

sudo make && sudo make install

cd apr-iconv-1.2.2/ && sudo ./configure --with-apr=/usr/local/apr --prefix=/usr/local/apr-iconv

sudo make && sudo make install

cd apr-util-1.6.1/ && sudo ./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr --with-apr-iconv=/usr/local/apr-iconv/bin/apriconv

sudo make && sudo make install

Modify the configuration file: sudo nano /etc/profile

Add in the last line:

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/apr/lib:/home/pi/NetConTop/tomcat/apache-tomcat-8.5.32/lib

tomcat-native installation:

cd /home/pi/NetConTop/tomcat/apache-tomcat-8.5.32/bin

sudo tar -zxvf tomcat-native.tar.gz

cd tomcat-native-1.2.17-src

cd native/

sudo ./configure --with-apr=/usr/bin/apr-1-config \

 --with-java-home=/home/pi/NetConTop/jdk/jdk1.8.0_301 \

 --with-ssl=yes \

 --prefix=/usr/local/apache-tomcat-8.5.32

sudo make && sudo make install



## 7. **Database installation**

sudo apt install mariadb-server

Configure mariadb-server: sudo mysql

use mysql;

UPDATE user SET password = password('1234') WHERE user = "root";

UPDATE user SET plugin = 'mysql_native_password' WHERE user = 'root';

flush privileges;

Restart the service after execution: sudo systemctl restart mariadb

After completion, execute login to see the effect: mysql -u root -p

Configure database remote access: sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

Find the commented out port and uncommented bind-address, uncomment and add comments respectively. After saving, restart the service.

sudo systemctl restart mariadb

mysql -u root -p

Create a remote access account

CREATE user 'root'@'%' identified by '1234';

GRANT ALL PRIVILEGES ON *.* TO root@'%' WITH GRANT OPTION;

FLUSH PRIVILEGES;

Insert into database

mysql -u root -p

create database netcontop;

use netcontop;

set names utf8;

source /home/pi/NetConTop/tomcat/netcontop.sql;



## 8. **Installation of octave**

sudo apt-get update

sudo apt-get install octave

sudo apt-get install octave-control

sudo apt-get install liboctave-dev

sudo apt-get install octave-sockets

## 9. redis-server

sudo apt-get install redis-server

sudo nano /etc/redis/redis.conf

Comment out the line bind 127.0.0.1::1

sudo /etc/init.d/redis-server restart
