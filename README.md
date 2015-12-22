# ![codixis](media/codixis.png)

Codixis standard Symfony LEMP setup 
========
Only for the `Digital Ocean` Lemp stack

## Ubuntu 14.*
### Install dependencies
``` bash
apt-get install acl git php5-cli
curl -s https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

### Add Swap for virtual machine
``` bash
fallocate -l 4G /swapfile 
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
vi /etc/fstab
## Add at the end : /swapfile   none    swap    sw    0   0
```

### Set time for PHP and SQL
``` bash
echo "Europe/Paris" > /etc/timezone
dpkg-reconfigure -f noninteractive tzdata
locale-gen fr_FR.UTF-8
sed -i '$ a\date.timezone = "Europe/Paris"' /etc/php5/cli/php.ini
sed -i '$ a\date.timezone = "Europe/Paris"' /etc/php5/fpm/php.ini
service php5-fpm restart
service mysql restart
```
### Install NewRelic sysmond 
``` bash
wget -O - https://download.newrelic.com/548C16BF.gpg | sudo apt-key add -
sudo sh -c 'echo "deb http://apt.newrelic.com/debian/ newrelic non-free" > /etc/apt/sources.list.d/newrelic.list'
sudo apt-get update
apt-get install newrelic-sysmond
nrsysmond-config --set 
vi /etc/newrelic/nrsysmond.cfg
## Add your licence key 
/etc/init.d/newrelic-sysmond start
```
### Install NewRelic PHP 
``` bash
sudo apt-get install newrelic-php5
sudo newrelic-install install
## Follow setup
```

### Install MySQL to S3 backup 
``` bash
apt-get install s3cmd
s3cmd --configure
## Configure your AWS credentials
apt-get install python-magic
vi /var/www/my.website.net/cron/mysql.sh
```
Add to `/var/www/my.website.net/cron/mysql.sh`  
Replacing : `MyDb`, `MyCompany`, `bucketname`, `MyPassword`
``` bash
#!/bin/sh
current_time=$(date "+%Y%m%d%H%M")
echo "Current Time : $current_time"
export BACKUP_FILE=$current_time.MyDb_db.backup.sql.gz
export DATABASE_SCHEMA_NAME=MyDb
export S3_BUNDLE=mycompany.backups/bucketname/
mysqldump --databases $DATABASE_SCHEMA_NAME -pMyPassword > temp.sql
gzip temp.sql
mv temp.sql.gz $BACKUP_FILE
s3cmd put $BACKUP_FILE s3://$S3_BUNDLE/$BACKUP_FILE
```
Set permissions and add to crontab
``` bash
chmod +x mysql.sh
crontab -e
## Add at the end : 0  0  *  *  *  /var/www/my.website.net/cron/mysql.sh
```
