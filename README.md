
# automaticWordPressDeployment


This repository contains all the necessary code and configuration files to set up an automated deployment process for a WordPress website using Nginx as the web server, LEMP (Linux, Nginx, MySQL, PHP) stack, and GitHub Actions as the CI/CD automation tool. The deployment process follows security best practices and ensures optimal performance of the website.




## Server Provisioning:

- (I am using AWS as the Cloud Provider)
- Provision an EC2 instance on AWS with a secure Linux distribution (e.g., Ubuntu 22.04).
- Configure security groups to allow necessary incoming traffic and restrict unnecessary access.
- Generate and configure SSH key pairs for secure remote access...

## Nginx, MySQL/MariaDB, and PHP Setup

 - Install and configure Nginx as the web server.

 ```
 sudo apt install nginx
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
```
