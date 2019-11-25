# Домашнее задание 5

## Systemd

* Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова. Файл и слово должны задаваться в /etc/sysconfig
* Из epel установить spawn-fcgi и переписать init-скрипт на unit-файл. Имя сервиса должно так же называться.
* Дополнить юнит-файл apache httpd возможностьб запустить несколько инстансов сервера с разными конфигами
* Скачать демо-версию Atlassian Jira и переписать основной скрипт запуска на unit-файл
* Задание необходимо сделать с использованием Vagrantfile и proviosioner shell (или ansible, на Ваше усмотрение)

## Процесс решения

[Vagrantfile](Vagrantfile) содержащий скрипт для настройки ВМ

### Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова. Файл и слово должны задаваться в /etc/sysconfig

Создаю конфиг с переменными для сервиса

		touch /etc/sysconfig/watchlog
		echo '# Configuration file for my watchdog service' >> /etc/sysconfig/watchlog
		echo '# Place it to /etc/sysconfig' >> /etc/sysconfig/watchlog
		echo '# File and word in that file that we will be monit' >> /etc/sysconfig/watchlog
		echo 'WORD="ALERT"' >> /etc/sysconfig/watchlog
		echo 'LOG=/var/log/watchlog.log' >> /etc/sysconfig/watchlog
		cat /etc/sysconfig/watchlog

Создаю произвольный лог в котором появляется нужное слово

		touch /var/log/watchlog.log
		echo '...string01' >> /var/log/watchlog.log
		echo '...string02' >> /var/log/watchlog.log
		echo '...string03' >> /var/log/watchlog.log
		echo '...string04' >> /var/log/watchlog.log
		echo '...string05' >> /var/log/watchlog.log
		echo '.... ALERT.' >> /var/log/watchlog.log
		echo '...string06' >> /var/log/watchlog.log
		echo '...string07' >> /var/log/watchlog.log
		echo '...string08' >> /var/log/watchlog.log
		echo '...string09' >> /var/log/watchlog.log
		echo '...string10' >> /var/log/watchlog.log
		echo '...string11' >> /var/log/watchlog.log
		echo '..  ALERT .' >> /var/log/watchlog.log
		echo '...string12' >> /var/log/watchlog.log
		cat /var/log/watchlog.log

Создаю скрипт, выолпняющий поиск по слову и логу из конфига, разрешаю его выполнение

		touch /opt/watchlog.sh
		echo '#!/bin/bash' >> /opt/watchlog.sh
		echo 'WORD=$1' >> /opt/watchlog.sh
		echo 'LOG=$2' >> /opt/watchlog.sh
		echo 'DATE=`date`' >> /opt/watchlog.sh
		echo 'if grep $WORD $LOG &> /dev/null' >> /opt/watchlog.sh
		echo 'then' >> /opt/watchlog.sh
		echo 'logger "$DATE: I found word, Master!"' >> /opt/watchlog.sh
		echo 'else' >> /opt/watchlog.sh
		echo 'exit 0' >> /opt/watchlog.sh
		echo 'fi' >> /opt/watchlog.sh
		chmod o+x /opt/watchlog.sh
		cat /opt/watchlog.sh

Создаю oneshot юнит в systemd указывая что запускать и какой конфиг брать

		touch /etc/systemd/system/watchlog.service
		echo '[Unit]' >> /etc/systemd/system/watchlog.service
		echo 'Description=My watchlog service' >> /etc/systemd/system/watchlog.service
		echo '[Service]' >> /etc/systemd/system/watchlog.service
		echo 'Type=oneshot' >> /etc/systemd/system/watchlog.service
		echo 'EnvironmentFile=/etc/sysconfig/watchlog' >> /etc/systemd/system/watchlog.service
		echo 'ExecStart=/opt/watchlog.sh $WORD $LOG' >> /etc/systemd/system/watchlog.service
		cat /etc/systemd/system/watchlog.service

Создаю таймер для сервиса (30с после загрузки и каждые 30сек) и запускаю его

		touch /etc/systemd/system/watchlog.timer
		echo '[Unit]' >> /etc/systemd/system/watchlog.timer
		echo 'Description=Run watchlog script every 30 second' >> /etc/systemd/system/watchlog.timer
		echo '[Timer]' >> /etc/systemd/system/watchlog.timer
		echo '# Run every 30 second' >> /etc/systemd/system/watchlog.timer
		echo 'OnBootSec=30' >> /etc/systemd/system/watchlog.timer
		echo 'OnUnitActiveSec=30' >> /etc/systemd/system/watchlog.timer
		echo 'Unit=watchlog.service' >> /etc/systemd/system/watchlog.timer
		echo '[Install]' >> /etc/systemd/system/watchlog.timer
		echo 'WantedBy=multi-user.target' >> /etc/systemd/system/watchlog.timer
		cat /etc/systemd/system/watchlog.timer
		systemctl enable watchlog.timer
		systemctl start watchlog.timer
		systemctl start watchlog.service


