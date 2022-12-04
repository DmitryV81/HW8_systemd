Домашняя работа №8. Инициализация системы и systemd.

Часть 1. Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова. Файл и слово должны задаваться в
/etc/sysconfig

Ход работы:

1. В каталоге /etc/sysconfig создаем файл watchlog, в котором находится конфигурация сервиса:
```
[root@systemd ~]#cat /etc/sysconfig/watchlog 
#Configuration file for my watchdog service
#Place it to /etc/sysconfig

#File and word in that file we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log
```
2. В каталоге var/log создаем файл watchlog.log, в который записываем строки, содержащие слово "ALERT":
```
[root@systemd ~]# cat /var/log/watchlog.log 
ALERT : NEW allert there
Another ALERT !
ALERT
ALERT
```
3. В каталоге /opt создаем скрипт watchlog.sh, который с помощью команды logger отправляет лог в системный журнал
```
[root@systemd ~]#cat /opt/watchlog.sh 
#!/bin/bash
WORD=$1
LOG=$2
DATE=`date`

if grep $WORD $LOG &> /dev/null
then
  logger "$DATE: I found word, Master!"
else
  exit 0
fi
```
4. В каталоге /etc/systemd/system создаем юнит для сервиса watchlog.service:
```
[root@systemd ~]# cat /etc/systemd/system/watchlog.service 
[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/sysconfig/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG
```
5. Там же создаем юнит для таймера watchlog.timer:
```
[root@systemd ~]# cat /etc/systemd/system/watchlog.timer   
[Unit]
Description=Run watchlog script every 30 second

[Timer]
#Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target
```
6. Запускаем таймер: systemctl start watchlog.timer и проверяем вывод
```
[root@systemd ~]# systemctl start watchlog.timer        
[root@systemd ~]# tail -f /var/log/messages
Dec  4 20:01:01 systemd systemd: Created slice User Slice of root.
Dec  4 20:01:01 systemd systemd: Started Session 5 of user root.
Dec  4 20:01:01 systemd systemd: Removed slice User Slice of root.
Dec  4 20:07:10 systemd systemd: Started Run watchlog script every 30 second.
Dec  4 20:11:07 systemd systemd: Starting My watchlog service...
Dec  4 20:11:07 systemd root: Sun Dec  4 20:11:07 UTC 2022: I found word, Master!
Dec  4 20:11:07 systemd systemd: Started My watchlog service.
Dec  4 20:11:38 systemd systemd: Starting My watchlog service...
Dec  4 20:11:38 systemd root: Sun Dec  4 20:11:38 UTC 2022: I found word, Master!
Dec  4 20:11:38 systemd systemd: Started My watchlog service.
Dec  4 20:12:08 systemd systemd: Starting My watchlog service...
Dec  4 20:12:08 systemd root: Sun Dec  4 20:12:08 UTC 2022: I found word, Master!
Dec  4 20:12:08 systemd systemd: Started My watchlog service.
Dec  4 20:12:38 systemd systemd: Starting My watchlog service...
Dec  4 20:12:38 systemd root: Sun Dec  4 20:12:38 UTC 2022: I found word, Master!
Dec  4 20:12:38 systemd systemd: Started My watchlog service.
Dec  4 20:13:08 systemd systemd: Starting My watchlog service...
Dec  4 20:13:08 systemd root: Sun Dec  4 20:13:08 UTC 2022: I found word, Master!
Dec  4 20:13:08 systemd systemd: Started My watchlog service.
```

Часть 2. Из epel установить spawn-fcgi и переписать init-скрипт на unit-файл. Имя сервиса должно также называться.

Ход работы:

