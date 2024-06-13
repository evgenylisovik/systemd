# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.box = "centos8stream"
  config.vm.box_url = "https://cloud.centos.org/centos/8-stream/x86_64/images/CentOS-Stream-Vagrant-8-20230501.0.x86_64.vagrant-virtualbox.box"
  config.vm.provision "shell", inline: <<-SHELL
  touch /etc/sysconfig/watchlog
  touch /opt/watchlog.sh
  echo -e '# Configuration file for my watchlog service\n# Place it to /etc/sysconfig\n\n# File and word in that file that we will be monit\nWORD="ALERT"\nLOG=/var/log/watchlog.log' > /etc/sysconfig/watchlog 
  echo -e '[Unit]\nDescription=Borg Backup\n\n[Service]\n ALERT Type=oneshot\n\nEnvironment="BORG_PASSPHRASE=testpassphrase"\nEnvironment=REPO=borg@192.168.56.11:/var/backup/\nEnvironment=BACKUP_TARGET=/etc\nExecStart=/bin/borg create \\\n   --stats             \\\n   ${REPO}::etc-{now:%%Y-%%m-%%d_%%H:%%M:%%S} ${BACKUP_TARGET}\nExecStart=/bin/borg check ${REPO}\nExecStart=/bin/borg prune \\\n     --keep-minutely 60        \\\n      --keep-hourly 24     \\\n    --keep-daily  90      \\\n    --keep-monthly 12     \\\n   --keep-yearly 1   \\\n${REPO}\nStandardOutput=syslog\nStandardError=syslog\nSyslogIdentifier=borg' > /var/log/watchlog.log 
  echo -e '#!/bin/bash\nWORD=$1\nLOG=$2\nDATE=`date`\nif grep $WORD $LOG &> /dev/null\nthen\nlogger "$DATE: I found word, Master!"\nelse\nexit 0\nfi' > /opt/watchlog.sh
  chmod +x /opt/watchlog.sh
  echo -e '[Unit]\nDescription=My watchlog service\n\n[Service]\nType=oneshot\n\nEnvironmentFile=/etc/sysconfig/watchlog\nExecStart=/opt/watchlog.sh $WORD $LOG' > /etc/systemd/system/watchlog.service 
  echo -e '[Unit]\nDescription=Run watchlog script every 30 second\n\n[Timer]\n# Run every 30 second\nOnUnitActiveSec=30\nUnit=watchlog.service\n\n[Install]\nWantedBy=multi-user.target' > /etc/systemd/system/watchlog.timer
  systemctl start watchlog.service
  systemctl start watchlog.timer
  sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
  sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
  yum install epel-release -y && yum install spawn-fcgi php php-cli -y
  sed '7,8s/#/ /' /etc/sysconfig/spawn-fcgi > /etc/sysconfig/spawn-fcgi-izmenen
  rm -f /etc/sysconfig/spawn-fcgi
  mv /etc/sysconfig/spawn-fcgi-izmenen /etc/sysconfig/spawn-fcgi
  echo -e '[Unit]\nDescription=Spawn-fcgi startup service by Otus\nAfter=network.target\n\n[Service]\n Type=simple\n\PIDFile=/var/run/spawn-fcgi.pid\nEnvironmentFile=/etc/sysconfig/spawn-fcgi\nExecStart=/usr/bin/spawn-fcgi -n $OPTIONS\nKillMode=process\n\n[Install]\nWantedBy=multi-user.target' > /etc/systemd/system/spawn-fcgi.service 
  systemctl start spawn-fcgi
  yum install nginx -y
  cat /vagrant/nginx@service > /etc/systemd/system/nginx@.service
  cp /vagrant/nginx@first /etc/nginx/nginx-first.conf
  cp /vagrant/nginx@second /etc/nginx/nginx-second.conf
  systemctl start nginx@first.service
  systemctl start nginx@second.service
  SHELL
end