### Из epel установить spawn-fcgi и переписать init-скрипт на unit-файл. Имя сервиса должно так же называться.

Устанавливаю репозиторий и пакеты + сервер httpd

		yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y

через sed убираю комментарии с опций в конфиге spawn-fcgi

		cp /etc/sysconfig/spawn-fcgi /root/spawn-fcgi.bak
		sed -e 's/\#S/S/; s/\#O/O/;w /etc/sysconfig/spawn-fcgi' spawn-fcgi.bak
		cat /etc/sysconfig/spawn-fcgi

создаю юнит 

		touch /etc/systemd/system/spawn-fcgi.service
		echo '[Unit]' >> /etc/systemd/system/spawn-fcgi.service
		echo 'Description=Spawn-fcgi startup service by Otus' >> /etc/systemd/system/spawn-fcgi.service
		echo 'After=network.target' >> /etc/systemd/system/spawn-fcgi.service
		echo '[Service]' >> /etc/systemd/system/spawn-fcgi.service
		echo 'Type=simple' >> /etc/systemd/system/spawn-fcgi.service
		echo 'PIDFile=/var/run/spawn-fcgi.pid' >> /etc/systemd/system/spawn-fcgi.service
		echo 'EnvironmentFile=/etc/sysconfig/spawn-fcgi' >> /etc/systemd/system/spawn-fcgi.service
		echo 'ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS' >> /etc/systemd/system/spawn-fcgi.service
		echo 'KillMode=process' >> /etc/systemd/system/spawn-fcgi.service
		echo '[Install]' >> /etc/systemd/system/spawn-fcgi.service
		echo 'WantedBy=multi-user.target' >> /etc/systemd/system/spawn-fcgi.service
		cat /etc/systemd/system/spawn-fcgi.service

Проверяю что сервис стартует и работает

		systemctl start spawn-fcgi
		systemctl status spawn-fcgi
		● spawn-fcgi.service - Spawn-fcgi startup service by Otus
		   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
		   Active: active (running) since Mon 2019-11-25 10:36:35 UTC; 26min ago
		 Main PID: 5682 (php-cgi)
		   CGroup: /system.slice/spawn-fcgi.service
		           ├─5682 /usr/bin/php-cgi
		           ├─5689 /usr/bin/php-cgi
		           ├─5690 /usr/bin/php-cgi
		           ├─5691 /usr/bin/php-cgi
		           ├─5692 /usr/bin/php-cgi
		           ├─5693 /usr/bin/php-cgi
		           ├─5694 /usr/bin/php-cgi
		           ├─5695 /usr/bin/php-cgi
		           ├─5696 /usr/bin/php-cgi
		           ├─5697 /usr/bin/php-cgi
		           ├─5698 /usr/bin/php-cgi
		           ├─5699 /usr/bin/php-cgi
		           ├─5700 /usr/bin/php-cgi
		           ├─5701 /usr/bin/php-cgi
		           ├─5702 /usr/bin/php-cgi
		           ├─5703 /usr/bin/php-cgi
		           ├─5704 /usr/bin/php-cgi
		           ├─5705 /usr/bin/php-cgi
		           ├─5706 /usr/bin/php-cgi
		           ├─5707 /usr/bin/php-cgi
		           ├─5708 /usr/bin/php-cgi
		           ├─5709 /usr/bin/php-cgi
		           ├─5710 /usr/bin/php-cgi
		           ├─5711 /usr/bin/php-cgi
		           ├─5712 /usr/bin/php-cgi
		           ├─5713 /usr/bin/php-cgi
		           ├─5714 /usr/bin/php-cgi
		           ├─5715 /usr/bin/php-cgi
		           ├─5716 /usr/bin/php-cgi
		           ├─5717 /usr/bin/php-cgi
		           ├─5718 /usr/bin/php-cgi
		           ├─5719 /usr/bin/php-cgi
		           └─5720 /usr/bin/php-cgi


### Дополнить юнит-файл apache httpd возможностьб запустить несколько инстансов сервера с разными конфигами

Через sed из оригинального /usr/lib/systemd/system/httpd.service создаю новый юнит /etc/systemd/system/httpd@.service с параметров httpd-%I

		sed -e 's/sysconfig\/httpd/sysconfig\/httpd-\%I/;w /etc/systemd/system/httpd@.service' /usr/lib/systemd/system/httpd.service

Создаю два конфига в sysconfig для двух разных экземпляров

		echo '# /etc/sysconfig/httpd-first' >> /etc/sysconfig/httpd-first
		echo 'OPTIONS=-f conf/first.conf' >> /etc/sysconfig/httpd-first
		echo '# /etc/sysconfig/httpd-second' >> /etc/sysconfig/httpd-second
		echo 'OPTIONS=-f conf/second.conf' >> /etc/sysconfig/httpd-second

