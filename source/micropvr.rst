.. _micropvr:

********
Описание
********

**MicroPVR** - комплекс программного обеспечения, предназначенный для создания видео-сервера записи Live-контента и
предоставления абонентам интерактивных видео-сервисов, таких как отложенный просмотр (Catch Up), видеомагнитофон (NPVR),
пауза (Pause TV), просмотр в сдвиге (Timeshift) и других.

Благодаря наличию API возможно создание собственных интерактивных сервисов.

Позволяет задействовать различные виды памяти СХД, например HDD, SSD, RAM, что позволяет создавать конфигурации для
обслуживания до 10Gbps абонентского трафика на один сервер.

.. _architecture:

Описание архитектуры
====================

**micropvr** - менеджер видео-записей. Управляющий процесс, реализующий API-интерфейс в формате JSON и взаимодействующий с
процессами записи контента.

**recorder** - модуль записи контента в формате MPEG-TS.

**micropvs** - модуль для nginx, взаимодействует с micropvr и предназначен для вещания записанных передач в формате HTTP
MPEG-TS.

**live555_mi** - модифицированная версия видео-сервера `Live555MediaServer <http://www.live555.com/mediaServer/>`_, взаимодействует с micropvr и предназначен для
вещания записанных передач в формате RTSP/UDP/RTP.

.. image:: img/micropvr-architecture.png

.. _system-requirements:

********************
Системные требования
********************

ПО MicroPVR предназначено для работы в ОС Linux Debian 64-битной версии и подобных
дистрибутивах, по-умолчанию поставляется версия для архитектуры amd64.

*Внимание! ПО MicroPVR не предназначено для работы на виртуальной машине по причине негарантированной стабильности
работы виртуального сетевого адаптера.*

Требования к аппаратному обеспечению, исходя из принимаемого/передающегося объема трафика:

+--------------------+------------------------------------------------------------------------------------------------------+
| Процессор          | 1 ядро 2.4Ghz+ на каждые 400Mbps трафика                                                             |
+--------------------+------------------------------------------------------------------------------------------------------+
| Оперативная память | 1GB на каждые 400Mbps трафика                                                                        |
+--------------------+------------------------------------------------------------------------------------------------------+
| Сетевая карта      | карта с Enterprise чипом Intel, имеющая несколько очередей для входящих пакетов (multiple Rx queues) |
+--------------------+------------------------------------------------------------------------------------------------------+

Расчет суммарного объема хранилища
==================================

Если:
::
    M - количество каналов определенного битрейта/качества
    N - средний битрейт одного канала в мегабитах в секунду
    K - глубина записи архива в днях
    X - результат расчета в терабайтах

Тогда формула расчета:
::
    X = ((((M * N) / 8) * 3600 * 24 * K) / 1024) / 1024

Например, 100 каналов SD в битрейте 3.5Mbps необходимо записывать на 7 дней:
::
    ((((100 * 3.5) / 8) * 3600 * 24 * 7) / 1024) / 1024 = 25,23TB

Расчет конфигурации хранилища
=============================

Конфигурация серверов может быть подобрана индивидуально исходя из бюджета и технических требований.
Рекомендованные значения - записывать не более 50 каналов на один сервер, исходя из этого, если:
::
    M - количество каналов определенного битрейта/качества
    N - средний битрейт одного канала в мегабитах в секунду
    K - глубина записи архива в днях
    X - общий объем хранилища
    Y - количество серверов

Тогда формула расчета:
::
    M = 50
    Y = X / (((((M * N) / 8) * 3600 * 24 * K) / 1024) / 1024)

При количестве одновременных запросов на сервис PVR, равном 1000 на каждый сервер
(т.е. суммарно 1000*Y одновременных пользователей), количество дисков хранилища на одном сервере должно
быть не менее 6, количество SSD в размере 500GB не менее 2.

При необходимости резервирования СХД количество дисков следует увеличить исходя из выбранной модели резервирования (RAID Level).

.. _install-and-using:

*************************
Установка и использование
*************************

Для работы micropvr необходимо наличие установленных библиотек *libjsonrpc*, *libjson*.

MicroPVR, а также все сопутствующие пакеты и необходимые библиотеки совместимых версий, поставляются в виде
установочных deb-пакетов и устанавливаются утилитой dpkg. См. :ref:`Где взять <download-software>`.

micropvr
========

