Requirements:
Server with CentOS 7 with root privileges

- Disable SELinux
setenforce 0 sed -i 's/enforcing/disabled/g' /etc/selinux/config

- Install the requirements packages
yum install epel-release httpd php php-common php-mysqlnd php-mbstring php-gd mariadb-server mod_ssl zip wget unzip -y

- Configure and start both httpd and mariadb services
sed -i 's/AllowOverride none/AllowOverride all/g' /etc/httpd/conf/httpd.conf
systemctl start httpd && systemctl start mariadb

- Setup the database
mysql 
CREATE DATABASE <name>; 
GRANT ALL PRIVILEGES ON <name>.* TO '<name>'@'localhost' IDENTIFIED BY '<password>';

- Get and configure Joomla
wget https://github.com/joomla/joomla-cms/releases/download/3.6.4/Joomla_3.6.4-Stable-Full_Package.zip
unzip Joomla_3.6.4-Stable-Full_Package.zip -d /var/www/html/
chown -R apache:apache /var/www/html

- Open a browser and go to localhost
Set Parameters
 

Set DB parameters configured previously
 

Set Parameters

 

Select the following option
 

Select the following options

 

- Backup the webserver logs with rotation of 7 days
yum update && yum install logrotate -y
yum install vim -y
vim /etc/logrotate.d/httpd
- put the following parameters in the file
/var/log/httpd/*log {
        daily
        rotate 7
        size 10M
        compress
}

- Notification when more than 10 4xx requests are returned by application
yum install mailx -y
cd /etc/httpd/logs/
vim check_errors.sh
- put the following parameters in the file
#!/bin/bash
output=$(grep -i -o 'HTTP/1.1" 4[0-9][0-9]' * | wc -l)
if [ $output -gt 10 ]
then
  echo "There are 10 or more 4** errors on your application" | mail -s "Joomla errors" receiver@example.com
fi

chmod 755 check_errors.sh
crontab -e
- put the following parameters in the file
0 */1 * * * root /etc/httpd/logs/check_errors.sh
