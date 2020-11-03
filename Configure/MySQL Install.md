# MySQL Install in Ubuntu

## MySQL Install
```  
$ sudo apt-get install mysql-server
```

## Start And Enable Service
```
$ sudo systemctl start mysql
$ sudo systemctl enable mysql
```

## Connect MySQl
```
$ sudo /usr/bin/mysql -u root -p
```

## Create Database 
```
> CREATE DATABASE {database_name}
> show databases
+--------------------+
| Database           |
+--------------------+
| {database_name}    |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
``` 

## Create User And Grant Privileges  

```
> CREATE USER '{user_name}'@'localhost' IDENTIFIED BY '{password}'
> SELECT User, Host, authentication_string FROM mysql.user;
+------------------+-----------+------------------------------------------------------------------------+
| User             | Host      | authentication_string                                                  |
+------------------+-----------+------------------------------------------------------------------------+
| debian-sys-maint | localhost | $A$005$O?W4<#&G2jw5IKbvL/zdxfFywthX6dJveRuiXlH0yXM51/JqR18DZmde2       |
| {user_name}      | localhost | $A$005$0^C3Gs-1{FSnueCHuq.UlR2014J0vk0RhIPWw2Od9YM3zVRM2dB3XZJIPlu/    |
| mysql.infoschema | localhost | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| mysql.session    | localhost | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| mysql.sys        | localhost | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| root             | localhost |                                                                        |
+------------------+-----------+------------------------------------------------------------------------+

# 전체 DB에 대한 권한  
> GRANT ALL PRIVILEGES ON * to {user_name}@localhost;
# 특정 DB에 대한 권한
> GRANT ALL PRIVILEGES ON {database_name}.* to {user_name}@localhost;
> FLUSH PRIVILEGES
> SHOW GRANTS FOR '{user_name}'@'localhost'
+--------------------------------------------------------------------------+
| Grants for {user_name}@localhost                                         |
+--------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `{user_name}`@`localhost`                          |
| GRANT ALL PRIVILEGES ON `{database_name}`.* TO `{user_name}`@`localhost` |
+--------------------------------------------------------------------------+
```


