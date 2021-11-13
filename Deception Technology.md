# Практическая работа №3 - Deception Technology

**Авторы: Миноцкий Ян, Носков Сергей**

**Задача: с помощью honeypot собрать содержательную информацию о злоумышленнике** 

## Использование honeypot

После клонирования репозитория запускаем демона docker. Получим следующий результат:

![](https://i.imgur.com/pTVvKZE.png)

Переходим в соседнюю вкладку и запустим симулятор действий хакера

```
chmod +x hackersActivity
./hackersActivity
```

Команда `chmod` необходима для выдачи различных прав доступа к файлам. В данном случае мы предоставили права на выполнение скрипта

Логи получаем с помощью команды `docker-compose logs -f`. Наш honeypot выявил следующие подозрительные действия. 

![](https://i.imgur.com/w6W6eko.png)

Злоумышленники используют telnet, ssh, http, redis и ftp. Разберём каждый в отдельности.


### Telnet

Порт: 8023

**Telnet** - это сетевая утилита, которая позволяет соединиться с удаленным портом любого компьютера и установить интерактивный канал связи, например, для передачи команд или получения информации.

```
honeypot_1  | services > telnet > category=telnet, date=2021-11-02 04:19:33.65527075 +0000 UTC m=+222.991966089, destination-ip=172.21.0.2, destination-port=8023, sensor=services, source-ip=172.21.0.1, source-port=46700, telnet.sessionid=c60bolcs0ev000bfsudg, token=c60bmtks0ev000bfsucg, type=connect
honeypot_1  | services > telnet > category=telnet, date=2021-11-02 04:20:20.9652506 +0000 UTC m=+270.301946088, destination-ip=172.21.0.2, destination-port=8023, sensor=services, source-ip=172.21.0.1, source-port=48722, telnet.sessionid=c60bp14s0ev000bfsuf0, token=c60bmtks0ev000bfsucg, type=connect
honeypot_1  | services > telnet > category=telnet, date=2021-11-02 04:20:20.970142214 +0000 UTC m=+270.306837616, destination-ip=172.21.0.2, destination-port=8023, sensor=services, source-ip=172.21.0.1, source-port=48722, telnet.password=root, telnet.sessionid=c60bp14s0ev000bfsuf0, telnet.username=root, token=c60bmtks0ev000bfsucg, type=password-authentication
honeypot_1  | services > telnet > category=telnet, date=2021-11-02 04:20:20.97042436 +0000 UTC m=+270.307119765, destination-ip=172.21.0.2, destination-port=8023, sensor=services, source-ip=172.21.0.1, source-port=48722, telnet.command=system-view, telnet.sessionid=c60bp14s0ev000bfsuf0, token=c60bmtks0ev000bfsucg, type=session
honeypot_1  | services > telnet > category=telnet, date=2021-11-02 04:20:25.965201056 +0000 UTC m=+275.301896399, destination-ip=172.21.0.2, destination-port=8023, sensor=services, source-ip=172.21.0.1, source-port=48722, telnet.command=display ip vpn-instance, telnet.sessionid=c60bp14s0ev000bfsuf0, token=c60bmtks0ev000bfsucg, type=session
honeypot_1  | services > telnet > category=telnet, date=2021-11-02 04:20:30.966280586 +0000 UTC m=+280.302975864, destination-ip=172.21.0.2, destination-port=8023, sensor=services, source-ip=172.21.0.1, source-port=48722, telnet.command=display arp interface GigabitEthernet 0/0/1, telnet.sessionid=c60bp14s0ev000bfsuf0, token=c60bmtks0ev000bfsucg, type=session
honeypot_1  | services > telnet > category=telnet, date=2021-11-02 04:20:35.967772482 +0000 UTC m=+285.304467762, destination-ip=172.21.0.2, destination-port=8023, sensor=services, source-ip=172.21.0.1, source-port=48722, telnet.command=display ipsec-tunnel limit vpn-instance vpna, telnet.sessionid=c60bp14s0ev000bfsuf0, token=c60bmtks0ev000bfsucg, type=session
honeypot_1  | services > telnet > category=telnet, date=2021-11-02 04:20:40.968001468 +0000 UTC m=+290.304696745, destination-ip=172.21.0.2, destination-port=8023, sensor=services, source-ip=172.21.0.1, source-port=48722, telnet.command=display snmp-server arp-sync table, telnet.sessionid=c60bp14s0ev000bfsuf0, token=c60bmtks0ev000bfsucg, type=session
```



* ` system-view` - просмотр данных о системе
* `display ip vpn-instance` - просмотр VPN-Instance

[Часть статьи на habr.com про VPN-Instance](https://habr.com/ru/post/273679/#VRF,%20VPN-Instance,%20Routing-instance)  

`display arp interface GigabitEthernet 0/0/1` - просмотр MAC-адреса устройства подключённого к интерфейсу `GigabitEthernet 0/0/1`

**ARP (англ. Address Resolution Protocol — протокол определения адреса)** — протокол в компьютерных сетях, предназначенный для определения MAC-адреса по IP-адресу другого компьютера.

[Хорошая статья на habr.com по ARP таблицам](https://habr.com/ru/post/80364/)

`display ipsec-tunnel limit vpn-instance vpna` - просмотр ограничений для IPsec соединения

**Internet Protocol Security (IPsec)** — это набор протоколов для обеспечения защиты данных, передаваемых по IP-сети. В отличие от SSL, который работает на [прикладном уровне](https://github.com/Yan-Minotskiy/network_config/blob/main/iptables.md#%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D1%8C-osi), IPsec работает на [сетевом уровне](https://github.com/Yan-Minotskiy/network_config/blob/main/iptables.md#%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D1%8C-osi) и может использоваться нативно со многими операционными системами, что позволяет использовать его без сторонних приложений (в отличие от [OpenVPN](https://github.com/Yan-Minotskiy/network_config/blob/main/OpenVPN-L3.md)). IPsec стал очень популярным протоколом для использования в паре с [L2TP](https://github.com/Yan-Minotskiy/network_config/blob/main/L2TP.md) или [IKEv2](https://github.com/Yan-Minotskiy/network_config/blob/main/IKEv2.md).

`display snmp-server arp-sync table` - просмотр таблиц ARP c помощью протокола SNMP

**SNMP (англ. Simple Network Management Protocol — простой протокол сетевого управления)** — стандартный интернет-протокол для управления устройствами в IP-сетях на основе архитектур TCP/UDP. 



### SSH
Порт: 8022

**SSH (англ. Secure Shell — «безопасная оболочка»)** — сетевой протокол прикладного уровня, позволяющий производить удалённое управление операционной системой и туннелирование TCP-соединений (например, для передачи файлов). Схож по функциональности с протоколами Telnet и rlogin, но, в отличие от них, шифрует весь трафик, включая и передаваемые пароли. SSH допускает выбор различных алгоритмов шифрования. SSH-клиенты и SSH-серверы доступны для большинства сетевых операционных систем.

```
honeypot_1  | services > ssh > category=ssh, date=2021-11-02 04:19:45.913453892 +0000 UTC m=+235.250149297, destination-ip=172.21.0.2, destination-port=8022, sensor=services, source-ip=172.21.0.1, source-port=44804, ssh.password=root, ssh.sessionid=c60boocs0ev000bfsueg, ssh.username=root, token=c60bmtks0ev000bfsucg, type=password-authentication
honeypot_1  | services > ssh > category=ssh, date=2021-11-02 04:19:45.91895143 +0000 UTC m=+235.255646771, destination-ip=172.21.0.2, destination-port=8022, sensor=services, source-ip=172.21.0.1, source-port=44804, ssh.exec=[]string{"pwd"}, ssh.payload=[]byte{0x0, 0x0, 0x0, 0x3, 0x70, 0x77, 0x64}, ssh.request-type=exec, ssh.sessionid=c60boocs0ev000bfsueg, token=c60bmtks0ev000bfsucg, type=ssh-request
honeypot_1  | services > ssh > category=ssh, date=2021-11-02 04:19:50.927430769 +0000 UTC m=+240.264126186, destination-ip=172.21.0.2, destination-port=8022, sensor=services, source-ip=172.21.0.1, source-port=44804, ssh.exec=[]string{"cat /etc/shadow"}, ssh.payload=[]byte{0x0, 0x0, 0x0, 0xf, 0x63, 0x61, 0x74, 0x20, 0x2f, 0x65, 0x74, 0x63, 0x2f, 0x73, 0x68, 0x61, 0x64, 0x6f, 0x77}, ssh.request-type=exec, ssh.sessionid=c60boocs0ev000bfsueg, token=c60bmtks0ev000bfsucg, type=ssh-request
honeypot_1  | services > ssh > category=ssh, date=2021-11-02 04:19:55.936456174 +0000 UTC m=+245.273151654, destination-ip=172.21.0.2, destination-port=8022, sensor=services, source-ip=172.21.0.1, source-port=44804, ssh.exec=[]string{"ls -la"}, ssh.payload=[]byte{0x0, 0x0, 0x0, 0x6, 0x6c, 0x73, 0x20, 0x2d, 0x6c, 0x61}, ssh.request-type=exec, ssh.sessionid=c60boocs0ev000bfsueg, token=c60bmtks0ev000bfsucg, type=ssh-request
honeypot_1  | services > ssh > category=ssh, date=2021-11-02 04:20:00.946118731 +0000 UTC m=+250.282814211, destination-ip=172.21.0.2, destination-port=8022, sensor=services, source-ip=172.21.0.1, source-port=44804, ssh.exec=[]string{"ss -tunlp"}, ssh.payload=[]byte{0x0, 0x0, 0x0, 0x9, 0x73, 0x73, 0x20, 0x2d, 0x74, 0x75, 0x6e, 0x6c, 0x70}, ssh.request-type=exec, ssh.sessionid=c60boocs0ev000bfsueg, token=c60bmtks0ev000bfsucg, type=ssh-request
honeypot_1  | services > ssh > category=ssh, date=2021-11-02 04:20:05.956234005 +0000 UTC m=+255.292929362, destination-ip=172.21.0.2, destination-port=8022, sensor=services, source-ip=172.21.0.1, source-port=44804, ssh.exec=[]string{"whoami"}, ssh.payload=[]byte{0x0, 0x0, 0x0, 0x6, 0x77, 0x68, 0x6f, 0x61, 0x6d, 0x69}, ssh.request-type=exec, ssh.sessionid=c60boocs0ev000bfsueg, token=c60bmtks0ev000bfsucg, type=ssh-request
```

Из логов мы можем понять, что произошла аутентификация по SSH в нашей системе, а затем выполнялись следующие команды.

* `pwd` - вывод пути к текущей директории
* `cat /etc/shadow` - чтение текстового файла, содержащего информацию о паролях пользователей системы
* `ls -la` - вывод подробного содержания директории
* `ss -tunlp` - просмотр сетевых подключений
* `whoami` - отображение имени текущего пользователя

Рекомендации:

1. Запретить подключение к системе от имени администратора. Обычно запрет идёт по умолчанию, но если нет, его можно поставить установив конфигурацию `PermitRootLogin no` в файле `/etc/ssh/sshd_config`, а затем применить изменения `systemctl restart ssh`. (Выполнять все эти действия необходимо с правами sudo.)
2. Запретить подключение по ssh, если в этом нет необходимости.
3. Устанавливать сложные пароли для пользователей.

**Лабораторные работы на данную тематику:**
[Лабораторная работа по сетям и системам передачи данных](https://github.com/Yan-Minotskiy/network_config/blob/main/SSH%2C%20NAT.md#SSH)
[Лабораторная работа по организации сетевой безопасности](https://github.com/Yan-Minotskiy/network_config/blob/main/Linux_Pt2.md#SSH)

### HTTP

Порт: 8080

**HTTP (англ. HyperText Transfer Protocol — «протокол передачи гипертекста»)** — протокол [прикладного уровня передачи данных](https://github.com/Yan-Minotskiy/network_config/blob/main/iptables.md#%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D1%8C-osi), изначально — в виде гипертекстовых документов в формате HTML, в настоящее время используется для передачи произвольных данных.

```
honeypot_1  | services > http > category=http, date=2021-11-02 04:20:55.974308041 +0000 UTC m=+305.311003332, destination-ip=172.21.0.2, destination-port=8080, http.header.accept-encoding=[]string{"gzip"}, http.header.user-agent=[]string{"Go-http-client/1.1"}, http.host=localhost, http.method=GET, http.proto=HTTP/1.1, http.sessionid=c60bp9ss0ev000bfsufg, http.url=/robots.txt, payload-hex=, payload-length=0, payload=, sensor=services, source-ip=172.21.0.1, source-port=33296, token=c60bmtks0ev000bfsucg, type=request
honeypot_1  | services > http > category=http, date=2021-11-02 04:21:00.978920622 +0000 UTC m=+310.315615983, destination-ip=172.21.0.2, destination-port=8080, http.header.accept-encoding=[]string{"gzip"}, http.header.user-agent=[]string{"Go-http-client/1.1"}, http.host=localhost, http.method=GET, http.proto=HTTP/1.1, http.sessionid=c60bp9ss0ev000bfsufg, http.url=/login.php, payload-hex=, payload-length=0, payload=, sensor=services, source-ip=172.21.0.1, source-port=33296, token=c60bmtks0ev000bfsucg, type=request
honeypot_1  | services > http > category=http, date=2021-11-02 04:21:05.982085543 +0000 UTC m=+315.318780861, destination-ip=172.21.0.2, destination-port=8080, http.header.accept-encoding=[]string{"gzip"}, http.header.user-agent=[]string{"Go-http-client/1.1"}, http.host=localhost, http.method=GET, http.proto=HTTP/1.1, http.sessionid=c60bp9ss0ev000bfsufg, http.url=/admin.php, payload-hex=, payload-length=0, payload=, sensor=services, source-ip=172.21.0.1, source-port=33296, token=c60bmtks0ev000bfsucg, type=request
honeypot_1  | services > http > category=http, date=2021-11-02 04:21:10.986040815 +0000 UTC m=+320.322736297, destination-ip=172.21.0.2, destination-port=8080, http.header.accept-encoding=[]string{"gzip"}, http.header.user-agent=[]string{"Go-http-client/1.1"}, http.host=localhost, http.method=GET, http.proto=HTTP/1.1, http.sessionid=c60bp9ss0ev000bfsufg, http.url=/admin, payload-hex=, payload-length=0, payload=, sensor=services, source-ip=172.21.0.1, source-port=33296, token=c60bmtks0ev000bfsucg, type=request
honeypot_1  | services > http > category=http, date=2021-11-02 04:21:15.991070372 +0000 UTC m=+325.327765682, destination-ip=172.21.0.2, destination-port=8080, http.header.accept-encoding=[]string{"gzip"}, http.header.user-agent=[]string{"Go-http-client/1.1"}, http.host=localhost, http.method=GET, http.proto=HTTP/1.1, http.sessionid=c60bp9ss0ev000bfsufg, http.url=/sitemap.xml, payload-hex=, payload-length=0, payload=, sensor=services, source-ip=172.21.0.1, source-port=33296, token=c60bmtks0ev000bfsucg, type=request
```

Из логов мы понимем, что злоумышленник отправлял GET запросы следующим страницам:

* `robots.txt` - это документ в формате `.txt`, содержащий инструкции по индексации конкретного сайта для поисковых ботов. Он указывает поисковикам, какие страницы веб-ресурса стоит проиндексировать, а какие не нужно допустить к индексации. Изменения в этом файле могут повлиять на положение сайта в поисковых системах.
* `login.php` - скорее всего страница авторизации
* `admin.php` - скорее всего панель администратора
* `sitemap.xml` - XML-файлы с информацией для поисковых систем (также для индексирования). Sitemaps могут помочь поисковикам определить местонахождение страниц сайта, время их последнего обновления, частоту обновления и важность относительно других страниц сайта для того, чтобы поисковая машина смогла более разумно индексировать сайт. Изменения в этом файле могут повлиять на положение сайта в поисковых системах.

Рекомендации: 
1. Использовать протокол HTTPS (расширение для http  с поддержкой шифрования)
2. Усилить защиту от несанкцианированного доступа к файлам сайта


### Redis

Порт: 6379

**Redis (от англ. remote dictionary server)** — резидентная система управления базами данных класса NoSQL с открытым исходным кодом, работающая со структурами данных типа «ключ — значение». Используется как для баз данных, так и для реализации кэшей, брокеров сообщений.

```
honeypot_1  | services > redis > category=redis, date=2021-11-02 04:21:30.996967824 +0000 UTC m=+340.333663156, destination-ip=172.21.0.2, destination-port=6379, redis.command=CONFIG GET *, sensor=services, source-ip=172.21.0.1, source-port=39696, token=c60bmtks0ev000bfsucg, type=redis-command
honeypot_1  | services > redis > category=redis, date=2021-11-02 04:21:36.000865282 +0000 UTC m=+345.337560921, destination-ip=172.21.0.2, destination-port=6379, redis.command=GET admin, sensor=services, source-ip=172.21.0.1, source-port=39696, token=c60bmtks0ev000bfsucg, type=redis-command
honeypot_1  | services > redis > category=redis, date=2021-11-02 04:21:41.00371519 +0000 UTC m=+350.340410551, destination-ip=172.21.0.2, destination-port=6379, redis.command=INFO, sensor=services, source-ip=172.21.0.1, source-port=39696, token=c60bmtks0ev000bfsucg, type=redis-command
honeypot_1  | services > redis > category=redis, date=2021-11-02 04:21:46.004756427 +0000 UTC m=+355.341451732, destination-ip=172.21.0.2, destination-port=6379, redis.command=SELECT 1, sensor=services, source-ip=172.21.0.1, source-port=39696, token=c60bmtks0ev000bfsucg, type=redis-command
honeypot_1  | services > redis > category=redis, date=2021-11-02 04:21:51.005990131 +0000 UTC m=+360.342685459, destination-ip=172.21.0.2, destination-port=6379, redis.command=client list, sensor=services, source-ip=172.21.0.1, source-port=39696, token=c60bmtks0ev000bfsucg, type=redis-command
```

Были использованы следующие команды:

`CONFIG GET *` - чтения параметров конфигурации работающего сервера Redis
`GET admin` - получение значения по ключу
`INFO` - возвращение информации и статистики о сервере
`SELECT 1` - получение информации из базы
`client list` - вывод списка клиентов

[Официальный сайт Redis с документацией](https://redis.com/)

### FTP

Порт: 8021

**FTP (англ. File Transfer Protocol)** — протокол передачи файлов по сети, появившийся в 1971 году задолго до HTTP и даже до TCP/IP, благодаря чему является одним из старейших прикладных протоколов. Изначально FTP работал поверх протокола NCP, на сегодняшний день широко используется для распространения ПО и доступа к удалённым хостам.

```
honeypot_1  | services > ftp > category=ftp, date=2021-11-02 04:22:06.009213363 +0000 UTC m=+375.345908655, destination-ip=172.21.0.2, destination-port=8021, ftp.command=USER anonymous, ftp.sessionid=7690e335eda59e8d76f8, sensor=services, source-ip=172.21.0.1, source-port=38394, token=c60bmtks0ev000bfsucg
honeypot_1  | services > ftp > category=ftp, date=2021-11-02 04:22:06.010018524 +0000 UTC m=+375.346713801, destination-ip=172.21.0.2, destination-port=8021, ftp.command=PASS anonymous, ftp.sessionid=c51accd883e4b8969054, sensor=services, source-ip=172.21.0.1, source-port=40436, token=c60bmtks0ev000bfsucg
honeypot_1  | services > ftp > category=ftp, date=2021-11-02 04:22:06.010782392 +0000 UTC m=+375.347477664, destination-ip=172.21.0.2, destination-port=8021, ftp.command=FEAT, ftp.sessionid=7690e335eda59e8d76f8, sensor=services, source-ip=172.21.0.1, source-port=38394, token=c60bmtks0ev000bfsucg
honeypot_1  | services > ftp > category=ftp, date=2021-11-02 04:22:06.011231 +0000 UTC m=+375.347926280, destination-ip=172.21.0.2, destination-port=8021, ftp.command=HELP, ftp.sessionid=c51accd883e4b8969054, sensor=services, source-ip=172.21.0.1, source-port=40436, token=c60bmtks0ev000bfsucg
honeypot_1  | services > ftp > category=ftp, date=2021-11-02 04:22:11.01498988 +0000 UTC m=+380.351685251, destination-ip=172.21.0.2, destination-port=8021, ftp.command=STOR /etc/passwd, ftp.sessionid=7690e335eda59e8d76f8, sensor=services, source-ip=172.21.0.1, source-port=38394, token=c60bmtks0ev000bfsucg
honeypot_1  | services > ftp > category=ftp, date=2021-11-02 04:22:16.017774406 +0000 UTC m=+385.354469881, destination-ip=172.21.0.2, destination-port=8021, ftp.command=LIST -l, ftp.sessionid=c51accd883e4b8969054, sensor=services, source-ip=172.21.0.1, source-port=40436, token=c60bmtks0ev000bfsucg
honeypot_1  | services > ftp > category=ftp, date=2021-11-02 04:22:21.018474481 +0000 UTC m=+390.355169784, destination-ip=172.21.0.2, destination-port=8021, ftp.command=PWD, ftp.sessionid=7690e335eda59e8d76f8, sensor=services, source-ip=172.21.0.1, source-port=38394, token=c60bmtks0ev000bfsucg
honeypot_1  | services > ftp > category=ftp, date=2021-11-02 04:22:26.019060811 +0000 UTC m=+395.355756097, destination-ip=172.21.0.2, destination-port=8021, ftp.command=APPE /exploit.sh, ftp.sessionid=c51accd883e4b8969054, sensor=services, source-ip=172.21.0.1, source-port=40436, token=c60bmtks0ev000bfsucg
```

`USER anonymous` - ввод имени пользователя при авторизации
`PASS anonymous` - ввод пароля пользователя при авторизации
`FEAT` - предоставление механизма быстрого определения того, какие расширенные функции  поддерживает FTP-сервер
`HELP` - получения информации о реализации протокола FTP-сервером, возвращат списка поддерживаемых (или распознанных) команд
`STOR` - загрузка копии локального файла на сервер
`LIST -l` - вывод информации о файлах на сервере
`APPE /exploit.sh` - загрузка файла на сервер

[Команды FTP. Часть 1.](https://www.serv-u.com/resource/tutorial/feat-opts-help-stat-nlst-xcup-xcwd-ftp-command)
[Команды FTP. Часть 2.](https://www.serv-u.com/resource/tutorial/appe-stor-stou-retr-list-mlsd-mlst-ftp-command)

## Использование nmap для сканирования сети

Утилита `nmap` в процессе сканирования сети перебирает доступный диапазон портов и пытается подключиться к каждому из них. Если подключение удалось, в большинстве случаев, передав несколько пакетов программа может даже узнать версию программного обеспечения, которые ожидает подключений к этому порту.

Основные флаги команды `nmap`:
`-sL` - просто создать список работающих хостов, но не сканировать порты nmap;
`-sP` - только проверять доступен ли хост с помощью ping;
`-PN` - считать все хосты доступными, даже если они не отвечают на ping;
`-sS/sT/sA/sW/sM` - TCP сканирование;
`-sU` - UDP сканирование nmap;
`-sN/sF/sX` - TCP NULL и FIN сканирование;
`-sC` - запускать скрипт по умолчанию;
`-sI` - ленивое Indle сканирование;
`-p` - указать диапазон портов для проверки;
`-sV` - детальное исследование портов для определения версий служб;
`-O` - определять операционную систему;
`-T[0-5]` - скорость сканирования, чем больше, тем быстрее;
`-D` - маскировать сканирование с помощью фиктивных IP;
`-S` - изменить свой IP адрес на указанный;
`-e` - использовать определенный интерфейс;
`--spoof-mac` - установить свой MAC адрес;
`-A` - определение операционной системы с помощью скриптов.

Установка `nmap`

`apt-get install nmap`

Сканирование портов nmap для нужного узла запустив утилиту без опций после обнаружения активных устройств в сети.

Вывод информации о развёрнутых сервисах honeypot

![](https://i.imgur.com/5HcSEON.png)

Сканировакние сервисов злоумшленника

![](https://i.imgur.com/b98jCcV.png)

## Выводы о злоумышленнике на основе полученной информации 
Попытки получения доступа к honeypot были через telnet и ssh. Далее хакеры собирали информацию о структуре сайта и о содержании базы данных. Далее используя протокол FTP передали вредоносный файл в систему. В ходе сканирования утилитой `nmap` был выявлен MAC-адрес злоумышленника: `02:42:AC:15:00:02`

В данной ситуации сотрудникам отдела информационной безопасности необходимо проверить надёжность конфеденциального доступа к системе через ssh, проверить настройки доступа к файлам сайта.

## Distributed  Deception Platform


На текущий момент компании всё реже используют одиночные honepot, сейчас используют целые комплексы ложных целей. Платформы для создания распределенной инфраструктуры ложных целей (Distributed  Deception Platform — DDP) являются развитием идеи ханипотов и создают поддельную ИТ-инфраструктуру, привлекательную для злоумышленников. Ложные цели помогают обнаружить присутствие злоумышленников в сети, если им удалось успешно обойти все средства защиты, и отвлекают их внимание от основных ресурсов предприятия.


## Источники

1. [Инструкция по выполнению лабораторной работы](https://hackmd.io/@ivanh/r1W-l-uz_)
2. [Отчёты по лабораторным работам по сетям и системам передачи данных, по организации компьютерных сетей и по организации сетевой безопасности](https://github.com/Yan-Minotskiy/network_config)
3. [Как пользоваться Nmap для сканирования сети](https://losst.ru/kak-polzovatsya-nmap-dlya-skanirovaniya)
4. [Сети для самых маленьких. Часть одиннадцатая. MPLS L3VPN | Habr](https://habr.com/ru/post/273679)
5. [Протокол ARP и «с чем его едят»](https://habr.com/ru/post/80364/)
6. [Официальный сайт Redis с документацией](https://redis.com/)
7. [FTP Commands: FEAT, OPTS, HELP, STAT, NLST, XCUP, XCWD](https://www.serv-u.com/resource/tutorial/feat-opts-help-stat-nlst-xcup-xcwd-ftp-command)
8. [FTP Commands: APPE, MLSD, MLST, LIST, RETR, STOR, STOU](https://www.serv-u.com/resource/tutorial/appe-stor-stou-retr-list-mlsd-mlst-ftp-command)
9. [Обзор рынка платформ для создания распределенной инфраструктуры ложных целей (Distributed Deception Platform)](https://www.anti-malware.ru/analytics/Market_Analysis/Distributed-Deception-Platform)
