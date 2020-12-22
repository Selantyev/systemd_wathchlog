# systemd_wathchlog
Systemd log monitoring service

**Cервис, который раз в 30 секунд мониторит лог на предмет наличия ключевого слова**

1. Создать файл с конфигурацией сервиса в каталоге /etc/sysconfig/

`vi /etc/sysconfig/sshd-accept`

```
# Configuration file for my sshd-accept service
# File and word in that file that we will be monit
WORD="Accepted"
LOG=/var/log/secure.log
```

2. Создать скрипт

```
#!/bin/bash
WORD=Accepted
LOG=/var/log/secure
DATE=`date`
if grep $WORD $LOG &> /dev/null
then
    logger "*****$DATE: I found accepted SSH connection!*****"
else
    exit 0
fi
```

`chmod 755 /opt/sshd-accept`

3. Создать юнит для сервиса

`vi /etc/systemd/system/sshd-accept.service`

```
[Unit]
Description=SSH accepted connection monitoring service

[Service]
Type=oneshot
EnvironmentFile=/etc/sysconfig/sshd-accept
ExecStart=/opt/sshd-accept.sh $WORD $LOG

[Install]
WantedBy=multi-user.target
```

4. Создать юнит для таймера

`vi /etc/systemd/system/sshd-accept.timer`

```
[Unit]
Description=Run sshd-accept script every 30 second

[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=sshd-accept.service

[Install]
WantedBy=multi-user.target
```

5. Запустить таймер

`systemctl daemon-reload`
`systemctl start sshd-accept.timer`

6. Проверить содержимое лога

`tail -f -n50 /var/log/secure`

```
Dec 22 11:01:01 centos8 systemd[1]: Starting SSH accepted connection monitoring service...
Dec 22 11:01:01 centos8 root[49701]: *****Tue Dec 22 11:01:01 UTC 2020: I found accepted SSH connection!*****
Dec 22 11:01:01 centos8 systemd[1]: Started SSH accepted connection monitoring service.
Dec 22 11:01:02 centos8 systemd-logind[890]: Session 10 logged out. Waiting for processes to exit.
Dec 22 11:01:02 centos8 systemd-logind[890]: Removed session 10.
Dec 22 11:01:02 centos8 systemd-logind[890]: Session 11 logged out. Waiting for processes to exit.
Dec 22 11:01:02 centos8 systemd-logind[890]: Removed session 11.
Dec 22 11:01:21 centos8 systemd[1]: Started Session 12 of user vagrant.
Dec 22 11:01:21 centos8 systemd-logind[890]: New session 12 of user vagrant.
Dec 22 11:01:34 centos8 systemd[1]: Starting SSH accepted connection monitoring service...
Dec 22 11:01:34 centos8 systemd-logind[890]: New session 13 of user vagrant.
Dec 22 11:01:34 centos8 systemd[1]: Started Session 13 of user vagrant.
Dec 22 11:01:34 centos8 root[49898]: *****Tue Dec 22 11:01:34 UTC 2020: I found accepted SSH connection!*****
Dec 22 11:01:34 centos8 systemd[1]: Started SSH accepted connection monitoring service.
Dec 22 11:02:16 centos8 systemd[1]: Starting SSH accepted connection monitoring service...
Dec 22 11:02:16 centos8 root[50163]: *****Tue Dec 22 11:02:16 UTC 2020: I found accepted SSH connection!*****
Dec 22 11:02:16 centos8 systemd[1]: Started SSH accepted connection monitoring service.
Dec 22 11:03:16 centos8 systemd[1]: Starting SSH accepted connection monitoring service...
Dec 22 11:03:16 centos8 root[50517]: *****Tue Dec 22 11:03:16 UTC 2020: I found accepted SSH connection!*****
Dec 22 11:03:16 centos8 systemd[1]: Started SSH accepted connection monitoring service.
Dec 22 11:04:16 centos8 systemd[1]: Starting SSH accepted connection monitoring service...
Dec 22 11:04:16 centos8 root[50870]: *****Tue Dec 22 11:04:16 UTC 2020: I found accepted SSH connection!*****
Dec 22 11:04:16 centos8 systemd[1]: Started SSH accepted connection monitoring service.
```

**Переписать init-скрипт на unit-файл**

1. Установить пакеты

`yum install -y epel-release && yum install -y spawn-fcgi php php-cli mod_fcgid httpd`

2. Скрипт, который будем переписывать

`less /etc/rc.d/init.d/spawn-fcgi`

3. Отредактируем файл /etc/sysconfig/spawn-fcgi

`vi /etc/sysconfig/spawn-fcgi`

```
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"
```

4. Создать юнит

`vi /etc/systemd/system/spawn-fcgi.service`

```
[Unit]
Description=Spawn-fcgi startup service
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
5. Запустить сервис

`systemctl daemon-reload`
`systemctl start spawn-fcgi.service`

6. Проверить статус сервиса

`systemctl status spawn-fcgi.service`

```
● spawn-fcgi.service - Spawn-fcgi startup service
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2020-12-22 11:34:20 UTC; 20s ago
 Main PID: 1285 (php-cgi)
    Tasks: 33 (limit: 11479)
   Memory: 23.5M
   CGroup: /system.slice/spawn-fcgi.service
           ├─1285 /usr/bin/php-cgi
           ├─1286 /usr/bin/php-cgi
           ├─1287 /usr/bin/php-cgi
           ├─1288 /usr/bin/php-cgi
           ├─1289 /usr/bin/php-cgi
           ├─1290 /usr/bin/php-cgi
           ├─1291 /usr/bin/php-cgi
           ├─1292 /usr/bin/php-cgi
           ├─1293 /usr/bin/php-cgi
           ├─1294 /usr/bin/php-cgi
           ├─1295 /usr/bin/php-cgi
           ├─1296 /usr/bin/php-cgi
           ├─1297 /usr/bin/php-cgi
           ├─1298 /usr/bin/php-cgi
           ├─1299 /usr/bin/php-cgi
           ├─1300 /usr/bin/php-cgi
           ├─1301 /usr/bin/php-cgi
           ├─1302 /usr/bin/php-cgi
           ├─1303 /usr/bin/php-cgi
           ├─1304 /usr/bin/php-cgi
           ├─1305 /usr/bin/php-cgi
           ├─1306 /usr/bin/php-cgi
           ├─1307 /usr/bin/php-cgi
           ├─1308 /usr/bin/php-cgi
           ├─1309 /usr/bin/php-cgi
           ├─1310 /usr/bin/php-cgi
           ├─1311 /usr/bin/php-cgi
           ├─1312 /usr/bin/php-cgi
           ├─1313 /usr/bin/php-cgi
           ├─1314 /usr/bin/php-cgi
           ├─1315 /usr/bin/php-cgi
           ├─1316 /usr/bin/php-cgi
           └─1317 /usr/bin/php-cgi

Dec 22 11:34:20 centos8 systemd[1]: Started Spawn-fcgi startup service.
```
