# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :otuslinux => {
        :box_name => "centos/7",
        :ip_addr => '192.168.11.101',

		
  },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

	  box.vm.provision "shell", inline: <<-SHELL
	      mkdir -p ~root/.ssh
              cp ~vagrant/.ssh/auth* ~root/.ssh
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
		echo 'OnBootSec=30' >> /etc/systemd/system/watchlog.timer
		echo 'OnUnitActiveSec=30' >> /etc/systemd/system/watchlog.timer
		echo 'Unit=watchlog.service' >> /etc/systemd/system/watchlog.timer
		echo '[Install]' >> /etc/systemd/system/watchlog.timer
		echo 'WantedBy=multi-user.target' >> /etc/systemd/system/watchlog.timer
		cat /etc/systemd/system/watchlog.timer
		systemctl enable watchlog.timer
		systemctl start watchlog.timer
		systemctl start watchlog.service
		yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y
		cat /etc/rc.d/init.d/spawn-fcgi
		cp /etc/sysconfig/spawn-fcgi /etc/sysconfig/spawn-fcgi.bak
		sed -e 's/\#S/S/; s/\#O/O/;w /etc/sysconfig/spawn-fcgi' /etc/sysconfig/spawn-fcgi.bak
		rm -f /etc/sysconfig/spawn-fcgi.bak
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
		touch /etc/systemd/system/httpd@.service
		sed -e 's|sysconfig/httpd|sysconfig/httpd-%I|;w /etc/systemd/system/httpd@.service' /usr/lib/systemd/system/httpd.service
		cat /etc/systemd/system/httpd@.service
		echo '# /etc/sysconfig/httpd-first' >> /etc/sysconfig/httpd-first
		echo 'OPTIONS=-f conf/first.conf' >> /etc/sysconfig/httpd-first
		echo '# /etc/sysconfig/httpd-second' >> /etc/sysconfig/httpd-second
		echo 'OPTIONS=-f conf/second.conf' >> /etc/sysconfig/httpd-second
		cat /etc/sysconfig/httpd-first
		cat /etc/sysconfig/httpd-second
		cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/first.conf
		sed -e 's/Listen\ 80/Listen\ 8080/;w /etc/httpd/conf/second.conf' /etc/httpd/conf/first.conf
		echo 'PidFile /var/run/httpd-second.pid' >> /etc/httpd/conf/second.conf
		systemctl start httpd@first
		systemctl start httpd@second
		ss -tnlp | grep httpd
		yum install -y wget java
		mkdir /root/jira
		cd /root/jira
		wget -q https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-software-8.5.1.tar.gz
		tar -xzvf atlassian-jira-software-8.5.1.tar.gz
		/bin/bash /root/jira/atlassian-jira-software-8.5.1-standalone/bin/start-jira.sh
		/bin/bash /root/jira/atlassian-jira-software-8.5.1-standalone/bin/stop-jira.sh
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
	SHELL
      end
  end
end
