# sstatus

sstaus is designed to be a quick server status checker. Written primarily for CentOS + cPanel, these scripts check for common settings and configurations that can help point you towards the source of an issue. Also, please note that these are not "offical" in any capacity but you are welcome to utilize them if you would like.


## Running via command line

I recommend just copy and pasting the one liners over to avoid downloading files or scripts on the customer's servers. Each script checks for specfic setting but sstatus_all runs them all and is what I primarly utilize. If sstatus_all is what you wanted to us, you would copy the following:

```
(BOLD='\e[1m';LBLU='\e[94m';NC='\e[0m';echo "";echo "https://git.liquidweb.com/jmaul/one_liners/tree/master/sstatus";echo "";echo "";echo -e "${BOLD}-----General Stats-----${NC}";echo "";echo -e "${BOLD}${LBLU}Hostname: "${NC};hostname;echo ""; echo -e "${BOLD}${LBLU}Uptime:"${NC};uptime;echo "";echo -e "${BOLD}${LBLU}Operating System:${NC}" ;if [[ -a /etc/redhat-release ]] ;then cat /etc/redhat-release|head -n1 ;fi ;if [[ -a /etc/lsb-release ]] ;then lsb_release -a ;fi ;if [[ -a /etc/debian_version ]] ; then cat /etc/debian_version ;fi ;echo "";echo -e "${BOLD}${LBLU}Disk Usage:${NC}" ;df -h ;df -Ph| awk '0+$5 >= 80 {print}' ;echo ""; echo -e "${BOLD}${LBLU}Memory Usage:${NC}" ;free -h ;echo ""; echo -e "${BOLD}${LBLU}Potential OOM notifications:${NC} $(grep -a Kill /var/log/messages|wc -l)"; echo -en "${BOLD}${LBLU}CPU(s): "${NC};grep --count ^processor /proc/cpuinfo;echo "";echo -e "${BOLD}-----Apache Stats-----${NC}";echo "";echo -e "${BOLD}${LBLU}Apache Version:${NC}";httpd -V|grep version|awk '{print $3,$4}';echo "";echo -e "${BOLD}${LBLU}Apache MPM:${NC}";httpd -V|grep MPM|awk '{print $NF}';echo "";echo -e "${BOLD}${LBLU}Checking for Apache confs:${NC}";if [[ -s /usr/local/apache/conf/includes/pre_virtualhost_global.conf ]];then echo -e "${BOLD}${LBLU}Contents of /usr/local/apache/conf/includes/pre_virtualhost_global.conf:${NC}";cat /usr/local/apache/conf/includes/pre_virtualhost_global.conf;else echo -e "${BOLD}${LBLU}No global Apache configurations in place.${NC}";fi;echo "";if [[ -s /usr/local/apache/conf/includes/pre_virtualhost_2.conf ]];then echo -e "${BOLD}${LBLU}Contents of /usr/local/apache/conf/includes/pre_virtualhost_2.conf:${NC}";cat /usr/local/apache/conf/includes/pre_virtualhost_2.conf;else echo -e "${BOLD}${LBLU}No Apache 2 configurations in place.${NC}";fi;echo "";ea_check () { if [[ $(httpd -V|egrep "HTTPD_ROOT=\"\/etc\/apache2") ]];then return;fi; } ;if ea_check $1;then if [[ $(grep max /opt/cpanel/ea-php*/root/usr/var/log/php-fpm/error.log) ]] ;then echo -e "${BOLD}${LBLU}Potential MaxChildren limit reached errors:${NC}" ; grep max /opt/cpanel/ea-php*/root/usr/var/log/php-fpm/error.log | wc -l ;echo -e "${BOLD}${LBLU}Top 5 offending domains:${NC}"; grep max /opt/cpanel/ea-php*/root/usr/var/log/php-fpm/error.log|awk '{print $5}'|sort|uniq -c|sort -rnk1|head -n5 ;else echo -e "${BOLD}${LBLU}No PHP-FPM limit errors"${NC} ;fi ;if [[ $(grep MaxRequestWorkers /usr/local/apache/logs/error_log) ]] ;then echo -ne "${BOLD}${LBLU}Potential MaxRequestWorkers limit errors: "${NC} ; grep MaxRequestWorkers /usr/local/apache/logs/error_log| wc -l ;else echo -e "${BOLD}${LBLU}No Apache limit errors.${NC}" ;fi;fi;echo "";echo -e "${BOLD}-----PHP Stats-----${NC}";echo "";if [[ $(httpd -V|egrep "HTTPD_ROOT=\"\/etc\/apache2") ]]; then echo -e "${BOLD}${LBLU}Inherent PHP version:${NC}";php -v|head -n1|awk '{print$1" "$2}'; echo -e "${BOLD}${LBLU}PHP versions in use on `hostname`:${NC}" ;grep "phpversion:" /var/cpanel/userdata/*/*|awk '{print $2}'|sort|uniq -c|sort -rnk1;echo "";if [[ $(yum list installed|grep "memcache\|opcache") ]]; then echo -e "${BOLD}${LBLU}PHP caching:${NC}";yum list installed|grep "memcache\|opcache"|awk '{print $1}'; else echo -e "${BOLD}${LBLU}No PHP caching${NC}";fi;else echo -e "${BOLD}${LBLU}Inherent PHP version:${NC}:"; php -v|head -n1|awk '{print$1" "$2}';if [[ $(php -m|grep cache) ]]; then echo -e "${BOLD}${LBLU}PHP caching:${NC}";php -m|grep cache; else echo -e "${BOLD}${LBLU}No PHP caching${NC}";fi;fi; echo ""; echo -e "${BOLD}${LBLU}Checking various memory_limit settings:"${NC}; echo -en "Current loaded php.ini file: ";echo -e ${LBLU}${BOLD}$(php -i|grep "Loaded Configuration File"|awk '{print$5}'|head -1);echo -e $(cat `php -i|grep "Loaded Configuration File"|awk '{print$5}'|head -1`| grep memory_limit)${NC};echo "";echo -e "${BOLD}-----cPanel Stats-----${NC}";echo "";echo -ne "${BOLD}${LBLU}cPanel: ${NC}";cat /usr/local/cpanel/version ;echo -ne "${BOLD}${LBLU}cPanel accounts: ${NC}";/scripts/updateuserdomains && cat /etc/trueuserdomains|wc -l ;echo -ne "${BOLD}${LBLU}cPanel domains: ${NC}";awk '/DNS*=/' /var/cpanel/users/*|grep -vE '(^| )DNS=( |$)'|cut -c 5-|wc -l ;echo "" ;if [[ $(httpd -V|egrep "HTTPD_ROOT=\"\/etc\/apache2") ]]; then echo -e "${BOLD} ${BLUB}--EasyApache 4 in use--${NC}"; echo $EA4H; else echo -e "${BOLD} ${BLUB}--EasyApache 3 in use--${NC}"; echo $EA3H; fi;echo -ne "${BOLD}${LBLU}Checking cPanel backups:${NC}";echo "";grep --color=never -i "enable\|dir\|retent" /var/cpanel/backups/config;echo "";echo -e "${BOLD}-----MySQL Stats-----${NC}";echo "";echo -e "${BOLD}${LBLU}MySQL Version:${NC}";mysql --version;echo "";echo -e "${BOLD}${LBLU}MySQL data usage:${NC}";mysql -Bse 'show variables like "datadir";'|awk '{print $2}'|xargs -I{} find {} -type f -printf "%s %f\n"|awk -F'[ ,.]' '{print $1, $NF}'|awk '{array[$2]+=$1} END {for (i in array) {printf("%-15s %s\n", sprintf("%.3f MB", array[i]/1048576), i)}}' | egrep --color=none '(MYI|ibd)'; echo ""; echo -e "${BOLD}${LBLU}Try to set buffers to the following:${NC}";mysql -Bse 'show variables like "datadir";'|awk '{print $2}'|xargs -I{} find {} -type f -printf "%s %f\n"|awk -F'[ ,.]' '{print $1, $NF}'|awk '{array[$2]+=$1} END {for (i in array) {printf("%-15s %s\n", sprintf("%.3f MB", array[i]/1048576), i)}}' | awk '{if($3~/MYI/)print"key_buffer_size\t\t",$1"M"};{if($3~/ibd/)a+=$1}END{print "innodb_buffer_pool_size\t",a"M"}';echo "";echo -e "${BOLD}${LBLU}Important MySQL configurations:${NC}";grep --color=never "max_connections\|key_buffer\|max_allowed_packet\|innodb_buffer_pool_size" /etc/my.cnf;mysql -e "SHOW GLOBAL STATUS;"|grep --color=none Max_used_;echo "";echo -e "${BOLD}${LBLU}Current MySQL queries:${NC}";mysqladmin proc stat;echo "")
```

