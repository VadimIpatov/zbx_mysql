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
UserParameter=mysql.is_alive,/usr/bin/mysqladmin ping | grep -c alive
```

* It's recommended to disable SELinux (or configure it)
* Import "templates_db_mysql.xml" into zabbix as template
* Create host objects and link with this template
* Provide database user credentials by using host-level macroses: {$MYSQL_USER} and {$MYSQL_PASSWORD}. Default: zabbix with empty password.
* You can also configure MySQL port ({$MYSQL_PORT}, default: 3306) and connection timeout ({$MYSQL_CONN_TIMEOUT}, default: 0) if you want.

