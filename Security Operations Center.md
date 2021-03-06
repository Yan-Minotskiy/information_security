# Практическая работа №2 - Security Operations Center
###### tags: `Методы и средства защиты информации`
**Авторы: Миноцкий Ян, Носков Сергей**
**Задача: При помощи  Security Operations Center провести анализ работы `usersSimulation`, определить IP адрес злоумышленника, количество запросов злоумышленника и количество легитимных пользователей системы.**
## Теория
Сегодня вопросами построения Security Operation Center (SOC) интересуются практически все представители отечественной экономики: от страховых компаний и банков до крупных промышленных предприятий. Такой интерес вызван прежде всего постоянно совершенствующимися атаками и потребностью в современном инструменте противодействия им. Действительно, SOC является одним из ключевых компонентов подразделения информационной безопасности любой организации. В первую очередь он нацелен на мониторинг, детектирование и оперативную реакцию на инциденты, и, как следствие, на сокращение ущерба и финансовых потерь, к которым тот или иной инцидент может привести.

При построении SOC важно помнить несколько моментов. [Security Operation Center](https://www.anti-malware.ru/analytics/Technology_Analysis/How_fast_run_SOC_Security_Operation_Center) — это не только технические средства. Это команда, задача которой обнаруживать, анализировать, реагировать, уведомлять о возникновении и предотвращать инциденты информационной безопасности. Еще один немаловажный компонент SOC — это процессы, поскольку подразумевается взаимодействие между сотрудниками подразделения, отвечающего за мониторинг и реагирование на инциденты, а также между различными подразделениями (например, ИБ и ИТ). От того, насколько качественно выстроены эти процессы, будет зависеть эффективность работы SOC. Технические средства являются лишь инструментами, позволяющими автоматизировать часть процессов, которые функционируют в SOC. Ярким доказательством этому может послужить SOC одной из региональных энергетических компаний, в котором разбор инцидентов осуществляется сотрудниками подразделения без использования средств автоматизации. Но здесь нельзя забывать о том, что конкретно в этом примере поток событий от компонентов инфраструктуры невелик. Если же поток событий довольно большой, без SIEM-системы не обойтись, поскольку скорость реакции на инциденты будет низкой. Таким образом, формула идеального SOC выглядит так: **SOC = Персонал + Процессы + Технические инструменты.**

Функции SOC на основе классификации [MITRE](https://www.mitre.org/sites/default/files/publications/pr-13-1028-mitre-10-strategies-cyber-ops-center.pdf):
![](https://i.imgur.com/k7oONSW.jpg)

## Используемые инструменты

1. [Elasticsearch](https://ru.wikipedia.org/wiki/Elasticsearch)
2. [Logstash](https://linux-notes.org/ustanovka-logstash-v-unix-linux/)
3. [Kibana](https://www.quora.com/What-is-Kibana-How-does-it-work#:~:text=Kibana%20-%20это%20плагин%20визуализации,карты%20поверх%20больших%20объемов%20данных)

**Elasticsearch** — это механизм индексирования и хранения полученной информации, а также полнотекстового поиска по ней. Он основан на библиотеке Apache Lucene и, по сути, является NoSQL database решением. Главная задача этого инструмента — организация быстрого и гибкого поиска по полученным данным. Для ее решения имеется возможность выбора анализаторов текста, функционал «нечеткого поиска», поддерживается поиск по информации на восточных языках (корейский, китайский, японский).

**Logstash** — это инструмент получения, преобразования и сохранения данных в общем хранилище. Его первой задачей является прием данных в каком-либо виде: из файла, базы данных, логов или информационных каналов. Далее полученная информация может модифицироваться с помощью фильтров, например, единая строка может быть разбита на поля, могут добавляться или изменяться данные, несколько строк могут агрегироваться и т.п. Обработанная информация посылается в системы — потребители этой информации. Говоря о связке ELK, потребителем информации будет Elasticsearch, однако возможны другие варианты, например системы мониторинга и управления (Nagios, Jira и др.), системы хранения информации (Google Cloud Storage, syslog и др.), файлы на диске. Возможен даже запуск команды при получении особого набора данных.

**Kibana** — это user friendly интерфейс, для Elasticsearch, который имеет большое количество возможностей по поиску данных в дебрях индексов Elasticsearch и отображению этих данных в удобочитаемых видах таблиц, графиков и диаграмм.

Таким образом, связка **ELK** является достаточно мощным инструментом сбора и аналитики информации.

> А для занимательного погружения в изучение ELK рекомендую статью [Kibana-мать или Зачем вам вообще нужны логи?](https://habr.com/ru/company/uteam/blog/278729/)

## Инструкция по установке
### Загрузка

```
git clone https://github.com/Ivanhahanov/InformationSecurityMethodsAndTools.git
```

### Развертывание
Переходим в директорию SOC:
```
cd SOC
```
Запускаем ELK стек:
```
docker-compose up -d elasticsearch logstash kibana 
```
Ждём, пока всё запустится 
Проверка осуществляется двумя способами:
1. Посмотреть логи `docker-compose logs -f`
2. Посмотреть процессы `docker-compose ps`

После того как ELK-стек запустился, необходимо запустить сервис с формой ввода логина и пароля:
```
docker-compose up service
```
## Инструкция по выполнению

Теперь у нас развёрнута инфраструктура состоящая из:
* Базы данных (Elasticsearch) 
* Сервиса для сбора логов (Logstash) 
* Сервиса для визуализации данных (Kibana)
* Сайта c формой входа.

Сервис уже автоматически пишет все свои логи в Logstash, тот в свою очередь пишет Elasticsearch, а визуализацию логов можно посмотреть используя Kibana

Итак, зашли в Kibana по адресу http://localhost:5601
```
Логин: elastic
Пароль: changeme
```
После ввода логина и пароля видим такое окно:
![](https://i.imgur.com/TiCkMxJ.png)

Добро пожаловать в Kibana!

Теперь переходим по адресу http://localhost:8080
![](https://i.imgur.com/XMZIJw8.png)
И создаем активность путем ввода различных учётных данных
По умолчанию **admin:admin**

Теперь смотрим, а записались ли логи наши действия в базу данных
Открываем выпадающее меню и выбераем Stack Management, а затем Index patterns

![](https://i.imgur.com/rLonHr3.png)
![](https://i.imgur.com/iaeBnNy.png)

Создаем индекс с помощью маски logstash-*
![](https://i.imgur.com/GzL94va.png)

В поле “Time Filed” указываем `@timestamp`
![](https://i.imgur.com/vGzmwlQ.png)
![](https://i.imgur.com/xrM1hTS.png)

После того, как создался паттерн, взглянем на наши данные
Для этого нам нужен модуль Discover, который находится в главном меню

![](https://i.imgur.com/snAzcCt.png)
![](https://i.imgur.com/wkUovIK.png)
> На скриншоте показаны данные уже после работы программы `usersSimulation`

Доступные поля, на которые стоит обратить внимание:

`rAddr` - IP адрес с которого был отправлен запрос
`login` - Введённые логи
`hashPassword` - Хэшированный пароль
`route` - URL сервера к которому был отправлен запрос
`message` - Сообщение сервиса
`level` - тип сообщений (warning, info)

Для генерации данных запустили программу `./usersSimulation`, которая будет отправлять запросы от различных пользователей

Данная программа эмулирует активность различных пользователей и выполняет следующие функции:

* Отправляет запросы на регистрацию от различных пользователей
* Эмулирует действия злоумышленника, который пытается подобрать пароль

Программа будет работать до тех пор, пока её не остановить с помощью сочетания клавиш `Ctrl-C`

Чем больше программа будет работать тем больше будет различных запросов для анализа.

У нас она отработала приблизительно 10 минут, за это время было сгенерировано 4182 запросов, которые нам сейчас предстоит проанализировать

### Визуализация
С помощью средств визуализации, создали среду для анализа трафика

На данном скриншоте мы видим IP с которого произведено больше всего запросов, следовательно это потенциальный злоумышленник, что мы и проверим путем более фундаментального анализа
![](https://i.imgur.com/fONa37x.png)

#### Пошаговое руководство по созданию среды для анализа трафика:

![](https://i.imgur.com/UgOwoKv.png)
![](https://i.imgur.com/yM6Tmz0.png)
![](https://i.imgur.com/QtzfS1N.png)
![](https://i.imgur.com/6wMhznj.png)
![](https://i.imgur.com/yg0Z97V.png)
![](https://i.imgur.com/eqkNJRS.png)
![](https://i.imgur.com/iBaFYEW.png)

По аналогии продолжаем добавлять фильтры, в итоге должно получиться следующее:

![](https://i.imgur.com/KL0UVtQ.png)

На этом скриншоте предоставлены все необходимые данные, чтобы определить необходимые параметры:

1. IP адрес злоумышленника - **123.248.123.172**
*Так как с этого IP было произведено максимальное количество нелигитимных запросов*
2. Количество запросов злоумышленника - **750**
3. Количество легитимных пользователей системы - **10**
*Все оставшиеся IP считаем легитимными, так как они успешно авторизировались, произведя при этом минимальное количество запросов*

## Источники и полезные ссылки

1. [Security Operation Center](https://www.anti-malware.ru/analytics/Technology_Analysis/How_fast_run_SOC_Security_Operation_Center)
2. [MITRE](https://www.mitre.org/sites/default/files/publications/pr-13-1028-mitre-10-strategies-cyber-ops-center.pdf)
3. [MITRE перевод на русский](https://rvision.pro/10-strategij-pervoklassnogo-soc-perevod-gajda-mitre/)
4. [Elasticsearch](https://ru.wikipedia.org/wiki/Elasticsearch)
5. [Logstash](https://linux-notes.org/ustanovka-logstash-v-unix-linux/)
6. [Kibana](https://www.quora.com/What-is-Kibana-How-does-it-work#:~:text=Kibana%20-%20это%20плагин%20визуализации,карты%20поверх%20больших%20объемов%20данных)
7. [Kibana-мать или Зачем вам вообще нужны логи?](https://habr.com/ru/company/uteam/blog/278729/)
8. [Kibana *local*](http://localhost:5601)
9. [Форма авторизации *local*](http://localhost:8080)