Which should output something similar to the following:

```
-----General Stats-----

Hostname: 
host2.tonallytubular.com

Uptime:
 07:06:54 up 40 days,  4:52,  1 user,  load average: 0.05, 0.03, 0.05

Operating System:
CentOS Linux release 7.4.1708 (Core) 

Disk Usage:
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda3        48G   13G   33G  29% /
devtmpfs        437M     0  437M   0% /dev
tmpfs           446M     0  446M   0% /dev/shm
tmpfs           446M   51M  395M  12% /run
tmpfs           446M     0  446M   0% /sys/fs/cgroup
/dev/loop0      3.9G   22M  3.6G   1% /tmp
tmpfs            90M     0   90M   0% /run/user/0

Memory Usage:
              total        used        free      shared  buff/cache   available
Mem:           891M        415M         91M         44M        384M        261M
Swap:          2.0G        263M        1.7G

Potential OOM notifications: 0
CPU(s): 2

-----Apache Stats-----

Apache Version:
Apache/2.4.33 (cPanel)

Apache MPM:
worker

Checking for Apache confs:
Contents of /usr/local/apache/conf/includes/pre_virtualhost_global.conf:
KeepAlive On
MaxKeepAliveRequests 150
KeepAliveTimeout 2
Timeout 30

<IfModule prefork.c>
    MaxClients 30
</IfModule>
<IfModule worker.c>
    ServerLimit 16
    StartServers 3
    ThreadsPerChild 25
    MaxClients 400
</IfModule>
<IfModule event.c>
    ServerLimit 16
    StartServers 3
    ThreadsPerChild 25
    MaxClients 400
</IfModule> 

No Apache 2 configurations in place.

No PHP-FPM limit errors
Potential MaxRequestWorkers limit errors: 1

-----PHP Stats-----

Inherent PHP version:
PHP 7.1.22
PHP versions in use on host2.tonallytubular.com:
      4 ea-php70
      3 ea-php71

PHP caching:
ea-php70-php-opcache.x86_64
ea-php71-php-opcache.x86_64
memcached.x86_64

Checking various memory_limit settings:
Current loaded php.ini file: /opt/cpanel/ea-php71/root/etc/php.ini
memory_limit = 64M

-----cPanel Stats-----

cPanel: 11.76.0.12
cPanel accounts: 7
cPanel domains: 7

 --EasyApache 4 in use--

Checking cPanel backups:
BACKUPDIR: /backup
BACKUPENABLE: yes
BACKUP_DAILY_ENABLE: yes
BACKUP_DAILY_RETENTION: 2
BACKUP_MONTHLY_ENABLE: no
BACKUP_MONTHLY_RETENTION: 1
BACKUP_WEEKLY_ENABLE: no
BACKUP_WEEKLY_RETENTION: 4

-----MySQL Stats-----

MySQL Version:
mysql  Ver 14.14 Distrib 5.6.41, for Linux (x86_64) using  EditLine wrapper

MySQL data usage:
0.217 MB        MYI
194.062 MB      ibd
76.000 MB       ibdata1

Try to set buffers to the following:
key_buffer_size		 0.217M
innodb_buffer_pool_size	 270.062M

Important MySQL configurations:
innodb_buffer_pool_size=64M
max_connections=100
key_buffer_size = 64M
max_allowed_packet=268435456
Max_used_connections	16

Current MySQL queries:
+-------+------+-----------+----+---------+------+-------+------------------+
| Id    | User | Host      | db | Command | Time | State | Info             |
+-------+------+-----------+----+---------+------+-------+------------------+
| 72172 | root | localhost |    | Query   | 0    | init  | show processlist |
+-------+------+-----------+----+---------+------+-------+------------------+
Uptime: 1848742  Threads: 1  Questions: 619645  Slow queries: 0  Opens: 1339  Flush tables: 1  Open tables: 1332  Queries per second avg: 0.335

```