Для запуска micropvr используется init.d скрипт: ``/etc/init.d/micropvr``:
::
    $ /etc/init.d/micropvr
    Usage: /etc/init.d/micropvr {start|stop|restart|force-reload|reload}

Файлы логов по-умолчанию сохраняются в ``/var/log/micropvr/micropvr.log``,
включена ротация логов через logrotate.d

nginx + micropvs
================

Для запуска nginx с модулем micropvs используется init.d скрипт: ``/etc/init.d/micropvs``:
::
    $ /etc/init.d/micropvs
    Usage: /etc/init.d/micropvs {start|stop|restart|force-reload|reload}

Файлы логов по-умолчанию сохраняются в ``/usr/local/nginx-micropvr/logs/``,
включена ротация логов через logrotate.d

live555_mi
==========

Для запуска live555_mi используется init.d скрипт: ``/etc/init.d/live555_mi``:
::
    $ /etc/init.d/live555_mi
    Usage: /etc/init.d/live555_mi {start|stop|restart|force-reload|reload}

Файлы логов по-умолчанию сохраняются в ``/var/log/live555_mi/``,
включена ротация логов через logrotate.d

.. _download-software:

Где взять
=========

Для всех
  Скачать необходимые инсталляционные пакеты можно в официальном техническом сообществе Microimpuls
  по ссылке http://forum.micro.im/ в разделе "Дистрибутивы и обновления ПО".

Для инженеров Microimpuls
  При установке ПО на сервер через систему оркестровки все необходимые установочные пакеты
  актуальных версий скачиваются из репозитория автоматически.

.. _configuration:

Конфигурация
============

.. _micropvr_configuration:

micropvr
--------

Файл конфигурации находится в ``/etc/micropvr/micropvr.conf``,
задаётся в формате JSON. Пример:
::
    {
        "log-foreground": false,
        "log-syslog": false,
        "log-verbose-level": 3,
        "log-path": "/var/log/micropvr/micropvr.log",
        "json-rpc-listen-host": "0.0.0.0",
        "json-rpc-listen-port": 4089,
        "task-postpone-time": 60,
        "task-activation-period": 900,
        "task-caching-period": 450,
        "records-checking-period": 60,
        "records-outdated-checking-period": 5,
        "records-removing-period": 5,
        "records-min-free-space": 100000, // space in MiB
        "recorder-check-free-space": true,
        "recorder-pid-path": "/var/run/",
        "recorder-cmd": "recorder",
        "recorder-log-enabled": true,
        "recorder-log-path": "/var/log/micropvr/recorder.log",
        "recorder-init-timeout": 5,
        "recorder-checking-period": 1,
        "channels" []
    }

.. _micropvr-options-description:

Описание параметров micropvr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

log-syslog ``bool``
  Использовать ли службу syslogd для записи логов в /var/log/syslog.
  Не рекомендуется включать при интенсивном логировании.

log-facility ``int``
  Тег в syslog.

log-path ``str``
  Путь до лог-файла для логирования напрямую без syslogd.

log-verbose-level ``int``
  Уровень логирования от 0 до 5, 5 - максимальный DEBUG уровень.

log-foreground ``bool``
  Вывод лога в stdout.

json-rpc-enabled ``bool``
  Включает интерфейс JSON RPC API. Через этот API без перезапуска micropvr
  отдельные потоки могут быть приостановлены или перезапущены.

json-rpc-listen-host ``str``
  Адрес интерфейса для ожидания входящих подключений к JSON RPC API.
  Значение "0.0.0.0" означает слушать на всех интерфейсах.

json-rpc-listen-port ``int``
  Номер порта TCP для JSON RPC API, по-умолчанию 9089.

task-activation-period ``int``
  Время в секундах, задающее условие: в очередь запуска попадают задачи,
  выполнение которых должно наступить в течение ближайших N секунд.
  По-умолчанию 900.

task-caching-period ``int``
  Период кеширования задач внутренним планировщиком, в секундах, по-умолчанию 450.

records-outdated-checking-period ``int``
  Период проверки и удаления устаревших записей на диске.

records-removing-period ``int``
  Минимальный интервал удаления устаревших записей.

records-min-free-space ``int``
  Критический объем свободного места на диске в MiB, при котором разрешена запись.

recorder-check-free-space ``bool``
  Определяет включение механизма проверки свободного места на диске.