1. Устанавливаем spawn-fcgi и необходимые для него пакеты:
```
 yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y
```
2. Вносим правки в файл /etc/sysconfig/spawn-fcgi:
```
[root@systemd ~]# cat /etc/sysconfig/spawn-fcgi
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"
```
3. В каталоге /etc/systemd/system создаём юнит-файл spawn-fcgi.service:
```
[root@systemd ~]# cat /etc/systemd/system/spawn-fcgi.service 
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/sysconfig/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target
```
4. Запускаем и проверяем:
```
[root@systemd ~]# systemctl start spawn-fcgi
[root@systemd ~]# systemctl status spawn-fcgi
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2022-12-04 20:25:19 UTC; 12s ago
 Main PID: 3489 (php-cgi)
   CGroup: /system.slice/spawn-fcgi.service
           ├─3489 /usr/bin/php-cgi
           ├─3490 /usr/bin/php-cgi
           ├─3491 /usr/bin/php-cgi
           ├─3492 /usr/bin/php-cgi
           ├─3493 /usr/bin/php-cgi
           ├─3494 /usr/bin/php-cgi
           ├─3495 /usr/bin/php-cgi
           ├─3496 /usr/bin/php-cgi
           ├─3497 /usr/bin/php-cgi
           ├─3498 /usr/bin/php-cgi
           ├─3499 /usr/bin/php-cgi
           ├─3500 /usr/bin/php-cgi
           ├─3501 /usr/bin/php-cgi
           ├─3502 /usr/bin/php-cgi
           ├─3503 /usr/bin/php-cgi
           ├─3504 /usr/bin/php-cgi
           ├─3505 /usr/bin/php-cgi
           ├─3506 /usr/bin/php-cgi
           ├─3507 /usr/bin/php-cgi
           ├─3508 /usr/bin/php-cgi
           ├─3509 /usr/bin/php-cgi
           ├─3510 /usr/bin/php-cgi
           ├─3511 /usr/bin/php-cgi
           ├─3512 /usr/bin/php-cgi
           ├─3513 /usr/bin/php-cgi
           ├─3514 /usr/bin/php-cgi
           ├─3515 /usr/bin/php-cgi
           ├─3516 /usr/bin/php-cgi
           ├─3517 /usr/bin/php-cgi
           ├─3518 /usr/bin/php-cgi
           ├─3519 /usr/bin/php-cgi
           ├─3520 /usr/bin/php-cgi
           └─3521 /usr/bin/php-cgi

Dec 04 20:25:19 systemd systemd[1]: Started Spawn-fcgi startup service by Otus.
Hint: Some lines were ellipsized, use -l to show in full.
```

Часть 3. Дополнить юнит-файл apache httpd возможностью запустить несколько инстансов сервера с разными конфигами

Ход работы:

1. Вносим правки в шаблон в конфигурации файла окружения, который находится в каталоге /usr/lib/systemd/system. Изначально файл называется httpd.service. Переименуем его в httpd@service.
```
[root@systemd ~]# cat /usr/lib/systemd/system/httpd@.service 
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=man:httpd(8)
Documentation=man:apachectl(8)

[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/httpd-%i 
ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
ExecStop=/bin/kill -WINCH ${MAINPID}
# We want systemd to give httpd some time to finish gracefully, but still want
# it to kill httpd after TimeoutStopSec if something went wrong during the
# graceful stop. Normally, Systemd sends SIGTERM signal right after the
# ExecStop, which would kill httpd. We are sending useless SIGCONT here to give
# httpd time to finish.
KillSignal=SIGCONT
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
В описании сервиса в строке EnvironmentFile=/etc/sysconfig/httpd-%i добавили спецификатор %i для подстановки имён инстансов.

2. В каталоге  /etc/sysconfig создаём два файла окружения httpd-first и httpd-second, в которых задается опция для запуска веб-сервера с необходимым конфигурационным файлом:
```
[root@systemd ~]# cat /etc/sysconfig/httpd-first 
#
# This file can be used to set additional environment variables for
# the httpd process, or pass additional options to the httpd
# executable.
#
# Note: With previous versions of httpd, the MPM could be changed by
# editing an "HTTPD" variable here.  With the current version, that
# variable is now ignored.  The MPM is a loadable module, and the
# choice of MPM can be changed by editing the configuration file
# /etc/httpd/conf.modules.d/00-mpm.conf.
# 

#
# To pass additional options (for instance, -D definitions) to the
# httpd binary at startup, set OPTIONS here.
#
OPTIONS="-f /etc/httpd/conf/first.conf"

