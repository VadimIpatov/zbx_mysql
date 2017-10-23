# Zabbix MySQL Monitoring Templates
A Zabbix templates for MySQL monitoring.

Consists of common metrics, innodb metrics and replication metrics (master-slave, master-master).

Tested on MySQL 5.6 and MariaDB 10.1.

Author: Vadim Ipatov <<vadim.ipatov@zabbix.com>> (<euphoria.vi@gmail.com>)

## Requires
* Zabbix >=3.4 (because the template uses [dependent items](https://www.zabbix.com/documentation/3.4/manual/config/items/itemtypes/dependent_items) and [value preprocessing](https://www.zabbix.com/documentation/3.4/manual/config/items/item#item_value_preprocessing) features that were introduced in 3.4)

## Installation
You need to configure every MySQL nodes as shown below:

* Create a database user:

```
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY '<PASSWORD>';
```

* Grant its user ONLY the following privileges:

```
GRANT REPLICATION CLIENT ON *.* TO 'zabbix'@'localhost' WITH MAX_USER_CONNECTIONS 5;
GRANT PROCESS ON *.* TO 'zabbix'@'localhost';
GRANT SELECT ON performance_schema.* TO 'zabbix'@'localhost';
```

* Copy "configs/mysql.conf" into your zabbix_agent include folder (default: /etc/zabbix/zabbix_agentd.d/) or manually add these parameters to config:

```
UnsafeUserParameters=1
UserParameter=mysql.query[*],MYSQL_PWD=$3 /usr/bin/mysql -u$2 -h 127.0.0.1 -P$4 --connect-timeout=$5 --xml -e '$1'
UserParameter=mysql.discovery.schema[*],OUTPUT=`MYSQL_PWD=$2 /usr/bin/mysql -u$1 -P$3 --connect-timeout=$4 -e 'SELECT TABLE_SCHEMA,ENGINE FROM INFORMATION_SCHEMA.TABLES GROUP BY TABLE_SCHEMA' | tail -n +2 | awk '{print "{\"{#SCHEMA}\":\""$$1 "\",\"{#TYPE}\":\""$$2"\"},"}'` && echo "{\"data\":[${OUTPUT::-1}]}"
UserParameter=mysql.is_alive,/usr/bin/mysqladmin ping | grep -c alive
```

* It's recommended to disable SELinux (or configure it)
* Import "templates_db_mysql.xml" into zabbix as template
* Create host objects and link with this template.