Копирую оригиинальный httpd.conf и конфиг первого сервиса, и с помощью sed создаю второй изменив в нем номер порта и добавив Pid

		cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/first.conf
		sed -e 's/Listen\ 80/Listen\ 8080/;w /etc/httpd/conf/second.conf' /etc/httpd/conf/first.conf
		echo 'PidFile /var/run/httpd-second.pid' >> /etc/httpd/conf/second.conf

Запускаю оба сервиса и вижу что экземпляры слушают разные порты

		systemctl start httpd@first
		systemctl start httpd@second
		ss -tnlp | grep httpd
		LISTEN     0      128         :::8080                    :::*                   users:(("httpd",pid=5738,fd=4),("httpd",pid=5737,fd=4),("httpd",pid=5736,fd=4),("httpd",pid=5735,fd=4),("httpd",pid=5734,fd=4),("httpd",pid=5733,fd=4),("httpd",pid=5732,fd=4))
		LISTEN     0      128         :::80                      :::*                   users:(("httpd",pid=5730,fd=4),("httpd",pid=5729,fd=4),("httpd",pid=5728,fd=4),("httpd",pid=5727,fd=4),("httpd",pid=5726,fd=4),("httpd",pid=5725,fd=4),("httpd",pid=5724,fd=4))



### Скачать демо-версию Atlassian Jira и переписать основной скрипт запуска на unit-файл

Устанавливаю wget и java для jira

		yum install -y wget java

скачиваю архив с jira, распаковываю

		mkdir /root/jira
		cd /root/jira
		wget -q https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-software-8.5.1.tar.gz
		tar -xzvf atlassian-jira-software-8.5.1.tar.gz

проверяю что стартует

		/bin/bash /root/jira/atlassian-jira-software-8.5.1-standalone/bin/start-jira.sh
		/bin/bash /root/jira/atlassian-jira-software-8.5.1-standalone/bin/stop-jira.sh

Создаю unit указав start и stop скрипты, pid

		echo '[Unit]' >> /etc/systemd/system/jira.service
		echo 'Description=Atlassian Jira' >> /etc/systemd/system/jira.service
		echo 'After=network.target' >> /etc/systemd/system/jira.service
		echo '[Service]' >> /etc/systemd/system/jira.service
		echo 'Type=forking' >> /etc/systemd/system/jira.service
		echo 'User=root' >> /etc/systemd/system/jira.service
		echo 'PIDFile=/root/jira/atlassian-jira-software-8.5.1-standalone/work/catalina.pid' >> /etc/systemd/system/jira.service
		echo 'ExecStart=/root/jira/atlassian-jira-software-8.5.1-standalone/bin/start-jira.sh' >> /etc/systemd/system/jira.service
		echo 'ExecStop=/root/jira/atlassian-jira-software-8.5.1-standalone/bin/stop-jira.sh' >> /etc/systemd/system/jira.service
		echo '[Install]' >> /etc/systemd/system/jira.service
		echo 'WantedBy=multi-user.target' >> /etc/systemd/system/jira.service

Перечитываю конфиги сервисов, запускаю jira и проверяю, что запустилось

		systemctl daemon-reload
		systemctl start jira
		systemctl status jira

		● jira.service - Atlassian Jira
		   Loaded: loaded (/etc/systemd/system/jira.service; disabled; vendor preset: disabled)
		   Active: active (running) since Mon 2019-11-25 10:45:42 UTC; 1min 31s ago
		  Process: 6067 ExecStart=/root/jira/atlassian-jira-software-8.5.1-standalone/bin/start-jira.sh (code=exited, status=0/SUCCESS)
		 Main PID: 6104 (java)
		   CGroup: /system.slice/jira.service
		           └─6104 /usr/bin/java -Djava.util.logging.config.file=/root/jira/atlassian-jira-software-8.5.1-standalone/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Xms384m -Xmx2048m -XX:Initi...
		
		Nov 25 10:45:42 otuslinux start-jira.sh[6067]: MMMMMM
		Nov 25 10:45:42 otuslinux start-jira.sh[6067]: +MMMMM
		Nov 25 10:45:42 otuslinux start-jira.sh[6067]: MMMMM
		Nov 25 10:45:42 otuslinux start-jira.sh[6067]: `UOJ
		Nov 25 10:45:42 otuslinux start-jira.sh[6067]: Atlassian Jira
		Nov 25 10:45:42 otuslinux start-jira.sh[6067]: Version : 8.5.1
		Nov 25 10:45:42 otuslinux start-jira.sh[6067]: If you encounter issues starting or stopping JIRA, please see the Troubleshooting guide at https://docs.atlassian.com/jira/jadm-docs-085/Troubleshooting+installation
		Nov 25 10:45:42 otuslinux start-jira.sh[6067]: Server startup logs are located in /root/jira/atlassian-jira-software-8.5.1-standalone/logs/catalina.out
		Nov 25 10:45:42 otuslinux start-jira.sh[6067]: Tomcat started.
		Nov 25 10:45:42 otuslinux systemd[1]: Started Atlassian Jira.

