# Практическая работа №1 - Phishing
###### tags: `Методы и средства защиты информации`
**Авторы: Миноцкий Ян, Носков Сергей**
**Задача: При помощи инструмента Gophish провести фишинговую рассылку на локальном почтовом сервере**
## Подготовка
Для начала необходимо загрузить с GitHub репозиторий
```
git clone https://github.com/Ivanhahanov/InformationSecurityMethodsAndTools.git
```
Переходим в директорию
```
cd InformationSecurityMethodsAndTools/Phishing
```
## Проверка работоспособности (можно пропустить)
Флаг ```-d``` запускает в фоновом режиме, и вы не будете видеть логи. Сейчас с ресурса DockerHub “спулятся” образы почтового сервера и приложения для фишинга
```
docker-compose up -d
```

[DockerHub](https://hub.docker.com/) - это аналог Github для образов docker образов, вы можете туда загружать свои образы, а можете загружать чужие. Все образы имеют рейтинг, вы можете узнать сколько раз образ загружали и сколько ему поставили звёздочек. Это полезная информация, так вы можете понять, насколько тот или иной образ популярен, и решить, стоит ли загружать именно его или поискать другую реализацию.

Для просмотра логов:
```
docker-compose logs -f
```
Почтовый сервер должен сообщить об ошибках, так как у нас пока нет пользователей и нет подписанного сертификата. Так что можно выполнить эту команду:
```
docker-compose down
```
Почтовый сервер плавно завершит работу

## Конфигурация
### Создание сертификатов
Нам необходимо создать самоподписанный [TLS сертификат](https://www.seonews.ru/analytics/sertifikaty-tls-ssl-bazovaya-informatsiya-dlya-vladeltsev-saytov/#:~:text=Сертификаты%20TLS%2FSSL%20–%20файлы%2C%20которые,был%20набран%20в%20строке%20браузера). Такие сертификаты используются для шифрования данных, самый распространённый пример - [HTTPS](https://ru.wikipedia.org/wiki/HTTPS).
В нашем случае он нужен для зашифровки писем, чтобы никто не мог их прочитать при перехвате трафика

В контейнере **docker-mailserver** есть утилита для генерации сертификатов.

Запустим контейнер в интерактивном режиме (флаг `-ti`)
И примонтируем раздел, чтобы сертификат создался у нас в директории MailServer/config/ssl локально
```
docker run -ti --rm -v "$(pwd)"/config/ssl:/tmp/docker-mailserver/ssl -h mail.domain.com -t tvial/docker-mailserver generate-ssl-certificate
```
Создание сертификата происходит в два этапа:
1. создание CA сертификата
2. создание конечного сертификата


Заполняем все необходимое в сертификате. Важно, чтобы пароль содержал цифры, латинские буквы в верхнем и нижнем регистрах, а так же спецсимволы.
Ставим пароль: `Asder15$`
> ВАЖНО! Вводимые поля должны быть идентичны, кроме одного поля Common Name (e.g. server FQDN or YOUR name). Здесь в первым случае надо указать `domain.com`, во втором `mail.domain.com`

> Поля `extra` оставить пустыми

Пример генерации сертификата:
```
CA certificate filename (or enter to create)

Making CA certificate ...
====
openssl req  -new -keyout ./demoCA/private/cakey.pem -out ./demoCA/careq.pem 
Generating a RSA private key
...........................+++++
.................................................................................................................................+++++
writing new private key to './demoCA/private/cakey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:RU
State or Province Name (full name) [Some-State]:Moscow
Locality Name (eg, city) []:Moscow
Organization Name (eg, company) [Internet Widgits Pty Ltd]:My-company
Organizational Unit Name (eg, section) []:Red Team      
Common Name (e.g. server FQDN or YOUR name) []:domain.com
Email Address []:admin@domain.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
==> 0
====
====
openssl ca  -create_serial -out ./demoCA/cacert.pem -days 1095 -batch -keyfile ./demoCA/private/cakey.pem -selfsign -extensions v3_ca  -infiles ./demoCA/careq.pem
Using configuration from /usr/lib/ssl/openssl.cnf
Enter pass phrase for ./demoCA/private/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number:
            5d:ac:b5:3f:cc:b6:26:bf:6a:20:dd:c1:ff:08:a0:be:14:1b:e4:3c
        Validity
            Not Before: Feb  8 13:40:27 2021 GMT
            Not After : Feb  8 13:40:27 2024 GMT
        Subject:
            countryName               = RU
            stateOrProvinceName       = Moscow
            organizationName          = My-company
            organizationalUnitName    = Red Team
            commonName                = domain.com
            emailAddress              = admin@domain.com
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                E9:4F:D1:CC:9D:09:14:A3:9C:23:68:8E:0E:76:9E:35:AE:22:56:61
            X509v3 Authority Key Identifier: 
                keyid:E9:4F:D1:CC:9D:09:14:A3:9C:23:68:8E:0E:76:9E:35:AE:22:56:61

            X509v3 Basic Constraints: critical
                CA:TRUE
Certificate is to be certified until Feb  8 13:40:27 2024 GMT (1095 days)

Write out database with 1 new entries
Data Base Updated
==> 0
====
CA certificate is in ./demoCA/cacert.pem
Ignoring -days; not generating a certificate
Generating a RSA private key
..................................................+++++
..............................................................+++++
writing new private key to '/tmp/docker-mailserver/ssl/mail.domain.com-key.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:RU
State or Province Name (full name) [Some-State]:Moscow
Locality Name (eg, city) []:Moscow
Organization Name (eg, company) [Internet Widgits Pty Ltd]:My-company
Organizational Unit Name (eg, section) []:Red Team       
Common Name (e.g. server FQDN or YOUR name) []:mail.domain.com
Email Address []:admin@domain.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
Using configuration from /usr/lib/ssl/openssl.cnf
Enter pass phrase for ./demoCA/private/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number:
            5d:ac:b5:3f:cc:b6:26:bf:6a:20:dd:c1:ff:08:a0:be:14:1b:e4:3d
        Validity
            Not Before: Feb  8 13:41:55 2021 GMT
            Not After : Feb  8 13:41:55 2022 GMT
        Subject:
            countryName               = RU
            stateOrProvinceName       = Moscow
            organizationName          = My-company
            organizationalUnitName    = Red Team
            commonName                = mail.domain.com
            emailAddress              = admin@domain.com
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                2F:6B:63:BE:40:11:03:4E:EC:E7:27:9E:E7:F8:3B:A8:82:9C:84:D9
            X509v3 Authority Key Identifier: 
                keyid:E9:4F:D1:CC:9D:09:14:A3:9C:23:68:8E:0E:76:9E:35:AE:22:56:61

Certificate is to be certified until Feb  8 13:41:55 2022 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
```

Пересобираем контейнеры командой `docker-compose up --build`

Флаг `--build` соберёт контейнеры заново, если появились какие-то изменения в файловой структуре или конфигурациях

### Создание пользователей

Наш сертификат готов! Теперь можем переходить к созданию пользователей. Они нам нужны для того, чтобы отправлять письма по электронной почте. Останавливаем наши контейнеры и переходим в деррикторию с почтовым сервером, где и будем создавать пользователей.

Для создания пользователей у нас есть [Bash-скрипт](https://habr.com/ru/company/ruvds/blog/325522/) [setup.sh](http://setup.sh)

```
# Перейти в деррикторию сервера
cd MailServer

# сделать скрипт исполняемым
chmod +x setup.sh

# создание email аккаунтов user/password
./setup.sh -i tvial/docker-mailserver:latest email add admin@domain.com admin123
./setup.sh -i tvial/docker-mailserver:latest email add user@domain.com user123

# список email аккаунтов 
./setup.sh -i tvial/docker-mailserver:latest email list
```
![](https://i.imgur.com/AqdDIbo.jpg)


### Настройка почтового клиета
В качестве почтового клиента возьмём предустановленный [ThunderBird](https://www.thunderbird.net/ru/)

![](https://i.imgur.com/SGLiaZI.jpg)

### Gophish 
#### Запуск
Чтобы запустить утилиту Gophish выполняем следующее:
`sudo docker run --rm -it -p 8080:80 nginx:stable-alpine`
переходим в директорию с проектом 
`cd InformationSecurityMethodsAndTools/Phishing/`
и выполняем команду
```
docker-compose up
```
или
```
sudo docker-compose up -d
```
Открываем ссылку https://localhost:3333
#### Авторизация
Логин `admin`
Пароль достается: `docker-compose logs | grep password` 
В директории: `cd InformationSecurityMethodsAndTools/Phishing/`

![](https://i.imgur.com/mjzayP0.jpg)

Меняем пароль на `987654321`

![](https://i.imgur.com/q3hvAhX.jpg)

#### Sending Profile
Настроили профиль отправителя
![](https://i.imgur.com/wet5IoZ.jpg)
> 
> ip для секции Host получаем с помощью команды:
> ```
> docker exec mail ip a
> ```
> Порт нужно ставить 25
> 

Проверили, работает!
![](https://i.imgur.com/2XsyKYJ.jpg)

#### Landing Page
Настроили фальшивую страницу
Нужно нажать на кнопку “Import Site” и ввести туда адрес любого сайта
![](https://i.imgur.com/aselU8i.jpg)

#### Email template
Создали шаблон письма примерно такого содержания:
> Уважаемый(ая) {{.FirstName}} {{.LastName}}
> В связи с политикой безопасности вам нужно как можно скорее сменить пароль на сайте https://ok.ru/. Войдите в систему по ссылке {{.URL}} а затем смените пароль в личном кабинете, иначе ваш профиль будет заблокирован.
> С уважением, служба техподдержки.

![](https://i.imgur.com/rebBBoK.jpg)

#### Group
Создали группу пользователей, которым будет отправленно письмо
![](https://i.imgur.com/TU5zX8k.jpg)

#### Campaign
Настроили рассылку :)
![](https://i.imgur.com/i2J5VWx.jpg)

На почте видим письмо и, конечно же, переходим по указанной ссылке, да еще и вводим свои данные
![](https://i.imgur.com/V1luA83.jpg)

Видим следующий результат, а значит всё отработало правильно!

![](https://i.imgur.com/5WrIdVA.jpg)

## Источники
1. [Инструкция по выполнению лабораторной работы №1](https://hackmd.io/@ivanh/Hk_giUdxd)
2. [DockerHub](https://hub.docker.com/)
3. [TLS сертификат](https://www.seonews.ru/analytics/sertifikaty-tls-ssl-bazovaya-informatsiya-dlya-vladeltsev-saytov/#:~:text=Сертификаты%20TLS%2FSSL%20–%20файлы%2C%20которые,был%20набран%20в%20строке%20браузера)
4. [HTTPS](https://ru.wikipedia.org/wiki/HTTPS)
5. [Bash-скрипт](https://habr.com/ru/company/ruvds/blog/325522/)
6. [setup.sh](http://setup.sh)
7. [ThunderBird](https://www.thunderbird.net/ru/)
8. [Gophish](https://localhost:3333)
9. [Шаблон для сайта-подделки](https://ok.ru/)
















