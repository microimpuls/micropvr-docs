.. _install-and-using:

************************
2. Установка и настройка
************************

Для работы micropvr необходимо наличие установленных библиотек *libjsonrpc*, *libjson*.

MicroPVR, а также все сопутствующие пакеты и необходимые библиотеки совместимых версий, поставляются в виде
установочных deb-пакетов и устанавливаются утилитой dpkg.

.. _download-software:

2.1. Где скачать установочные пакеты?
=====================================

Для всех
  Инсталляционные пакеты ПО распространяются через FTP, доступ к которому предоставляется на время действия
  договора. Более подробная инфорамция по ссылке http://forum.micro.im/ в разделе "Дистрибутивы и обновления ПО".

Для инженеров Microimpuls
  При установке ПО на сервер через систему оркестровки все необходимые установочные пакеты
  актуальных версий скачиваются из репозитория автоматически.

.. _configuration:

2.2. Конфигурационные файлы и управление процессами
===================================================

.. _micropvr_configuration:

2.2.1. Менеджер записей - процесс micropvr
------------------------------------------

Для запуска micropvr используется init.d скрипт: ``/etc/init.d/micropvr``: ::

    $ /etc/init.d/micropvr
    Usage: /etc/init.d/micropvr {start|stop|restart|force-reload|reload}

Файлы логов по-умолчанию сохраняются в ``/var/log/micropvr/micropvr.log``.

.. _micropvr-options-description:

2.2.1.1. Описание параметров файла конфигурации micropvr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Файл конфигурации находится в ``/etc/micropvr/micropvr.conf``,
задаётся в формате JSON. Пример: ::

    {
        "log-foreground": false,
        "log-syslog": false,
        "log-verbose-level": 3,
        "log-path": "/var/log/micropvr/micropvr.log",
        "log-state-period": 0, // in minutes
        "log-state-path": "/var/log/micropvr/micropvr_state.log",
        "json-rpc-listen-host": "0.0.0.0",
        "json-rpc-listen-port": 4089,
        "task-postpone-time": 60, // in seconds
        "records-checking-period": 60, // in seconds
        "records-removing-period": 5, // in seconds
        "records-min-free-space": 10000, // space in MiB
        "records-default-reserve-size": 20480, //space in MiB
        "records-block-active-time": 2 // in minutes
        "recorder-check-free-space": true,
        "recorder-pid-path": "/var/run/micropvr/",
        "recorder-cmd": "recorder",
        "recorder-log-enabled": true,
        "recorder-log-path": "/var/log/micropvr/recorder.log",
        "recorder-init-timeout": 5, // in seconds
        "recorder-checking-period": 1 // in seconds
    }

log-syslog ``bool``
  Использовать ли службу syslogd для записи логов в /var/log/syslog.
  Не рекомендуется включать при интенсивном логировании.

log-facility ``int``
  Тег в syslog.

log-path ``str``
  Путь до лог-файла для логирования напрямую без syslogd.
  
log-verbose-level ``int``
  Уровень логирования от 0 до 6, 6 - максимальный EXTENDED уровень.

log-state-period ``int``
  *С версии 1.5.0*
  
  Период записи лога состояния в минутах. При значении 0 запись отключается.
  
log-state-path ``str``
  *С версии 1.5.0*
  
  Путь до файла в который будет записываться лог состояния.
  
log-foreground ``bool``
  Вывод лога в stdout.

json-rpc-enabled ``bool``
  Включает интерфейс JSON RPC API. Через этот API без перезапуска micropvr
  отдельные потоки могут быть приостановлены или перезапущены.

json-rpc-listen-host ``str``
  Адрес интерфейса для ожидания входящих подключений к JSON RPC API.
  Значение "0.0.0.0" означает слушать на всех интерфейсах.

json-rpc-listen-port ``int``
  Номер порта TCP для JSON RPC API, по-умолчанию 4089.

task-postpone-time ``int``
  Время в секундах, на которое будет отложена задача при неудачной попытке ее выполнения.
  По-умолчанию 60.

task-activation-period ``int``
  *Убрано в версии 1.5.0*
  
  Время в секундах, задающее условие: в очередь запуска попадают задачи,
  выполнение которых должно наступить в течение ближайших N секунд.
  По-умолчанию 900.

task-caching-period ``int``
  *Убрано в версии 1.5.0*
  
  Период кеширования задач внутренним планировщиком, в секундах, по-умолчанию 450.

records-checking-period ``int``
  Время проверки размера записей в секундах, по умолчанию 60. Определяет точность позиционирования по архиву для функции перемотки.

records-removing-period ``int``
  Минимальный интервал удаления устаревших записей в секундах. По умолчанию 5.
  
records-outdated-checking-period  ``int``
  *Убрано в версии 1.5.0*
  
  Интервал проверки устаревших записей в секундах. По умолчанию 5.

records-min-free-space ``int``
  *С версии 1.2.1*
  
  Минимальный объем свободного места на диске в MiB, при котором разрешена запись.
  
