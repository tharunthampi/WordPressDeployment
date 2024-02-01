
# automaticWordPressDeployment


This repository contains all the necessary code and configuration files to set up an automated deployment process for a WordPress website using Nginx as the web server, LEMP (Linux, Nginx, MySQL, PHP) stack, and GitHub Actions as the CI/CD automation tool. The deployment process follows security best practices and ensures optimal performance of the website.




## 🛠️ Server Provisioning

- (I am using AWS as the Cloud Provider)
- Provision an EC2 instance on AWS with a secure Linux distribution (e.g., Ubuntu 22.04).
- Configure security groups to allow necessary incoming traffic and restrict unnecessary access.
- Generate and configure SSH key pairs for secure remote access...

## 💻 Nginx, MySQL/MariaDB, and PHP Setup

 - Install and configure Nginx as the web server.

 ```
 sudo apt install nginx
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
```
- If firewall is active/going to enable we need to allow NGINX with firewall we using here ufw

```
sudo ufw app list
sudo ufw allow 'Nginx HTTP'
sudo ufw allow 'Nginx HTTPS'
sudo ufw allow ssh
sudo ufw status
sudo ufw enable
```
- restart nginx :
```
sudo systemctl restart nginx
```
- Visit public ip to see nginx welcome page :

```
http://wordpress.servebeer.com/

```
- Install and configure mysql server :

```
sudo apt install mysql-server 
sudo mysql_secure_installation
sudo systemctl restart mysql
sudo systemctl enable mysql
```
- If you enabled ufw allow traffic to mysql on its default port 3306 :
```
    sudo ufw allow from any to any port 3306 proto tcp

```
- Install and setup PHP :
```
apt install php-fpm php-mysql php php-curl php-gd php-intl php-zip 
add-apt-repository ppa:ondrej/php
apt-get install php7.4-fpm
sudo systemctl enable php7.4-fpm
sudo systemctl start php7.4-fpm
```
## 🌐 WordPress Website Configuration

- Navigate to your Nginx web server root directory.
```
cd /var/www/html/
```
- Download and Extract WordPress:
```
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xzvf latest.tar.gz
```
- Set Permissions:
```
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress
```
- Create a MySQL Database and User:
  
    Access MySQL with root privileges: 
```
sudo mysql -u root -p
```
change root password and create user for wordpress :

```
    sudo mysql
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password by 'Testpassword@123';
    CREATE DATABASE wp;
    CREATE USER 'wp_user'@localhost IDENTIFIED BY 'Testpassword@123';
    GRANT ALL PRIVILEGES ON wp.* TO 'wp_user'@localhost;
    FLUSH PRIVILEGES;
    exit;
```

- Configure WordPress:
```
cp /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php

vi /var/www/html/wordpress/wp-config.php
```
Update the database details with the database name, username, and password we created earlier.

- To Secure  WordPress Installation:

After installation, follow best practices for WordPress security. This includes strong passwords, limiting user privileges

- Configure Nginx to access the wordpress :

