# Домашнее задание 5

## Systemd

* Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова. Файл и слово должны задаваться в /etc/sysconfig
* Из epel установить spawn-fcgi и переписать init-скрипт на unit-файл. Имя сервиса должно так же называться.
* Дополнить юнит-файл apache httpd возможностьб запустить несколько инстансов сервера с разными конфигами
* Скачать демо-версию Atlassian Jira и переписать основной скрипт запуска на unit-файл
* Задание необходимо сделать с использованием Vagrantfile и proviosioner shell (или ansible, на Ваше усмотрение)

## Процесс решения

### Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова. Файл и слово должны задаваться в /etc/sysconfig

		touch /etc/sysconfig/watchlog
		echo '# Configuration file for my watchdog service' >> /etc/sysconfig/watchlog
		echo '# Place it to /etc/sysconfig' >> /etc/sysconfig/watchlog
		echo '# File and word in that file that we will be monit' >> /etc/sysconfig/watchlog
		echo 'WORD="ALERT"' >> /etc/sysconfig/watchlog
		echo 'LOG=/var/log/watchlog.log' >> /etc/sysconfig/watchlog
		cat /etc/sysconfig/watchlog
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
		touch /etc/systemd/system/watchlog.service
		echo '[Unit]' >> /etc/systemd/system/watchlog.service
		echo 'Description=My watchlog service' >> /etc/systemd/system/watchlog.service
		echo '[Service]' >> /etc/systemd/system/watchlog.service
		echo 'Type=oneshot' >> /etc/systemd/system/watchlog.service
		echo 'EnvironmentFile=/etc/sysconfig/watchlog' >> /etc/systemd/system/watchlog.service
		echo 'ExecStart=/opt/watchlog.sh $WORD $LOG' >> /etc/systemd/system/watchlog.service
		cat /etc/systemd/system/watchlog.service
		touch /etc/systemd/system/watchlog.timer
		echo '[Unit]' >> /etc/systemd/system/watchlog.timer
		echo 'Description=Run watchlog script every 30 second' >> /etc/systemd/system/watchlog.timer
		echo '[Timer]' >> /etc/systemd/system/watchlog.timer
		echo '# Run every 30 second' >> /etc/systemd/system/watchlog.timer
		echo 'OnUnitActiveSec=30' >> /etc/systemd/system/watchlog.timer
		echo 'Unit=watchlog.service' >> /etc/systemd/system/watchlog.timer
		echo '[Install]' >> /etc/systemd/system/watchlog.timer
		echo 'WantedBy=multi-user.target' >> /etc/systemd/system/watchlog.timer
		cat /etc/systemd/system/watchlog.timer
		systemctl start watchlog.timer

### Из epel установить spawn-fcgi и переписать init-скрипт на unit-файл. Имя сервиса должно так же называться.

		yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y
		cat /etc/rc.d/init.d/spawn-fcgi
		cp /etc/sysconfig/spawn-fcgi /root/spawn-fcgi.bak
		sed -e 's/\#S/S/; s/\#O/O/;w /etc/sysconfig/spawn-fcgi' spawn-fcgi.bak
		cat /etc/sysconfig/spawn-fcgi
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
		systemctl start spawn-fcgi
		systemctl status spawn-fcgi

### Дополнить юнит-файл apache httpd возможностьб запустить несколько инстансов сервера с разными конфигами

		sed -e 's/sysconfig\/httpd/sysconfig\/httpd-\%I/;w /etc/systemd/system/httpd@.service' /usr/lib/systemd/system/httpd.service
		echo '# /etc/sysconfig/httpd-first' >> /etc/sysconfig/httpd-first
		echo 'OPTIONS=-f conf/first.conf' >> /etc/sysconfig/httpd-first
		echo '# /etc/sysconfig/httpd-second' >> /etc/sysconfig/httpd-second
		echo 'OPTIONS=-f conf/second.conf' >> /etc/sysconfig/httpd-second
		cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/first.conf
		sed -e 's/Listen\ 80/Listen\ 8080/;w /etc/httpd/conf/second.conf' /etc/httpd/conf/first.conf
		echo 'PidFile /var/run/httpd-second.pid' >> /etc/httpd/conf/second.conf
		systemctl start httpd@first
		systemctl start httpd@second
		ss -tnlp | grep httpd


### Скачать демо-версию Atlassian Jira и переписать основной скрипт запуска на unit-файл

		yum install -y wget java
		wget https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-software-8.5.1.tar.gz
		mkdir /root/jira
		cd /root/jira
		tar -xzvf atlassian-jira-software-8.5.1.tar.gz
		/bin/bash /root/jira/atlassian-jira-software-8.5.1-standalone/bin/start-jira.sh
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
		systemctl daemon-reload
		systemctl start jira
		systemctl status jira