records-default-reserve-size ``int``
  *С версии 1.4.0*
  
  Объём резервируемого на диске места для одной активной записи в MiB, по умолчанию 20480.
  Запись не будет производиться, если включен механизм проверки свободного места на диске и объём места после резервирования станет меньше минимально разрешённого.
  По умолчанию 20480.
  
records-block-active-time ``int``
  *С версии 1.7.0*
  
   Время блокировки удаления активных записей в минутах после истечения их срока жизни. Запись считается активной, если к ней было хотя бы одно обращение. По умолчанию 240.

recorder-check-free-space ``bool``
  *С версии 1.2.1*
  
  Определяет включение механизма проверки свободного места на диске. По умолчанию false.
  
recorder-cmd ``str``
  *С версии 1.5.0*
  
  Команда запуска модуля MicroPVR recorder, который осуществляет запись
  потока в файл (для запуска recorder и совместимых по CLI-интерфейсу программ).
  По умолчнию "recorder".

recorder-pid-path ``str``
  Путь для записи pid-файлов recorder'ов, по-умолчанию "/var/run/micropvr".

recorder-log-enabled ``bool``
  Разрешить писать recorder'у в лог, по-умолчанию false.

recorder-log-path ``str``
  Путь до лог-файла recorder'а, по-умолчанию "/var/log/micropvr/recorder.log".

recorder-init-timeout ``int``
  Время в секундах на перезапуск recorder'a в случае неудачного старта,
  по-умолчанию 5. Если recorder не удалось запустить за это время, выполнение
  задачи будет отложено.

recorder-cheking-period ``int``
  Период проверки состояния recorder'ов, в секундах, по-умолчанию 1.  

score-max-score ``float``
  *С версии 1.5.1*
  
  Максимальное значение **score**, при котором метод **is_alive** возвращает **true**. По умолчанию 20.0.

score-max-net-load ``integer``
  *С версии 1.5.1*
  
  Максимальная загрузка исходящего сетевого потока в Mbit/sec. По умолчанию 700.

score-max-sessions ``integer``
  *С версии 1.5.1*
  
  Максимальное количество сессий. По умолчанию 10000.

score-max-cpu-la1 ``float``
  *С версии 1.5.1*
  
  Максимальное значение средней загрузки вычислительных ресурсов за 1 минуту. По умолчанию 1.0.

.. _micropvs_configuration:

2.2.2. Стриминг записей в формате HTTP - процесс nginx с модулем micropvs
-------------------------------------------------------------------------

Для запуска nginx с модулем micropvs используется init.d скрипт: ``/etc/init.d/micropvs``: ::

    $ /etc/init.d/micropvs
    Usage: /etc/init.d/micropvs {start|stop|restart|force-reload|reload}

Файлы логов по-умолчанию сохраняются в ``/usr/local/nginx-micropvr/logs/``.

.. _micropvs-options-description:

2.2.2.1. Описание параметров micropvs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Файл конфигурации находится в ``/usr/local/nginx-micropvr/conf/nginx.conf``,
пример: ::

    worker_processes 16;
    events {
        worker_connections 4096;
        use epoll;
        multi_accept on;
    }
    http {
        access_log logs/access.log;
        error_log logs/error.log;
        include mime.types;
        default_type application/octet-stream;
        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 5;
        send_timeout 36000;
        server {
            listen 8080;
            location / {
                pvr_api_host "127.0.0.1";
                pvr_api_port 4089;
                ts;
            }
            location = /nginx-stats {
                stub_status on;
                access_log off;
                allow 127.0.0.1;
                deny all;
            }
        }
    }

pvr_api_host ``str``
  IP-адрес JSON-RPC API процесса micropvr.

pvr_api_port ``int``
  Порт JSON-RPC API процесса micropvr.

ts
  Подключение модуля micropvs.

Остальные параметры стандартные для сервера `nginx <http://nginx.org/en/docs/>`_.

.. _live555_configuration:

2.2.3. Стриминг записей в формате RTSP - live555_mi
---------------------------------------------------

Для запуска live555_mi используется init.d скрипт: ``/etc/init.d/live555_mi``: ::

    $ /etc/init.d/live555_mi
    Usage: /etc/init.d/live555_mi {start|stop|restart|force-reload|reload}

Файлы логов по-умолчанию сохраняются в ``/var/log/live555_mi/``.


.. _monit-script:

2.3. Скрипт для monit
=====================

Для слежения за процессом micropvr удобно использовать monit, пример скрипта: ::

    check process micropvr with pidfile /var/run/micropvr.pid
        start program = "/etc/init.d/micropvr start" with timeout 60 seconds
        stop program  = "/etc/init.d/micropvr stop"
        if cpu > 60% for 2 cycles then alert
        if cpu > 90% for 5 cycles then restart
        if totalmem > 6000.0 MB for 5 cycles then restart
        if 3 restarts within 5 cycles then timeout
        group micropvr