```
mv /etc/nginx/sites-enabled/default /etc/nginx/sites-enabled/wordpress
```
Edit the config file wordpress
```
vi /etc/nginx/sites-enabled/wordpress
```
Scroll down to the location / {.Add the following lines in this section.
```
try_files $uri $uri/ /index.php?$args;
```
Uncommend below lines
```
 location ~ \.php$ {
                include snippets/fastcgi-php.conf;
fastcgi_pass unix:/run/php/php7.4-fpm.sock;
```
Add index.php below section

```
index index.php index.html index.htm index.nginx-debian.html;

```
Visit public ip to see wordpress setup page :

```
https://wordpress.servebeer.com/wp-admin/

```
## Implement SSL/TLS certificate using Let's Encrypt for secure communication

  Install  certbot
```
sudo apt install certbot python3-certbot-nginx
```
verify nginx conf file with below command
```
sudo nginx -t 
```
reload nginx :
```
sudo systemctl reload nginx
```
generate certs with certbot :
```
sudo certbot --nginx -d wordpress.servebeer.com
```
Update WordPress Address (URL) and Site Address (URL) from wp-admin page

```
Site Address (URL): https://wordpress.servebeer.com
WordPress Address (URL): https://wordpress.servebeer.com

```
## GitHub Repository Setup

- Create a GitHub repository for this project.

- Generate id_rsa key files
```
ssh-keygen -t rsa -b 4096 -C "mailid"
```
- Go to GitHub account >> Settings >> SSH section >> add id_rsa.pub key
- Go to repository >> Settings >>Actions secrets and variables >> add below  secrets

```
HOST: The hostname or IP address of the production server.
USERNANE: root
SSH_PRIVATE_KEY : generated id_rsa.pub key
```
- Link to Your GitHub Repository:
Go back to GitHub, and  find the URL for the repository. Link the local repository to GitHub by following the instructions on the GitHub website.

```
cd /var/www/html/wordpress
git init
git remote add origin  your-github-repo-url.git
git add .
git branch -M main
git push -u origin main
```
- files pushed from the ec2 instances to the git repository
- The WordPress site should now be running on the server, and the code is version-controlled in the GitHub repository. we can push updates to GitHub whenever we make changes to our site.
# 🛠️ GitHub Actions Workflow

This workflow is designed to be super-easy to integrate into your own WordPress project or any other PHP-based project. It's designed to ensure code quality and deploy changes to both staging and production environments from the main branch.
 
 - clikc on actions button on our git repo, click on set up a workflow yourself button. create a main.yaml file

 - this deployment file i am trying to build themes by installing dependacies and building then after that i am syncing the directory with rsync utility to sych with my wordpress site location files and send securly.

 ```
 name: Push Code to EC2

on:
  push:
    branches:
      - main

jobs:
  push_to_ec2:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2

      - name: Set up SSH
        uses: webfactory/ssh-agent@v0.8.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      - name: Install dependencies
        run: |
          # Add your commands to install project dependencies
          # For example, if you are using Node.js:
          # npm install
      - name: Build the project
        run: |
          # Add your commands to build the project
          # For example, if you are using Node.js:
          # npm run build
      - name: Install rsync
        run: sudo apt-get update && sudo apt-get install -y rsync

      - name: Add EC2 instance host key to known hosts
        run: |
          ssh-keyscan -H $HOST >> $HOME/.ssh/known_hosts
        env:
          HOST: ${{ secrets.HOST }}

      - name: Push code to EC2
        run: |
          # Set IP address and directory paths
          EC2_IP="$HOST"
          SOURCE_DIR="$GITHUB_WORKSPACE/"
          DEST_DIR="/var/www/html/wordpress/"
          # Use rsync with SSH key authentication and compression
          rsync -avz --delete-after --exclude='.git' -e "ssh -i $SSH_AUTH_SOCK" "$SOURCE_DIR" "${{ secrets.USERNAME }}@$EC2_IP:$DEST_DIR"
        env:
          HOST: ${{ secrets.HOST }}
          USERNAME: ${{ secrets.USERNAME }}
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```

- Testing Workflow

careate new test file >> push the change >> clikc on actions button on our git repo,thre we can see the result.
## Optimize nginx performance

- lets turn off server_token in nginx config file to turn off server info in error page and stop click jacking attacks add this lines to nginx conf file.

```
vi /etc/nginx/nginx.conf
server_tokens off;
```
add the following line in to
```
vi /etc/nginx/sites-available/default

```
add below line in to server block :
```
more_clear_headers Server;
```
Modifing gzip compression values in nginx.conf files for better compression add below configs

```
 gzip on;

        gzip_comp_level 5;
gzip_min_length 256;
gzip_proxied any;
gzip_vary on;
gzip_types
    application/atom+xml
    application/javascript
    application/json
    application/ld+json
    application/manifest+json
    application/rss+xml
    application/vnd.geo+json
    application/vnd.ms-fontobject
    application/x-font-ttf
    application/x-web-app-manifest+json
    application/xhtml+xml
    application/xml
    font/opentype
    image/bmp
    image/svg+xml
    image/x-icon
    text/cache-manifest
    text/css
    text/plain
    text/vcard
    text/vnd.rim.location.xloc
    text/vtt
    text/x-component
    text/x-cross-domain-policy;
```
save and close conf file, restart nginx to take effect using below command :

```
    sudo nginx -t
    sudo systemctl restart nginx   
```
- By modifying the number of worker connections, you can simultaneously manage the maximum number of links your server can handle

```
vi /etc/nginx/nginx.conf

worker_connections 1024;
```
## setup Nginx FastCGI Page Cache With WordPress
Edit Nginx main configuration file.

```
vi /etc/nginx/nginx.conf
```
In the http {…} context, add the following 2 lines:
```
    fastcgi_cache_path /etc/nginx/cache levels=1:2 keys_zone=wpcache:200m max_size=10g inactive=2h use_temp_path=off;
    fastcgi_cache_key "$scheme$request_method$host$request_uri";
    fastcgi_ignore_headers Cache-Control Expires Set-Cookie;
```

```
vi /etc/nginx/sites-available/default
```
Scroll down to the location ~ \.php$ section. Add the following lines in this section.

```
fastcgi_cache phpcache;
fastcgi_cache_valid 200 301 302 60m;
fastcgi_cache_use_stale error timeout updating invalid_header http_500 http_503;
fastcgi_cache_min_uses 1;
fastcgi_cache_lock on;
add_header X-FastCGI-Cache $upstream_cache_status;
```
save and close the file

```
    sudo nginx -t
    sudo systemctl restart nginx        
```

Ensure that all alterations and additions made to the WordPress files, including themes and plugins, are incorporated into the local Git repository. This facilitates seamless deployment to remote servers directly from the local Git directory. By committing changes to the Git repository, the modifications are then automatically deployed to the WordPress site through the utilization of GitHub Actions. This streamlined process ensures that each committed change triggers an automated deployment to the live WordPress site, enhancing efficiency and maintaining synchronization between the local development environment and the remote server.
