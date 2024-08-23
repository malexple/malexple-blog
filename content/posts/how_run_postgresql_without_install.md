+++
title = "Как запустить PostgreSQL на Windows без установки"
draft = false
date = 2024-08-15
[taxonomies]
categories = ["database"]
tags = ["postgres", "automation"]
+++

Для начала необходимо скачать zip архив с сайта https://www.enterprisedb.com/download-postgresql-binaries
Данный способ будет работать для 16, 15 и 14 версии PostgreSQL. Может будет работать для других версий, но это я уже не проверял. Скачанный zip нужно распаковать, и в корне новой папки PostgreSQL, создать файл с расширением *.bat. 
В данный файл необходимо поместить ниже следующий скрипт: 

```bat
@ECHO ON
REM Setting environment variables to run PostgreSQL
@SET PATH="%CD%\bin";%PATH%
@SET PGDATA=%CD%\data
@SET PGDATABASE=postgres
@SET PGUSER=postgres
@SET PGPORT=5432
@SET PGLOCALEDIR=%CD%\share\locale
REM %CD%\bin\initdb -U postgres -A trust
%CD%\bin\pg_ctl -D %CD%/data -l logfile start
ECHO "Press Enter to stop the server"
pause
%CD%\bin\pg_ctl -D %CD%/data stop
Information was taken from here
```
Что происходит в скрипте? C 3 строчки по 8 выставляются параметры для postgresql путь к папке, название бд, пользователь, порт для подключения. Строчки помеченные как REM это коментарии. Десятая строчка запускает postgresql. Пока запущена консоль postgresql будет запущен и на консоли будет выводиться текст "Press Enter to stop the server". Нажмите "Enter" если хотите завершить работу с postgresql, 13 строчка останавливает сервер postgresql.

При первом запуске раскомментируйте строку с вызовом initdb. Будет проинициализировано создание БД. При следующих запусках данную строчку нужно закомментировать.  При необходимости можете перенести папку с PostgreSQL  на USB-устройство.  Переменная %CD% возвращает путь к папке, в которой находится командный файл.

Данный способ является лишь одним из вариантов запуска PostgreSQL для экспериментов и тестирования. Надеюсь данная статья будет полезна.

Ну и напоследок. Если вам надо сконфигурировать PostgreSQL под определенные возможности железки есть сайт для генерации настроек. Там вы указывает операционную систему. Количество ядер, памяти и другое.

[www.pgconfig.org](https://www.pgconfig.org/)
[habr.com](https://habr.com/ru/sandbox/201774/)