#
# This setting ensures the httpd process is started in the "C" locale
# by default.  (Some modules will not behave correctly if
# case-sensitive string comparisons are performed in a different
# locale.)
#
LANG=C
```
```
[root@systemd ~]# cat /etc/sysconfig/httpd-second 
#
# This file can be used to set additional environment variables for
# the httpd process, or pass additional options to the httpd
# executable.
#
# Note: With previous versions of httpd, the MPM could be changed by
# editing an "HTTPD" variable here.  With the current version, that
# variable is now ignored.  The MPM is a loadable module, and the
# choice of MPM can be changed by editing the configuration file
# /etc/httpd/conf.modules.d/00-mpm.conf.
# 

#
# To pass additional options (for instance, -D definitions) to the
# httpd binary at startup, set OPTIONS here.
#
OPTIONS="-f /etc/httpd/conf/second.conf"

#
# This setting ensures the httpd process is started in the "C" locale
# by default.  (Some modules will not behave correctly if
# case-sensitive string comparisons are performed in a different
# locale.)
#
LANG=C
```
3. В каталоге с конфигурацией httpd /etc/httpd/conf создаем два файла конфигурации first.conf и second.conf. В каждом из них необходимо изменить порт и PidFile. В first.conf оставляем порт 80 и прописываем путь до пид-файла /var/run/httpd-first.pid. В second.conf порт меняем на 8080 и путь до пид-файла /var/run/httpd-second.pid
```
[root@systemd ~]# cat /etc/httpd/conf/first.conf 

PidFile /var/run/httpd-first.pid
Listen 80
[root@systemd ~]# cat /etc/httpd/conf/second.conf 

PidFile /var/run/httpd-second.pid
Listen 8080
```
Часть вывода опущена.

4. Проверяем:
```
[root@systemd ~]# systemctl start httpd@first.service
[root@systemd ~]# systemctl start httpd@second.service
[root@systemd ~]# ss -ntupl
Netid  State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
udp    UNCONN     0      0         *:961                   *:*                   users:(("rpcbind",pid=367,fd=7))
udp    UNCONN     0      0      127.0.0.1:323                   *:*                   users:(("chronyd",pid=366,fd=5))
udp    UNCONN     0      0         *:68                    *:*                   users:(("dhclient",pid=1844,fd=6))
udp    UNCONN     0      0         *:111                   *:*                   users:(("rpcbind",pid=367,fd=6))
udp    UNCONN     0      0      [::]:961                [::]:*                   users:(("rpcbind",pid=367,fd=10))
udp    UNCONN     0      0         [::1]:323                [::]:*                   users:(("chronyd",pid=366,fd=6))
udp    UNCONN     0      0      [::]:111                [::]:*                   users:(("rpcbind",pid=367,fd=9))
tcp    LISTEN     0      128       *:111                   *:*                   users:(("rpcbind",pid=367,fd=8))
tcp    LISTEN     0      128       *:22                    *:*                   users:(("sshd",pid=680,fd=3))
tcp    LISTEN     0      100    127.0.0.1:25                    *:*                   users:(("master",pid=863,fd=13))
tcp    LISTEN     0      128    [::]:111                [::]:*                   users:(("rpcbind",pid=367,fd=11))
tcp    LISTEN     0      128    [::]:8080               [::]:*                   users:(("httpd",pid=2973,fd=4),("httpd",pid=2972,fd=4),("httpd",pid=2971,fd=4),("httpd",pid=2970,fd=4),("httpd",pid=2969,fd=4),("httpd",pid=2968,fd=4),("httpd",pid=2967,fd=4))
tcp    LISTEN     0      128    [::]:80                 [::]:*                   users:(("httpd",pid=2927,fd=4),("httpd",pid=2926,fd=4),("httpd",pid=2925,fd=4),("httpd",pid=2924,fd=4),("httpd",pid=2923,fd=4),("httpd",pid=2922,fd=4),("httpd",pid=2921,fd=4))
tcp    LISTEN     0      128    [::]:22                 [::]:*                   users:(("sshd",pid=680,fd=4))
tcp    LISTEN     0      100       [::1]:25                 [::]:*                   users:(("master",pid=863,fd=14))
```
