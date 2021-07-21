# Config a Web Server on Ubuntu 20.04

### These instructions assume that you have already created a droplet with ubuntu, if you don't know how to do that, see how [here](https://www.digitaloceanbr.com.br/como-criar-droplet-digitalocean.html#:~:text=Com%20o%20cadastro%20ativo%20realize,cria%C3%A7%C3%A3o%20de%20um%20novo%20servidor.).
### if you need create another user, see how [here](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04).
### These Documentation is created with based on this archicles
* https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-20-04-pt
* https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-20-04-pt
* https://www.digitalocean.com/community/tutorials/how-to-configure-apache-http-with-mpm-event-and-php-fpm-on-ubuntu-18-04-pt
* https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-20-04
# 1. Setting Up a Basic Firewall
### 1.1 Type on bash 
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

### 1.3 Enable firewall
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

# 2. Config Apache Web Serve
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
sudo ufw allow 'Apache Full'
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
Apache Full                   ALLOW       Anywhere                
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Apache Full (v6)                ALLOW       Anywhere (v6)
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
Enable our file
```bash
sudo a2ensite your_domain.conf
```
And disable default file
```bash
sudo a2dissite 000-default.conf
```
Next, let's test for configuration errors:
```bash
sudo apache2ctl configtest
```
Expected out put
```bash
Output
Syntax OK
```
If all ok lets restart apache for apply changes
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


```bash
sudo a2dismod mpm_prefork
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

# 6 Configure Let's Encrypt

### 6.1 Installing Certbot

```bash
 sudo apt install certbot python3-certbot-apache
```

### 6.2 Reload apache for the changes to be applied

```bash
 sudo apt install certbot python3-certbot-apache
```

### 6.3 Obtain an SSL Certificate
```bash
 sudo certbot --apache
```

This script will prompt you to answer a series of questions in order to configure your SSL certificate. First, it will ask you for a valid e-mail address. This email will be used for renewal notifications and security notices:

 ```bash
Output
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator apache, Installer apache
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): you@your_domain
```

After providing a valid e-mail address, hit ENTER to proceed to the next step. You will then be prompted to confirm if you agree to Let’s Encrypt terms of service. You can confirm by pressing A and then ENTER:

```bash
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: A
```

Next, you’ll be asked if you would like to share your email with the Electronic Frontier Foundation to receive news and other information. 

```bash
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: N
```

The next step will prompt you to inform Certbot of which domains you’d like to activate HTTPS for. The listed domain names are automatically obtained from your Apache virtual host configuration, that’s why it’s important to make sure you have the correct ServerName and ServerAlias settings configured in your virtual host. If you’d like to enable HTTPS for all listed domain names (recommended), you can leave the prompt blank and hit ENTER to proceed. Otherwise, select the domains you want to enable HTTPS for by listing each appropriate number, separated by commas and/ or spaces, then hit ENTER.

```bash
Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: your_domain
2: www.your_domain
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 
```
You’ll see output like this:
```bash
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for your_domain
http-01 challenge for www.your_domain
Enabled Apache rewrite module
Waiting for verification...
Cleaning up challenges
Created an SSL vhost at /etc/apache2/sites-available/your_domain-le-ssl.conf
Enabled Apache socache_shmcb module
Enabled Apache ssl module
Deploying Certificate to VirtualHost /etc/apache2/sites-available/your_domain-le-ssl.conf
Enabling available site: /etc/apache2/sites-available/your_domain-le-ssl.conf
Deploying Certificate to VirtualHost /etc/apache2/sites-available/your_domain-le-ssl.conf
```

Next, you’ll be prompted to select whether or not you want HTTP traffic redirected to HTTPS. In practice, that means when someone visits your website through unencrypted channels (HTTP), they will be automatically redirected to the HTTPS address of your website. Choose 2 to enable the redirection, or 1 if you want to keep both HTTP and HTTPS as separate methods of accessing your website.
```bash
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
```

After this step, Certbot’s configuration is finished, you recive this output:

```bash
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled https://your_domain and
https://www.your_domain

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=your_domain
https://www.ssllabs.com/ssltest/analyze.html?d=www.your_domain
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/your_domain/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/your_domain/privkey.pem
   Your cert will expire on 2020-07-27. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

### 6.4 Verifying Certbot Auto Renewal

```bash
sudo systemctl status certbot.timer
```

```bash
Output

● certbot.timer - Run certbot twice daily
     Loaded: loaded (/lib/systemd/system/certbot.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Tue 2020-04-28 17:57:48 UTC; 17h ago
    Trigger: Wed 2020-04-29 23:50:31 UTC; 12h left
   Triggers: ● certbot.service

Apr 28 17:57:48 fine-turtle systemd[1]: Started Run certbot twice daily.
```

Try test renewal process run this command:

```bash
sudo certbot renew --dry-run
```

If don't have erros, are all ok, Certbot will renew your certificates
