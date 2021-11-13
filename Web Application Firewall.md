# Практическая работа №4 - Web Application Firewall

**Авторы: Миноцкий Ян, Носков Сергей.**

**Задача: обнаружить уязвимости веб-приложения и устранить их при помощи WAF.**

## Часть 1. Обнаружение уязвимостей Web-приложения

Заходим по адресу http://localhost/setup.php 

Нажимаем `Create/Reset Database` и затем проходим авторизацию.
Логин: `admin`
Пароль `password`

### Command Injection

**Command Injection** - это атака, целью которой является выполнение произвольных команд на ОС с помощью уязвимого веб-приложения.

[Хорошая статья на Хабре про данную уязвимость](https://habr.com/ru/company/pentestit/blog/550252/)

Нам необходимо каким-то образом написать команду `ls` в поле для ввода ip-адреса, чтобы он обрабатывался отдельно от команды ping.

Мы можем изучать сфайловую структуру...

![](https://i.imgur.com/jxlLadt.png)

```
yandex.ru > /dev/null; ls source
```

![](https://i.imgur.com/dqf91He.png)

```
yandex.ru > /dev/null; cat source/high.php
```

Смотреть содержимое этих файлов...

![](https://i.imgur.com/Hn1fHva.png)

```
8.8.8.8 > /dev/null; cat /etc/passwd
```

![](https://i.imgur.com/eVgYw66.png)

**Рекомендации**

1. Отказ от использования функции `exec()`
2. Фильтровать входные значения
3. Использовать "белые" списки
4. [Использовать WAF](#Часть-2-Устранение-уязвимостей-при-помощи-Web-Application-Firewall)

### File Inclusion

**File Inclusion** - это возможность использования и выполнения файлов на серверной стороне. Уязвимость позволяет удаленному пользователю получить доступ с помощью специально сформированного запроса к произвольным файлам на сервере, в том числе содержащую конфиденциальную информацию.

После обращения с запросом к следующей странице мы можем наблюдать содержимое файла `/etc/passwd`.

`http://localhost/vulnerabilities/fi/?page=/etc/passwd`

![](https://i.imgur.com/azXoek6.png)

**Рекомендации:**
1. Регулярно проверять безопасность веб-приложения с целью предотвращения возникновения уязвимостей
2. Добавить в журнал веб-сервера строчку `<?php exit(1); ?>`. Данный скрипт будет предотвращать любые попытки выполнения php-кода в этом файле.
3. [Использовать WAF](#Часть-2-Устранение-уязвимостей-при-помощи-Web-Application-Firewall)



### SQL Injection 

**SQL инъекция** - это размещение вредоносного кода в SQL инструкции с помощью ввода веб страницы.

![](https://i.imgur.com/akGD3Lt.png)

[Лабораторная работа по SQL-инъекциям по дисциплине клиент-серверные системы управления базами данных.](https://github.com/Yan-Minotskiy/postgreslab/blob/main/%D0%9B%D0%B0%D0%B1%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80%D0%BD%D1%8B%D0%B5%20%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D1%8B/%D0%9B%D0%B0%D0%B1%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80%D0%BD%D0%B0%D1%8F%20%E2%84%968%20-%20SQL-%D0%B8%D0%BD%D1%8A%D0%B5%D0%BA%D1%86%D0%B8%D0%B8%20%D0%B2%20%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%BD%D1%83%D1%8E%20%D0%B1%D0%B0%D0%B7%D1%83%20%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85.md)

Применим Union Based SQL-injection

```
1' union select user,password from users -- -
```

Получим следующее:

![](https://i.imgur.com/oy6DVWA.png)


**Рекомендации:**
1. Реализововать на уровне API, чтобы в GET и POST запрос, нельзя было вставлять ключевые слова для SQL запросов.
2. Исполльзовать экранирование спец. символов.
3. [Использовать WAF](#Часть-2-Устранение-уязвимостей-при-помощи-Web-Application-Firewall)

### XSS reflected

**XSS (англ. Cross-Site Scripting — «межсайтовый скриптинг»)** — тип атаки на веб-системы, заключающийся во внедрении в выдаваемую веб-системой страницу вредоносного кода (который будет выполнен на компьютере пользователя при открытии им этой страницы) и взаимодействии этого кода с веб-сервером злоумышленника.

[полезная статья про уязвимости XSS](https://course.secsem.ru/wiki/%D0%92%D0%B5%D0%B1-%D0%B1%D0%B5%D0%B7%D0%BE%D0%BF%D0%B0%D1%81%D0%BD%D0%BE%D1%81%D1%82%D1%8C/%D0%A3%D1%8F%D0%B7%D0%B2%D0%B8%D0%BC%D0%BE%D1%81%D1%82%D0%B8_XSS)

Введём в строку и отправим следующий запрос:

```
<img src='x' onerror='alert(document.cookie)'>
```

![](https://i.imgur.com/GcfjnL7.png)

**Рекомендации:**

1. Запретить чтение куки из JS-кода, выставив атрибут куки `HttpOnly`, он выставляется вместе с кукой в заголовке ответа `Set-Cookie: session=eyJ1c2VyIjoibGlsIiwidXNlcl9pZCI6M30.XI9rjw.-tX3LIQ-cwyUnNlOuQqyVXBu99A; HttpOnly; Path=/`.
2. [Использовать WAF](#Часть-2-Устранение-уязвимостей-при-помощи-Web-Application-Firewall)

### Прочие уязвимости

**Брутфорс (от англ. brute force — грубая сила)** — метод угадывания пароля (или ключа, используемого для шифрования), предполагающий систематический перебор всех возможных комбинаций символов до тех пор, пока не будет найдена правильная комбинация.

**CSRF (англ. cross-site request forgery — «межсайтовая подделка запроса», также известна как XSRF)** — вид атак на посетителей веб-сайтов, использующий недостатки протокола HTTP. Если жертва заходит на сайт, созданный злоумышленником, от её лица тайно отправляется запрос на другой сервер (например, на сервер платёжной системы), осуществляющий некую вредоносную операцию (например, перевод денег на счёт злоумышленника). Для осуществления данной атаки жертва должна быть аутентифицирована на том сервере, на который отправляется запрос, и этот запрос не должен требовать какого-либо подтверждения со стороны пользователя, которое не может быть проигнорировано или подделано атакующим скриптом.




## Часть 2. Устранение уязвимостей при помощи Web Application Firewall

**Файрвол веб-приложений (англ. Web application firewall, WAF)** — совокупность мониторов и фильтров, предназначенных для обнаружения и блокирования сетевых атак на веб-приложение. WAF относятся к [прикладному уровню модели OSI](https://github.com/Yan-Minotskiy/network_config/blob/main/iptables.md#%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D1%8C-osi).


Для защиты от основных уязвимостей с помощью WAF я прописал в файл `rules.conf`, который находится в директории `custom`, следующие правила:

```
SecRule ARGS "@contains &&" "phase:2,log,deny,msg:'Command injection'id:700009"
SecRule ARGS "@contains pwd" "phase:2,log,deny,msg:'Command injection'id:700010"
SecRule ARGS "@contains cat" "phase:2,log,deny,msg:'Command injection'id:700011"
SecRule ARGS "@contains etc" "phase:2,log,deny,msg:'File Inclusion'id:700012"
SecRule ARGS "@contains union" "phase:2,log,deny,msg:'SQL injection'id:700013"
SecRule ARGS "@contains alert('xss')" "phase:2,log,deny,msg:'XSS_Reflected'id:700014"
SecRule ARGS "@contains alert('document.cookie')" "phase:2,log,deny,msg:'XSS_Reflected'id:700015"
SecRule ARGS "@contains alert('bingo')" "phase:2,log,deny,msg:'XSS_Reflected'id:700016"
```

Проверочный скрипт `exploration.py` показал следующий результат.

![](https://i.imgur.com/ML8yCSd.png)

И теперь, когда мы пытаемся эксплуатировать уязвимость, пояляется следующее сообщение.

![](https://i.imgur.com/yd1WHNo.png)


Стоит отметить, что прописанные выше правила могут являтся лишь временной мерой закрытия уязвимости сайта. Правила WAF можно улучшить, например, прописать регулярное выражение для ip-адреса, но в таком случае урежится функцианал (ме не сможем обращаться к сервисам по доменному имени).



## Источники

1. [Инструкция по выполнению лабораторной работы](https://hackmd.io/@ivanh/HyrJ2Ldl_)
2. [Документация по написанию правил на WAF](https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual-(v2.x)#SecRule)
3. [OWASP TOP 10 уязвимостей в 2021 году](https://owasp.org/Top10/)
4. [OS Command Injection: ось под контролем | Habr](https://habr.com/ru/company/pentestit/blog/550252/)
5. [Решение лабораторной работы по SQL-инъекциям по дисциплине клиент-серверные системы управления базами данных.](https://github.com/Yan-Minotskiy/postgreslab/blob/main/%D0%9B%D0%B0%D0%B1%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80%D0%BD%D1%8B%D0%B5%20%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D1%8B/%D0%9B%D0%B0%D0%B1%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80%D0%BD%D0%B0%D1%8F%20%E2%84%968%20-%20SQL-%D0%B8%D0%BD%D1%8A%D0%B5%D0%BA%D1%86%D0%B8%D0%B8%20%D0%B2%20%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%BD%D1%83%D1%8E%20%D0%B1%D0%B0%D0%B7%D1%83%20%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85.md)
6. [Методы борьбы с SQL инъекциями](https://xdan.ru/kak-opredelit-chto-sajt-podvergsja-sql-injectiont.html)
7. [Чек-лист устранения SQL-инъекций | Habr](https://habr.com/ru/company/pentestit/blog/546232/)
8. [Веб-безопасность/Уязвимости XSS](https://course.secsem.ru/wiki/%D0%92%D0%B5%D0%B1-%D0%B1%D0%B5%D0%B7%D0%BE%D0%BF%D0%B0%D1%81%D0%BD%D0%BE%D1%81%D1%82%D1%8C/%D0%A3%D1%8F%D0%B7%D0%B2%D0%B8%D0%BC%D0%BE%D1%81%D1%82%D0%B8_XSS)



