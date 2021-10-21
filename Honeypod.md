![](screens/Снимок%20экрана%202021-09-24%20062050.png)

`apt-get install nmap`

Сканирование портов nmap для нужного узла запустив утилиту без опций после обнаружения активных устройств в сети.

![](screens/Снимок%20экрана%202021-09-24%20063802.png)

![](screens/Снимок%20экрана%202021-09-24%20064740.png)


```
PORT     STATE SERVICE    REASON
8021/tcp open  ftp-proxy  syn-ack ttl 64
8022/tcp open  oa-system  syn-ack ttl 64
8080/tcp open  http-proxy syn-ack ttl 64
```

```
nmap -A 172.21.0.2
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-23 20:50 PDT
Nmap scan report for 172.21.0.2
Host is up (0.00023s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE    VERSION
8021/tcp open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 Welcome to the Go FTP Server
|     Command not found.
|     Command not found.
|   Help, SSLSessionReq: 
|     220 Welcome to the Go FTP Server
|     Command not found.
|   NULL, SMBProgNeg: 
|_    220 Welcome to the Go FTP Server
| ssl-cert: Subject: organizationName=/countryName=
| Not valid before: 2021-09-24T03:17:38
|_Not valid after:  2022-09-24T03:17:38
|_ssl-date: TLS randomness does not represent time
8022/tcp open  ssh        (protocol 2.0)
| fingerprint-strings: 
|   NULL: 
|_    SSH-2.0-OpenSSH_6.6.1p1 2020Ubuntu-2ubuntu2
| ssh-hostkey: 
|_  2048 67:8c:9d:32:e8:a0:1a:f4:0e:eb:e4:ac:95:dc:13:db (RSA)
8080/tcp open  http-proxy Nginx
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest, HTTPOptions: 
|     HTTP/1.0 200 OK
|     Server: Nginx
|_    Content-Length: 0
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Nginx
|_http-title: Site doesn't have a title.
3 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port8021-TCP:V=7.80%I=7%D=9/23%Time=614D4AF1%P=x86_64-pc-linux-gnu%r(NU
SF:LL,22,"220\x20Welcome\x20to\x20the\x20Go\x20FTP\x20Server\r\n")%r(Gener
SF:icLines,52,"220\x20Welcome\x20to\x20the\x20Go\x20FTP\x20Server\r\n500\x
SF:20Command\x20not\x20found\.\r\n500\x20Command\x20not\x20found\.\r\n")%r
SF:(Help,3A,"220\x20Welcome\x20to\x20the\x20Go\x20FTP\x20Server\r\n500\x20
SF:Command\x20not\x20found\.\r\n")%r(SSLSessionReq,3A,"220\x20Welcome\x20t
SF:o\x20the\x20Go\x20FTP\x20Server\r\n500\x20Command\x20not\x20found\.\r\n
SF:")%r(SMBProgNeg,22,"220\x20Welcome\x20to\x20the\x20Go\x20FTP\x20Server\
SF:r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port8022-TCP:V=7.80%I=7%D=9/23%Time=614D4AF1%P=x86_64-pc-linux-gnu%r(NU
SF:LL,2D,"SSH-2\.0-OpenSSH_6\.6\.1p1\x202020Ubuntu-2ubuntu2\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port8080-TCP:V=7.80%I=7%D=9/23%Time=614D4AF6%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,35,"HTTP/1\.0\x20200\x20OK\r\nServer:\x20Nginx\r\nContent-Leng
SF:th:\x200\r\n\r\n")%r(HTTPOptions,35,"HTTP/1\.0\x20200\x20OK\r\nServer:\
SF:x20Nginx\r\nContent-Length:\x200\r\n\r\n")%r(FourOhFourRequest,35,"HTTP
SF:/1\.0\x20200\x20OK\r\nServer:\x20Nginx\r\nContent-Length:\x200\r\n\r\n"
SF:);
MAC Address: 02:42:AC:15:00:02 (Unknown)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=9/23%OT=8021%CT=1%CU=31702%PV=Y%DS=1%DC=D%G=Y%M=0242AC
OS:%TM=614D4B80%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=107%TI=Z%CI=Z%II
OS:=I%TS=A)OPS(O1=M5B4ST11NW7%O2=M5B4ST11NW7%O3=M5B4NNT11NW7%O4=M5B4ST11NW7
OS:%O5=M5B4ST11NW7%O6=M5B4ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%
OS:W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M5B4NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S
OS:=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%R
OS:D=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=
OS:0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U
OS:1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DF
OS:I=N%T=40%CD=S)

Network Distance: 1 hop
Service Info: Host: Welcome

TRACEROUTE
HOP RTT     ADDRESS
1   0.23 ms 172.21.0.2

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 150.02 seconds

```

Источники:

https://losst.ru/kak-polzovatsya-nmap-dlya-skanirovaniya