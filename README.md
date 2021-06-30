# Config a Web Server on Ubuntu 20.04

### These instructions assume that you have already created a droplet with ubuntu, if you don't know how to do that, see how [here](https://www.digitaloceanbr.com.br/como-criar-droplet-digitalocean.html#:~:text=Com%20o%20cadastro%20ativo%20realize,cria%C3%A7%C3%A3o%20de%20um%20novo%20servidor.).
### if you need create another user, see how [here](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04).
### These Documentation is created with based on this archicles
* https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-20-04-pt
* https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-20-04-pt
* https://www.digitalocean.com/community/tutorials/how-to-configure-apache-http-with-mpm-event-and-php-fpm-on-ubuntu-18-04-pt
# 1. Setting Up a Basic Firewall
### 1.1type on bash 
```bash
sudo ufw app list
```

```bash
Out put
Available applications:
  OpenSSH
```
### 1.2 We need to make sure that the firewall allows SSH connections so that we can log back in next time. We can allow these connections by typing:
```bash
sudo ufw allow OpenSSH
```

### 1.3 enable firewall
```bash
sudo ufw enable
```

test if works type:
```bash
sudo ufw status
```
expected out put
```bash
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

# 2. config Apache Web Serve
### 2.1 Let's start by updating the local package index
```bash
  sudo apt update
```
### 2.2 Install apache
```bash
  sudo apt install apache2
```
### 2.3 Adjust Firewall to permit access to web ports
#### 2.3.1 List the applications using ufw:
```bash
  sudo ufw app list
```
Expected
```bash
  Output
Available applications:
  Apache
  Apache Full
  Apache Secure
  OpenSSH
```
#### 2.3.2 Release the port to apache
```bash
sudo ufw allow 'Apache'
```

check if it worked by typing:
```bash
sudo ufw status
```

```bash
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Apache                     ALLOW       Anywhere                
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Apache (v6)                ALLOW       Anywhere (v6)
```

### 2.4 Verify your webserver
The web server should already be up and running.

Check with the init systemd system to ensure the service is working by typing:
```bash
sudo systemctl status apache2
```

```bash
Output
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2020-04-23 22:36:30 UTC; 20h ago
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 29435 (apache2)
      Tasks: 55 (limit: 1137)
     Memory: 8.0M
     CGroup: /system.slice/apache2.service
             ├─29435 /usr/sbin/apache2 -k start
             ├─29437 /usr/sbin/apache2 -k start
             └─29438 /usr/sbin/apache2 -k start
```

### 2.5 Config Virtual Host
#### 2.5.1 Create a new diretory for your virtual host
```bash
     sudo mkdir /var/www/your_domain
```
#### 2.5.2 Grant the necessary permissions to this new directory
```bash
  sudo chown -R $USER:$USER /var/www/your_domain
```
```bash
  sudo chmod -R 755 /var/www/your_domain
```

#### 2.5.3 Now for test create a index.html in this directory
```bash
sudo nano /var/www/your_domain/index.html
```
and paste this html code:
```hmtl
<html>
    <head>
        <title>Welcome to Your_domain!</title>
    </head>
    <body>
        <h1>Success!  The your_domain virtual host is working!</h1>
    </body>
</html>
```
#### 2.5.4 In order for Apache to serve this content, it is necessary to create a virtual host file with the correct directives.
```bash
sudo nano /etc/apache2/sites-available/your_domain.conf
```
and paste and replace your_domain for the name of your virtual host
```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName your_domain
    ServerAlias www.your_domain
    DocumentRoot /var/www/your_domain
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
#### 2.5.5 Make apche using the virtual hsot
enable our file
```bash
sudo a2ensite your_domain.conf
```
and disable default file
```bash
sudo a2dissite 000-default.conf
```
Next, let's test for configuration errors:
```bash
sudo apache2ctl configtest
```
expected out put
```bash
Output
Syntax OK
```
if all ok lets restart apache for apply changes
```bash
sudo systemctl restart apache2
```

Now your virtual directory is configured access your domain and test if a index.html page is online

# 3 Config MYSQL server
```bash
sudo apt install mysql-server
```
When prompted, confirm the installation by typing Y and then ENTER.

It is recommended that you run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and block access to your database system. Start the interactive script by running:

```bash
sudo mysql_secure_installation
```
This script will ask you if you want to configure the VALIDATE PASSWORD PLUGIN.

Answer Y for yes, or anything else to continue without enabling it.

If you answer yes, you will be asked to select a password validation level.
```bash
There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary              file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 1
```
If you enable password validation, you will be presented with the password strength for the root password and your server will ask you if you want to continue with that password. If you are satisfied with your current password, type Y for “yes” at the prompt:
```bash
Estimated strength of the password: 100
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
```
For the rest of the questions, press Y and press the ENTER key at each prompt. This will remove some anonymous users and test database, disable remote logins for root, and load these new rules so that MySQL immediately respects the changes you've made.

When finished, test if you can log in to the MySQL console by typing:
```bash
sudo mysql
```
If works you see this text:
```bash
Output
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 22
Server version: 8.0.19-0ubuntu5 (Ubuntu)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

For exit mysql type:
```bash
mysql> exit;
```

# 4 Install PHP
We will install PHP and modules to communicate with MYSQL and APACHE for this type:
```bash
sudo apt install php libapache2-mod-php php-mysql
```
See PHP version:
```bash
php -v
```
# 5 Install PHP-FPM
### 5.1 Changing the Multiprocessing module
type this commands:
```bash
sudo systemctl stop apache2
```
```bash
sudo a2dismod php7.4
```
```
```bash
sudo a2dismod mpm_prefork
```
```
```bash
sudo a2enmod mpm_event
```
### 5.2 Configuring Apache HTTP to use FastCGI process manager
First install php-fpm
```bash
sudo apt install php-fpm
```
To communicate, Apache HTTP and PHP need a library that enables this capability
```bash
sudo apt install libapache2-mod-fcgid
```

Now firt enable php-fpm module
```bash
sudo a2enconf php7.4-fpm
```

Enable Apache proxy HTTP
```bash
sudo a2enmod proxy
```
Enable FastCGI proxy module on Apache HTTP:
```bash
sudo a2enmod proxy_fcgi
```
Now with everything installed let's do a configuration check:
```bash
sudo apachectl configtest
```
expected
```bash
Output
Syntax OK
```

Now restart Apache
```bash
sudo systemctl restart apache2
```
