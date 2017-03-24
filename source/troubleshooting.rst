.. _troubleshooting:

*********************************
C. Решение проблем и рекомендации
*********************************

.. _sysctl.conf:

C.1. Рекомендуемые параметры ядра
=================================

Изменения нужно вносить в файл /etc/sysctl.conf: ::

    kernel.shmmax = 2473822720
    kernel.shmall = 4097152000
    net.core.rmem_default = 8388608
    net.core.rmem_max = 16777216
    net.core.wmem_default = 8388608
    net.core.wmem_max = 16777216
    net.ipv4.tcp_syncookies = 1
    net.ipv4.tcp_tw_recycle = 0
    net.ipv4.tcp_tw_reuse = 0
    net.ipv4.tcp_keepalive_time = 10
    net.ipv4.tcp_fin_timeout = 5

Затем выполнить команду для применения изменений: ::

    sysctl -p

.. _limits.conf:

C.2. Дополнительные настройки ОС при очень большом количестве одновременных подключений
=======================================================================================

В файле ``/etc/security/limits.conf`` необходимо прописать: ::

    *               soft    nofile          16384
    *               hard    nofile          16384