If you wanted to run an individual block of those 'stats', those can be found broken up and named by what they check.

### What to do with this information:

Well, that's up to you. The whole point of this script is to provide a brief overview of things to know where to check from there. For example, if you see signs of OOM, you can then look through /var/log/messages for the last OOM meesage and troubleshoot from there. If you see Apache or PHP-FPM limits being reached, you can check those logs out and see if they can or need to be increased (or decreased). If there's no Apache configurations in place, you can easily add some by following our wiki:

https://wiki.int.liquidweb.com/articles/Apache_Settings

#### Notes on the MySQL checks

Please do not just blindly adjust the MySQL buffers to match what the script recommends. This is just a reference for what would be ideal so use that to use your best judgment, hit up a senior techncian if you are unsure, or check out our wiki on it:

https://wiki.int.liquidweb.com/articles/MySQL_Performance

## Authors & Acknowledgments

- Shout out to the people who's code got scraped from our wiki's for this (especially the MySQL wiki's).
- Ndilernia for providing and suggesting a kernel checker.
- Conofrio for getting the buffers to add up for us.


### Known & Reported bugs:

- [ ] OOM Kiler false positives cause by FTP users deleting files
- [ ] greps should be updated to grep -a to avoid occasional binary errors
- [ ] EA3 - Apache checks throws error about "/opt/cpanel/ea-php*/root/usr/var/log/php-fpm/error.log: No such file or directory" becasue that doesn't exist on EA3.
- [ ] PHP-FPM MaxChildren checker is supposed to pring the top 5 results and is piped to head at the end but it doesn't appear to work properly. Not sure what is causing that.

### Features & Suggestions

Nothing further planned really beyond trimming the fat. If you do have any suggestinos or tools you want to contribute, let me know!

- [ ] Trim the fat

