---
layout: single
title:  "Setup LEMP (Linux, Nginx, Mysql, PHP) stack for Laravel Apps"
categories:
  - Web
tags:
  - linux
  - laravel
  - php
  - nginx
---
Setting up your development environment sometimes can be overwhelming. you miss a command you'll end up scratching your head.
Let's admin it, we forget sometimes. That's why having a checklist important. And also, it saves a lot of time as we should focus on the actual program itself.

**Note:** For this demo. I'm using these follwing stacks `OS - Ubuntu 18.04 LTS` and `PHP Version 7.2`
{: .notice--info}

#### IMPORTANT: Before we start. I assume you are familiar with `linux`, `git` and `vim` commands.

Alright! let's do this.

### 1. Nginx Setup
Execute the script below line by line (Do not copy and paste the whole script in the command).
```yaml
sudo apt update
sudo apt install nginx -y
```
Enable nginx
```yaml
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 2. install PHP and its dependencies
You may change the php versions from the script below, depending on your current version.
```yaml
sudo apt install php7.2 php7.2-curl php7.2-common php7.2-cli php7.2-mysql php7.2-mbstring php7.2-fpm php7.2-xml php7.2-zip php7.2-gd php7.2-int php7.2-xsl git unzip zip -y
```
Next, navigate to `cd /etc/php/7.2/fpm` and open the `php.ini` file
```yaml
sudo vim php.ini
```
find and uncomment the `cgi.fix_pathinfo` and change the value from `1` to `0`.
once done. save the file by pressing `ESC` key, type `:x`, then hit `enter` to confirm.

Enable PHP
```yaml
sudo systemctl enable php7.2-fpm
sudo systemctl start php7.2-fpm
```

### 3. Maria DB setup
**Note:** You may skip this step if your not using Database locally.
{: .notice--info}
```yaml
sudo apt install mariadb-server mariadb-client -y
sudo systemctl start mysql
```
mysql installation
```yaml
sudo mysql_secure_installation
```
Then, answer the setup of questions by typing `Y` for yes and `N` for no.
### 4. Install composer
```yaml
sudo apt install composer -y
```

### 5. Connecting to project's github repository
Before you proceed from this step. I advise; that your project should be uploaded to your github repository for your convenience.
```yaml
sudo ssh-keygen -t rsa -b 4096 -C "github"
```
Just hit `enter` until the question is done.
```yaml
sudo ssh-keygen -t rsa -b 4096 -C "github"
```
A random encrypted key will appear. copy the `rsa` key until the end of `==` string only and then proceed to the next step.

### 6. Add Deploy keys from Github account
1. Go to your repository `settings`
2. Select `Deploy keys` from the sidebar menu
3. Click `Add Deploy key` button
4. Put some `title` then paste the  `rsa` key to the `key` textarea
5.  Click `Add Key` to confirm
Going back to your `command` line, run the script below to confirm the connection to your github repository.
```yaml
sudo ssh -T git@github.com
```
A confirmation question will appear. just confirm by answering `yes` in order to proceed.

### 7. Copying the resources / code resource
Create your project target folder
```yaml
mkdir -p /var/www/html/site
```
Pull project resource from your github repository
```yaml
sudo git clone git@github.com{user}/{repo}.git /var/www/html/site
```
Once you successfully copied the repo. make sure to install all of laravel's packages.
**Note:** Make sure your on your site directory (`\var\www\html\site`).
{: .notice--danger}
```yaml
sudo composer install
```
**Note:** You may replace the web app name by changing the `site` part from the above script by your prefer name.
{: .notice--info}

### 8. Enabling your site configuration.
Navigate to `nginx` folder.
```yaml
cd /etc/nginx
```
To create your `site` nginx configuration.
```yaml
sudo vim /etc/nginx/sites-available/default
```
Paste this sample server configuration and replace the `site` part to your project name
```yaml
server {
         listen 80 default_server;
         listen [::]:80 default_server ipv6only=on;
         # Log files for Debugging
         access_log /var/log/nginx/site-access.log;
         error_log /var/log/nginx/site-error.log;
         # Webroot Directory for Laravel project
         root /var/www/html/site/public;
         index index.php index.html index.htm;
         # Your Domain Name
         server_name site.com;

         location / {
                 try_files $uri $uri/ /index.php?$query_string;
         }
         # PHP-FPM Configuration Nginx
         location ~ \.php$ {
                 try_files $uri =404;
                 fastcgi_split_path_info ^(.+\.php)(/.+)$;
                 fastcgi_pass unix:/run/php/php7.2-fpm.sock;
                 fastcgi_index index.php;
                 fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                 include fastcgi_params;
         }
 }
 ```
After that, save the file `(vim command is required)`.

Then, the created `site` config file should also be linked to `sites-enabled` folder.
```yaml
sudo ln -s /etc/nginx/sites-available/site /etc/nginx/sites-enabled/
```

Open and the `default` file config and remove the `default_server` script on it. To give way to our `site` settings.
```yaml
sudo vim /etc/nginx/sites-available/default
```
Make sure to remove the `default_server` part. your first lines should be same from the script below.
```yaml
serrver {
    listen 80;
    listen [::]80;
```
Save the file `(vim command is required)`

### 9. Permissions
Give permission to your project folder in order to store and display assets.
```yaml
sudo chown -R www-data:root /var/www/html/site
sudo chmod 755 /var/www/html/site/storage
```
### 10. Finishing touch
For the last part for this setup. make sure you execute the following script to completely connect your app.
**Note:** Make sure your on your project path `cd /var/www/html/site` for the following script below
{: .notice--danger}
```yaml
 cp .env.example .env
 sudo vim .env
 ```
 confirm your database connection then save the file `(vim command is required)`
 ```yaml
 sudo php artisan key:generate
 ```
 Now. restart the nginx to make your that all the changes will be applied.
 ```yaml
 sudo systemctl restart nginx
 ```
 you should now able to open the site by opening a browser and time the server's `IP` in the URL bar.

To check if our configuration is correct; you may run the command below.
```yaml
sudo nginx -t
```
you should see an `OK` message there's no any problem with our setup.