recorder-cmd ``str``
  Команда запуска модуля MicroPVR recorder, который осуществляет запись
  потока в файл (для запуска recorder и совместимых по CLI-интерфейсу программ).

recorder-pid-path ``str``
  Путь для записи pid-файлов recorder'ов, по-умолчанию "/var/run/micropvr".

recorder-log-enabled ``bool``
  Разрешить писать recorder'у в лог, по-умолчанию false.

recorder-log-path ``str``
  Путь до лог-файла recorder'а, по-умолчанию "/var/log/micropvr/recorder.log"

recorder-init-timeout ``int``
  Время в секундах на перезапуск recorder'a в случае неудачного старта,
  по-умолчанию 5. Если recorder не удалось запустить за это время, выполнение
  задачи будет отложено.

recorder-cheking-period ``int``
  Период проверки состояния recorder'ов, в секундах, по-умолчанию 1.

channels ``list``
  Список потоков для записи. Позволяет задать задачи на запись статисчески, без
  использования внешнего управляющего сервера. Опции каналов соответствуют
  параметрам метод **Add task** :ref:`JSON-RPC API <jsonrpc-api>`.

.. _micropvs_configuration:

micropvs
--------

Файл конфигурации находится в ``/usr/local/nginx-micropvr/conf/nginx.conf``,
пример:
::
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


.. _micropvs-options-description:

Описание параметров micropvs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

pvr_api_host ``str``
  IP-адрес JSON-RPC API процесса micropvr.

pvr_api_port ``int``
  Порт JSON-RPC API процесса micropvr.

ts
  Подключение модуля micropvs.

Остальные параметры стандартные для сервера `nginx <http://nginx.org/en/docs/>`_.

.. _monit-script:

Скрипт для monit
================

Для слежения за процессами micropvr удобно использовать monit, пример скрипта:
::
    check process micropvr with pidfile /var/run/micropvr.pid
        start program = "/etc/init.d/micropvr start" with timeout 60 seconds
        stop program  = "/etc/init.d/micropvr stop"
        if cpu > 60% for 2 cycles then alert
        if cpu > 90% for 5 cycles then restart
        if totalmem > 6000.0 MB for 5 cycles then restart
        if 3 restarts within 5 cycles then timeout
        group micropvr

.. _jsonrpc-api:

*********************
Описание JSON-RPC API
*********************

См. `Документация MicroPVR API <https://smarty.microimpuls.com/docs/micropvr_api/>`_

.. _middleware-integration:

********************************
Интеграция с IPTV/OTT Middleware
********************************

Решение MicroPVR по-умолчанию поддерживается системой Microimpuls IPTV/OTT Middleware (`Smarty <http://mi-smarty-docs.readthedocs.io/>`_).

Для интеграции MicroPVR со сторонней системой Middleware необходимо использовать :ref:`JSON-RPC API <jsonrpc-api>`.

Формат HTTP URL для доступа к записанному контенту на необходимую позицию времени:
::
    http://<host>:8080/ts?channel_id=<channel id>&timestamp=<unix utc timestamp>

Вместо ``<channel_id>`` подставляется уникальный идентификатор канала, соответствующий тому, который был передан при старте записи,
``<unix utc timestamp`` -  время в формате UNIX Timestamp, соответствующее старту передачи по EPG по UTC+0.

.. _troubleshooting:

******************************
Решение проблем и рекомендации
******************************

.. _sysctl.conf:

Рекомендуемые параметры ядра
============================

Изменения нужно вносить в файл /etc/sysctl.conf:
::
    kernel.shmmax = 2473822720
    kernel.shmall = 4097152000
    net.core.rmem_default = 262144
    net.core.rmem_max = 8388608
    net.core.wmem_default = 262144
    net.core.wmem_max = 8388608
    net.ipv4.tcp_syncookies = 1
    net.ipv4.tcp_tw_recycle = 0
    net.ipv4.tcp_tw_reuse = 0
    net.ipv4.tcp_keepalive_time = 10
    net.ipv4.tcp_fin_timeout = 5

Затем выполнить команду для применения изменений:
::
    sysctl -p

.. _limits.conf:

Дополнительные настройки ОС при очень большом количестве одновременных подключений
==================================================================================

В файле ``/etc/security/limits.conf`` необходимо прописать:
::
    *               soft    nofile          16384
    *               hard    nofile          16384
