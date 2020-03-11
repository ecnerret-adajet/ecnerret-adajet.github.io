---
layout: single
title:  "Connect Sql Server on PHP apps (on Mac Mojave OS)"
categories:
  - Web
tags:
  - sql-server
  - php
  - mac-mojave
---
I find it hard to see resources or related post on how to integrate from these tech ( Sql server,PHP and Mac Mojave OS). especially, you can much relate when you are a laravel developer and you are using Sql server as your database of choice.

Without further ado, Here's the simplified instruction on how you can connect SQL server with PHP on Mac Mojave OS.
### 1. Install FreeTDS and unixODBC
```yaml
brew update
brew install unixodbc freetds
```
### 2. freetds.conf configuration file
locate `freetds.conf` file from this path `/usr/local/etc/`. then run the command below to edit the file.
```yaml
vim freetds.conf
```
after that, add the script below the `freetds.conf` file. replace the `host` parameter with the SQL server host name or IP.
```yaml
[MYMSSQL]
host = 10.99.99.99
port = 1433
tds version = 7.3
```
### 3. Test connection
once you save the config file, your good to go. to test if the settings is correct, run the command below where `user` and `password` should be the actual sql credential.
```yaml
tsql -S MYMSSQL -U user -P password
```
if it works, you should see the following:
```
locale is "en_US.UTF-8"
locale charset is "UTF-8"
using default charset "UTF-8"
1>
```
Now. you may test your application to confirm if it will run properly.

However, you may continue from these steps when you encounter a `Driver not found` error message.

### 1. Install Microsoft Drivers for PHP for SQL server
Open `command` and run the following script one by one.
```yaml
brew install php-pear php7.3-dev
sudo pecl install sqlsrv
sudo pecl install pdo_sqlsrv
```
**Note:** Replace the php version on `brew install php-pear php7.3-dev` command based on your current version.
{: .notice--danger}
### 2. Load the drivers
again, make sure to replace the `php version` from the script below:
```yaml
sudo su
echo "extension=sqlsrv.so" > /etc/php/7.3/mods-available/sqlsrv.ini
echo "extension=pdo_sqlsrv.so" > /etc/php/7.3/mods-available/pdo_sqlsrv.ini
exit
```
 You may enable the extension for either fpm or both

Enable sqlsrv extensions for cli
 ```
sudo ln -s /etc/php/7.3/mods-available/sqlsrv.ini /etc/php/7.3/cli/conf.d/20-sqlsrv.ini
sudo ln -s /etc/php/7.3/mods-available/pdo_sqlsrv.ini /etc/php/7.3/cli/conf.d/20-pdo_sqlsrv.ini
```

Enable sqlsrv extensions for fpm
 ```
sudo ln -s /etc/php/7.3/mods-available/sqlsrv.ini /etc/php/7.3/cli/conf.d/20-sqlsrv.ini
sudo ln -s /etc/php/7.3/mods-available/pdo_sqlsrv.ini /etc/php/7.3/cli/conf.d/20-pdo_sqlsrv.ini
```
finally, restart your `php-fpm` and `nginx/apache` services and it should connect.
