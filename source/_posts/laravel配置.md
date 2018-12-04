---
title: laravel配置
date: 2018-12-03 17:19:08
tags: "PHP"
---
### 准备环境

参考路径： https://www.codecasts.com/discuss/laravel/laravel-project-from-scratch-deployment-752

#### 安装PHP环境

1. 添加php源
```bash

sudo tee -a /etc/apt/sources.list.d/php.list << EOF
deb http://ppa.launchpad.net/ondrej/php/ubuntu $(lsb_release -c --short) main 
deb-src http://ppa.launchpad.net/ondrej/php/ubuntu $(lsb_release -c --short) main 
EOF
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv 14AA40EC0831756756D7F66C4F4EA0AAE5267A6C
sudo apt update
```
2. 安装php
```bash
echo y | sudo apt install libnss3-tools jq xsel 
echo y | sudo apt install php7.1-cli php7.1-common php7.1-curl php7.1-json php7.1-mbstring php7.1-mcrypt php7.1-opcache php7.1-readline php7.1-xml php7.1-zip php7.1-sqlite3 php7.1-mysql php7.1-pgsql
echo y | sudo apt install php7.1 php7.1-fpm
```
3.扩展,mysql安装
```bash
sudo apt-get -y install php7.1-mysql
sudo apt-get install php7.1-fpm  php7.1-curl php7.1-xml php7.1-mcrypt php7.1-json php7.1-gd php7.1-mbstring
sudo apt-get -y install nginx
sudo apt-get -y install mysql-server-5.7
```
4.测试php

```php
php -v
```


##### 配置PHP
1. 编辑php.ini文件 
```bash
操作：
sudo vim /etc/php/7.1/fpm/php.ini

//具体修改：

将cgi.fix_pathinfo=1这一行去掉注释，将1改为0

```
2. 编辑 www.conf 文件
```bash
sudo vim /etc/php/7.1/fpm/pool.d/www.conf 

//具体配置：(一般不用修改，下面nginx配置会用到这个路径)
listen = /run/php/php7.1-fpm.sock

```
3. 重启php-fpm

```bash
sudo service php7.1-fpm restart
```

#### 安装Nginx环境

1. 添加Nginx源
```bash
sudo tee -a /etc/apt/sources.list.d/nginx.list << EOF
deb http://ppa.launchpad.net/nginx/stable/ubuntu $(lsb_release -c --short) main 
deb-src http://ppa.launchpad.net/nginx/stable/ubuntu $(lsb_release -c --short) main 
EOF
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv 8B3981E7A6852F782CC4951600A6F0A3C300EE8C
sudo apt update
```

2. 安装Nginx
```bash
echo y | sudo apt install nginx
```
##### 安装composer

```bash
wget https://getcomposer.org/composer.phar
chmod +x composer.phar
sudo mv composer.phar /usr/local/bin/composer
```

切换到国内composer源

```bash
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

##### 配置Nginx环境
  1. 修改default文件
  
  ```bash
  sudo vim /etc/nginx/sites-available/default
  ```
  基础配置：
  ```bash
   listen 80 default_server;
          listen [::]:80 default_server ipv6only=on;
          # root /var/www/html;
          root /var/www/laravel-ubuntu/public; #定位到public文件夹
          index index.php index.html index.htm;
  
          # Make site accessible from http://localhost/
          server_name localhost;
          # 路由重定向
          location / {
                  # First attempt to serve request as file, then
                  # as directory, then fall back to displaying a 404.
                  try_files $uri $uri/ /index.php?$query_string;
                  # Uncomment to enable naxsi on this location
                  # include /etc/nginx/naxsi.rules
          }
          # 解析php语法配置
          location ~ \.php$ {
                  try_files $uri /index.php =404;
                  fastcgi_split_path_info ^(.+\.php)(/.+)$;
                  fastcgi_pass unix:/run/php/php7.1-fpm.sock;  # 具体在/etc/php/7.1/fpm/pool.d/www.conf 中配置 
                  fastcgi_index index.php;
                  fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                  include fastcgi_params;
          }
  ```
   注意：laravel-ubuntu 这个目录的所有者为: www-data:www-data
  
  ```bash
  
  //服务器根目录
   sudo chown -R www-data:www-data  /var/www/html
  
  //laravel目录
  sudo chown -R www-data:www-data  /var/www/html/laravel-ubuntu
  
  ```
  
  2. 配置nginx.conf文件
  
  ```bash
   sudo vi /etc/nginx/nginx.conf
   
   // 具体配置 ,必须和/var/www/html文件夹中的`项目目录拥有者`相同
   user  www-data;
   
  ```

  ##### 给storage文件夹权限
  ```bash
    sudo chmod 777 /laravel-ubuntu/storage/
  ```
  
  ##### 重启Nginx
  
   ```bash
   sudo service  nginx  restart
   ```
   
   
   
