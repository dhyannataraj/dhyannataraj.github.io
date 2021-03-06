---
layout: post
title:  "Настройка DICOM сервера Conquest / Configuring and Running Conquest DICOM server"
date:   2010-03-02 22:48:45
description: 
categories:
- blog
---

*This article is written in Russian. You might want to read a [Google Translation](https://translate.google.com/translate?sl=auto&tl=en&js=y&prev=_t&hl=en&ie=UTF-8&u=http%3A%2F%2Fdhyannataraj.github.io%2Fblog%2F2010%2F03%2F02%2FBuilding-and-configuring-Conquest-DICOM-for-linux%2F&edit-text=) in order to get the main idea.*

Тут речь пойдет об установке открытого [DICOM](http://ru.wikipedia.org/wiki/Dicom) сервера [Conquest](http://www.xs4all.nl/~ingenium/dicom.html) под ОС Linux (на примере Debian GNU/Linux)

Сервер умеет хранить индексную информацию в базах данных PostgreSQL, mysql и sqlite. Файлы исследований во всех случаях хранятся в файловой системе. 
Для случая PostgreSQL и mysql, сервер подключается к внешним базам данных, указанным в конфиге. Для случая sqlite эта мини-субд вкомпилируется напрямую в сервер. 

Справедливости ради, надо заметить, что мне не удалось собрать Conquest с sqlite, и я не пробовал устанавливать его с Postgresql, поэтому речь пойдет о сборе с использованием mysql.

1. **Скачиваем дистрибутив conquest'а для сборки под линукс** со страницы [http://www.xs4all.nl/~ingenium/dicom.html](http://www.xs4all.nl/~ingenium/dicom.html) (на момент написания страницы последняя версия -- [ftp://ftp-rt.nki.nl/outbox/MarcelVanHerk/dicomserver/conquestlinux1415.tar.gz](ftp://ftp-rt.nki.nl/outbox/MarcelVanHerk/dicomserver/conquestlinux1415.tar.gz) )

2. **Ставим пакет  libmysqlclient-dev** с заголовочными файлами библиотеки для соединения с mysql

В дистрибутиве Conquest не используется ни одна система сборки. Для сборки используется руками написанный .sh файл. Всего там четыре сборочных .sh файла, собирающие и устанавливающие Conqest с поддержкой заданной базы данных. Нас интересует файл maklinux_mysql

3. **Создаем директории /usr/lib/cgi-bin и /var/www**, поскольку maklinux_mysql сам их создавать не умеет. Если есть желание использовать другие пути, следует отредактировать maklinux_mysql. (Так же отмечу, что все помещаемое в  /var/www для нас бессмысленно, так как туда кладется какой-то ocx для MS IE который нам бесполезен, так что  строчки относящиеся /var/www можно просто закомментировать)

4. **Запускаем maklinux_mysql с правами root'а**

5. **Создаем в mysql базу данных с именем conquest и пользотвателя для работы с ней**

6. **Редактируем файл конфигурации /usr/lib/cgi-bin/dicom.ini** Добавляем туда имя имя пользователя базы данных (поле Username) пароль этого пользователя (поле Password), и имя самой базы данных (поле SQLServer)

7. **Создаем папку для хранения данных сервера**, по умолчанию предполагается что это /.data

8.  **Прописываем файл конфигурации /usr/lib/cgi-bin/dicom.ini** путь к созданной директории в поле MAGDevice0 (в самом низу). ВНИМАНИЕ: следите за тем чтобы в конце строк в файле конфигурации не стояло пробелов, Conquest к этому чувствителен, и сочтет что пробел -- часть пути

9. **Скопировать директорию samples** из которая живет в директории data дистрибутива, в созданную директорию для хранения данных

10. **Создать структуру базы данных** запустив программу dgate с ключом -r (запускать следует находясь строго в /usr/lib/cgi-bin/ иначе оно не найдет файлы конфигурации прописанные в конфиге без полного пути)

11. **Проверить что в базе данных появились таблицы**

12. **Редактировать /usr/lib/cgi-bin/acrnema.map** добавть туда IP адреса, DICOM имена и порты клиентов которые будут подсоединяться к серверу.

Следует помнить, что клиент -- понятие условное, оба и клиент и сервер являются по сути серверами и должны видеть дург-друга и уметь открвывать tcp соединение со своей стороны. Поэтому ни один из узлов не должен быть спрятан за NAT'ом...

13. **Запустить dgate с ключем -b** -  запуск сервера  в рабочее состояние в режиме отладки. (После того как работоспособность сервера будет подтверждена, -b можно будет убрать.

### Чем тестировать:

#### K-PACS

 K- PACS: [http://www.k-pacs.net/10555.html](http://www.k-pacs.net/10555.html) -  бесплатная windows-only программа с закрытым кодом. Под wine'ом не работает. 

Настройка подключения к серверу происходит по нажатию на кнопку с тремя компьютерами. Если конквест был поставлен с настройками по умолчанию, должны быть примерно такими:

![Экран настройки серверного подключения K-PACS ]({{ site.url }}/assets/images/2010-03-02-k-paks_config.png)

Если же нажать на кнопку с папкой сервером и еще какой-то хренью, то попадаешь в окно настроек локального сервера:

![Экран настройки локальных свойств K-PACS ]({{ site.url }}/assets/images/2010-03-02-k-paks_config2.png)

Данные из полей K-PACS Server Application Entry Title и Server Port следует добавить в /usr/lib/cgi-bin/acrnema.map вместе с IPшником той машины на которой этот K-PAСS запускается. Примерно вот так:


<pre>KPServer                10.0.0.2        104             un</pre>

Далее, при выборе вкладки database показывают список локальных картинок. При выборе вкладки network - сетевых. Но чтобы хоть что-то показали надо нажать кнопку Search. И вспомнить поставить галочку рядом с одним из серверов в списке вкладки.

Левые кнопки работают только если рядом с какой-то записью поставить галочку. От кнопки transfer выпадает список внешних серверов на которые можно изображение передать.

За счет кнопки import можно создавать новые картинки из обычных jpeg'ов.

#### MITO

MITO - Medical Imaging TOolkit - [http://amico.icar.cnr.it/mito.php](http://amico.icar.cnr.it/mito.php) опенсорсный просмоторщих DICOM, написанный на C++/wxWidgets. Бинарники дают только под винду, под линукс его судя по всему никогда не собирали, но теоретически -- это возможно. Ну или если откуда брать код для дальнейшей работы, так это отсюда.

Настроить MITO следует через меню Pacs/Preferences 

Во вкладе Network:

![Скриншот вкладки Network ]({{ site.url }}/assets/images/2010-03-02-mito_config.png)

Server Node: IP адрес Conquesta 
Server Port: Порт к которому прибинден Conquest (по умолчанию 5678)
Called AP Title: Имя сервера (в нашем случае, если эта настройка конквеста не менялись, CONQUEST1)
Calling AP Title: Имя клиента. Например MITO_CLIENT
Storage AP Title: Не совсем понял, но работает если совпадает с именем клиента, то есть MITO_CLIENT

Во второй вкладке, Transfer/Reseive:

![Скриншот вкладки Transfer/Reseive ]({{ site.url }}/assets/images/2010-03-02-mito_config2.png)

Storage Server Port: порт по которому локальный сервер MITO будет принимать информацию, например 1234
Receive/Local Files Directory: Папка в которой MITO будет хранить локальную информацию...

На сервере, в файле  /usr/lib/cgi-bin/acrnema.map следует правильно прописать данные MITO, его IP, имя, и порт. Что-то вроде этого:

<pre>MITO_CLIENT                10.0.0.2        1234             un</pre>

Для проверки работоспособности жмем Query в панеле инструментов, В появившемся окне снова кнопку Query, выбераем HEAD EXP2 и жмем кнопку Download. снимок головы неизвестного героя должен загрузиться в локальный сервер MITO.

## Update:

При переносе статьи из одного движка на другой на github был обнаружен проект занимающийся пакитированием Conquest'а под дебиан: [https://github.com/spectra/conquest-dicom-server](https://github.com/spectra/conquest-dicom-server) Работоспособность проекта я не проверял, но может быть кому-то эта ссылка облегчит жизнь.
