
# LAMP
>_This is my detailed guide on setting up a LAMP server on BunsenLabs GNU/Linux 9.8 (Helium) x86_64. It should work fine on other Debian distros like *buntu._
>
>_I also show you how to setup, enable and disable VirtualHosts in your `/home/` directory with proper permissions._

>>_This guide will install the following versions:_
>> - _Apache/2.4.25 (Debian)_
>> - _Mariadb  Ver 15.1 Distrib 10.1.37-MariaDB,_
>> - _PHP 7.0.33-0+deb9u3_
>> - _UFW 0.35_
>>
>> _Your requirements may be different than mine. There are plenty of guides and tutorials out there but these versions suit me just fine `:)`_

---
## Table of Contents
 - [Apache](#APACHE)
 - [MariaDB](#MARIADB)
 - [PHP](#PHP)
 - [UFW](#UFW)
 - [VirtualHosts](#VIRTUALHOSTS)
 - [Adminer](#ADMINER)
 - [Permissions](#PERMISSIONS)

---
## APACHE
```
sudo apt-get install apache2 -y
sudo a2enmod rewrite
sudo systemctl restart apache2
```
[top](#LAMP)
---
## MARIADB
```
sudo apt install mariadb-server -y
```
Secure MariaDB
```
sudo mysql_secure_installation
```
> Enter current password for root (enter for none): **ENTER**

> Set root password? **N**

> Remove anonymous users? **ENTER**

> Disallow root login remotely? **ENTER**

> Remove test database and access to it? **ENTER**

> Reload privilege tables now? **ENTER**

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
If you should need to login in the future, this is the command
```
mariadb -u admin -p
```
[top](#LAMP)
---
## PHP
```
sudo apt install php libapache2-mod-php php-mysql php-gd -y
```
Apache will try and serve `index.html` before `index.php` so we need to change the order.
```
sudo nano /etc/apache2/mods-enabled/dir.conf
```
> BEFORE:
```
<IfModule mod_dir.c>
        DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
</IfModule>
```
> AFTER:
```
<IfModule mod_dir.c>
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
Restart apache and check status
```
sudo systemctl restart apache2
sudo systemctl status apache2
```
>OUTPUT:
>```
>apache2.service - The Apache HTTP Server
>   Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
>   Active: active (running) since Thu 2019-04-04 11:22:09 MDT; 8s ago
>  Process: 680 ExecStop=/usr/sbin/apachectl stop (code=exited, status=0/SUCCESS)
>  Process: 689 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCESS)
> Main PID: 695 (apache2)
>    Tasks: 6 (limit: 4915)
>   CGroup: /system.slice/apache2.service
>           ├─695 /usr/sbin/apache2 -k start
>           ├─712 /usr/sbin/apache2 -k start
>           ├─713 /usr/sbin/apache2 -k start
>           ├─714 /usr/sbin/apache2 -k start
>           ├─715 /usr/sbin/apache2 -k start
>           └─716 /usr/sbin/apache2 -k start
>
>Apr 04 11:22:09 acer systemd[1]: Starting The Apache HTTP Server...
>Apr 04 11:22:09 acer systemd[1]: Started The Apache HTTP Server.
>```
> Feel free to install any php-modules you need. I installed phpgd as it is needed for my chosen CMS - [ProcessWire](https://github.com/processwire/processwire).

Search for php modules
```
apt search php- | less
```
View details of a module
```
apt show php-gd
```
Let's test that php is working. Copy/Paste this entire block
```
sudo tee -a /var/www/html/info.php << END
<?php
  phpinfo();
?>
END
```
Check it out here [localhost/info.php](http://localhost/info.php)

Let's remove the file
```
sudo rm /var/www/html/info.php
```
[top](#LAMP)
---
## UFW

> #### What is UFW?
>
> _**Uncomplicated Firewall** is a program for managing a netfilter firewall designed to be easy to use. It uses a command-line interface consisting of a small number of simple commands, and uses iptables for configuration._
```
sudo apt install ufw
```
Open firewall access to post 80/http:
```
sudo ufw allow http
```
Open firewall access to port 22/ssh:
```
sudo ufw allow ssh
```
Let's enable UFW
```
sudo ufw enable
```
> OUTPUT:
>```
>active and enabled on system startup
>```
Let's check the status of UFW:
```
sudo ufw status
```

>OUTPUT:
>```
>To                         Action      From
>--                         ------      ----
>80/tcp                     ALLOW       Anywhere
>22/tcp                     ALLOW       Anywhere
>80/tcp (v6)                ALLOW       Anywhere (v6)
>22/tcp (v6)                ALLOW       Anywhere (v6)
>```














See what's available
```
sudo ufw app list
```
You should see the following output or similar
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
Let's give access to `http` and `https`
```
sudo ufw allow in "WWW Full"
```
[top](#LAMP)
---
## VIRTUALHOSTS
> _I prefer to have my projects served directly from my home directory instead of serving files from `/var/www/html/projectX`._
>
> _Plus is just makes it easy to have all my own personal files in `home` anyways._
```
mkdir ~/www
```
Let's create a file in the `/etc/apache2/sites-available/` directory for each virtual host that we want to set up.

I like to use the TLD of `.webdev`. You can use `local`, `localhost`, `test`, anything you want.

Let's make `project1.webdev`

Copy/Paste this entire block _(don't forget to replace my username `organizedfellow` with your own)_.


```
sudo tee -a /etc/apache2/sites-available/project1.webdev.conf << END
<VirtualHost *:80>
	ServerAdmin webmaster@project1.webdev
	ServerName project1.webdev
	ServerAlias www.project1.webdev
	DocumentRoot /home/organizedfellow/www/project1.webdev/public_html/
	ErrorLog /home/organizedfellow/www/project1.webdev/logs/error.log
	CustomLog /home/organizedfellow/www/project1.webdev/logs/access.log combined
    <Directory "/home/organizedfellow/www/project1.webdev/public_html">
        AllowOverride all
        Options Indexes FollowSymLinks
        Require local
    </Directory>
</VirtualHost>
END
```
Let's create the folder structure for our project
```
mkdir -p ~/www/project1.webdev/public_html/
mkdir -p ~/www/project1.webdev/logs/
```
As you can see, apache logs will be kept directly within out project. I find this makes troubleshooting and debugging a bit simpler.

```
sudo tee -a ~/www/project1.webdev/public_html/index.php << END
<?php
  echo "Hello World!";
?>
END
```
Before editing our `hosts` file, we back it up
```
sudo cp /etc/hosts /etc/hosts.bak
```
Now we can edit `hosts` file
```
sudo echo "127.0.0.1 www.project1.webdev project1.webdev" | sudo tee -a /etc/hosts
```
We have to tell apache where to look for our projects since we will be using our `/home/` directory
```
sudo nano /etc/apache2/apache2.conf
```
FIND:
```
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```
Replace `/var/www/` with `/home/organizedfellow/www/`

Like this _(don't forget to replace my username `organizedfellow` with your own)_
```
<Directory /home/organizedfellow/www/>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```
Don't save yet, go to last line of `apache.conf` and add this:
```
ServerName 127.0.0.1
```
Now save & exit.

Before we can view our `project1.webdev` in your browser, we need to enable it.

But first, let's see what is currently available.
```
ls /etc/apache2/sites-available
```
> OUTPUT:
>```
>000-default.conf  default-ssl.conf  project1.webdev.conf
>```
If you run `sites-enabled`
```
ls /etc/apache2/sites-enabled
```
> OUTPUT:
>```
>000-default.conf
>```
Let's enable our project
```
sudo a2ensite project1.webdev.conf
```
And then disable the default `000-default.conf`.
```
sudo a2dissite 000-default.conf
```
Restart apache
```
sudo systemctl reload apache2
```
Visit your project in your browser http://project1.webdev

[top](#LAMP)
---
## ADMINER

> #### What is Adminer?
> _Adminer (formerly phpMinAdmin) is a full-featured database management tool written in PHP. Conversely to phpMyAdmin, it consist of a single file ready to deploy to the target server. Adminer is available for MySQL, MariaDB, PostgreSQL, SQLite, MS SQL, Oracle, Firebird, SimpleDB, Elasticsearch and MongoDB._

Let's install adminer and give it its own VirtualHost.
```
sudo tee -a /etc/apache2/sites-available/adminer.webdev.conf << END
<VirtualHost *:80>
	ServerAdmin webmaster@adminer.webdev
	ServerName adminer.webdev
	ServerAlias www.adminer.webdev
	DocumentRoot /home/organizedfellow/www/adminer.webdev/public_html/
	ErrorLog /home/organizedfellow/www/adminer.webdev/logs/error.log
	CustomLog /home/organizedfellow/www/adminer.webdev/logs/access.log combined
    <Directory "/home/organizedfellow/www/adminer.webdev/public_html">
        AllowOverride all
        Options Indexes FollowSymLinks
        Require local
    </Directory>
</VirtualHost>
END
```
Let's create the folder structure for adminer.
```
mkdir -p ~/www/adminer.webdev/public_html/
mkdir -p ~/www/adminer.webdev/logs/
```
Now we can edit `hosts` file
```
sudo echo "127.0.0.1 www.adminer.webdev adminer.webdev" | sudo tee -a /etc/hosts
```
Now if you run `cat /etc/hosts` you should see that both `project1.webdev` and `adminer.webdev` are added at the bottom.
Now we get to actually installing adminer. It's super simple as it is just a single page php file.
As of today (2019-04-04) the current version is `4.7.1`.
```
cd ~/www/adminer.webdev/public_html
```
```
wget https://github.com/vrana/adminer/releases/download/v4.7.1/adminer-4.7.1-mysql-en.php
mv adminer-4.7.1-mysql-en.php index.php
```
Enable adminer.
```
sudo a2ensite adminer.webdev.conf
```
And restart apache.
```
sudo service apache2 restart
```
You can now log in to Adminer with the username and password you set up above when installing and securing MariaDB.

[top](#LAMP)
---
## PERMISSIONS
Before we give apache ownership of `~/www` we add `$USER` to the group `www-data` _(don't forget to replace my username `organizedfellow` with your own)_.
```
sudo usermod -a -G www-data organizedfellow
```
Then give apache ownership
```
sudo chgrp www-data ~/www
```
Now gives members of `www-data` group read, write and directory rights.
```
sudo chmod g+rwxs ~/www
```
[top](#LAMP)
---
## ToDo

- [ ] install an easy to use development SMTP server for localhost testing
  - [ ] a quick google got me this using postfix https://gist.github.com/raelgc/6031274
- [ ] install localhost certificate to enable `https`
  - [ ] a quick google got me this [How to get HTTPS working on your local development environment in 5 minutes](https://medium.freecodecamp.org/how-to-get-https-working-on-your-local-development-environment-in-5-minutes-7af615770eec) and this [Certificates for localhost](https://letsencrypt.org/docs/certificates-for-localhost/)
- [ ] install nodejs and npm, maybe even composer

---
```
This work is free. You can redistribute it and/or modify it under the
terms of the Do What The Fuck You Want To Public License, Version 2,
as published by Sam Hocevar. See http://www.wtfpl.net/ for more details.
```
