
# LAMP

_This is my short guide on setting up LAMP on a fresh installation of BunsenLabs GNU/Linux 9.8 (Helium) x86_64._

## INSTALL APACHE
```
sudo apt-get install apache2
sudo a2enmod rewrite
```
---
## INSTALL UFW FIREWALL
```
sudo apt install ufw
```
See what's available
```
sudo ufw app list
```
OUTPUT
> 
```
Available applications:
...
  WWW
  WWW Cache
  WWW Full
  WWW Secure
...
```

Allow WWW
```
sudo ufw allow in "WWW Full"
```
---
## INSTALL MARIADB
```
sudo apt install mariadb-server
```
Secure MariaDB
```
sudo mysql_secure_installation
```
> Enter current password for root (enter for none): ENTER

> Set root password? N

> Remove anonymous users? ENTER

> Disallow root login remotely? ENTER

> Remove test database and access to it? ENTER

> Reload privilege tables now? ENTER

Create new root user with admin privileges.
```
sudo mariadb
```
This is where you set up your new user and password, change `admin/password` below.
```
GRANT ALL ON *.* TO 'admin'@'localhost' IDENTIFIED BY 'password' WITH GRANT OPTION;
```
```
FLUSH PRIVILEGES;
```
```
exit
```
Login like this
```
mariadb -u admin -p
```
---
## INSTALL PHP
```
sudo apt install php libapache2-mod-php php-mysql php-gd
```
Apache will try and serve `index.html` before `index.php` so we need to change the order.
```
sudo nano /etc/apache2/mods-enabled/dir.conf
```
Restart apache and check status
```
sudo systemctl restart apache2
sudo systemctl status apache2
```
Search for php modules
```
apt search php- | less
```
View details of a module
```
apt show package_name
```
TEST PHP is working
```
sudo nano /var/www/html/info.php
```
```
<?php
    phpinfo();
?>
```
Check it out here [localhost/info.php](http://localhost/info.php)

---
## VIRTUALHOSTS
```
mkdir ~/www && cd ~/www
```
First, create a file in the `/etc/apache2/sites-available/` directory for each virtual host that you want to set up.
```
sudo nano /etc/apache2/sites-available/qwerty.webdev.conf
```
```
<VirtualHost *:80>
	ServerAdmin webmaster@qwerty.dev
	ServerName qwerty.dev
	ServerAlias www.qwerty.dev
	DocumentRoot /home/jaimito/www/qwerty.dev/public_html/
	ErrorLog /home/jaimito/www/qwerty.dev/logs/error.log
	CustomLog /home/jaimito/www/qwerty.dev/logs/access.log combined
</VirtualHost>
```
Make the folder structure
```
mkdir -p /home/jaimito/www/qwerty.dev/public_html/
mkdir -p /home/jaimito/www/qwerty.dev/logs/
nano ~/www/qwerty.dev/public_html/index.php
```
Add virtualhost to hosts file
```
sudo nano /etc/hosts
```
Enable virtualhost
```
sudo a2ensite qwerty.webdev.conf
```
Tell apache to load `~/www`
```
sudo nano /etc/apache2/apache2.conf
```
FIND:
```
<Directory /var/www/>
...
<Directory /srv/>
```
PASTE this after
```
<Directory /home/jaimito/www/>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```
Go to last line of file and paste:
```
ServerName 127.0.0.1
```
Granting Permissions to the directories.
```
sudo chown -R www-data:www-data /home/jaimito/www
sudo chmod -R 755 /home/jaimito/www
```
Restart apache
```
sudo service apache2 restart
```
---

## INSTALL ADMINER
```
sudo nano /etc/apache2/sites-available/adminer.webdev.conf
```
```
<VirtualHost *:80>
  ServerAdmin webmaster@church.webdev
  ServerName church.webdev
  ServerAlias www.church.webdev
  DocumentRoot /home/jaimito/www/church.webdev/public_html/
  ErrorLog /home/jaimito/www/church.webdev/logs/error.log
  CustomLog /home/jaimito/www/church.webdev/logs/access.log combined
  <Directory /home/jaimito/www/church.webdev/public_html>
     AllowOverride all
     Options Indexes FollowSymLinks
     Require local
  </Directory>
</VirtualHost>
```
```
mkdir -p /home/jaimito/www/adminer.webdev/public_html/
mkdir -p /home/jaimito/www/adminer.webdev/logs/
cd ~/www/adminer.webdev/public_html
wget https://github.com/vrana/adminer/releases/download/v4.7.1/adminer-4.7.1-mysql-en.php && mv adminer-4.7.1-mysql-en.php index.php
```
Add virtualhost to hosts file
```
sudo nano /etc/hosts
```
Enable virtualhost
```
sudo a2ensite adminer.webdev.conf
```
```
sudo service apache2 restart
```
---
## Take ownership of the `.git` folder back
```
sudo chown -R jaimito:jaimito ~/www/church.webdev/.git
```
---
