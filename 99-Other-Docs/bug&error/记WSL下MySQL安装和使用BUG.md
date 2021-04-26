这里完全按照[WSL官网](https://docs.microsoft.com/en-us/windows/wsl/tutorials/wsl-database)安装MySQL

------

To install MySQL on WSL (Ubuntu 18.04):

1. Open your WSL terminal (ie. Ubuntu 18.04).
2. Update your Ubuntu packages: `sudo apt update`
3. Once the packages have updated, install MySQL with: `sudo apt install mysql-server`
4. Confirm installation and get the version number: `mysql --version`

You may also want to run the included security script. This changes some of the less secure default options for things like remote root logins and sample users. To run the security script:

1. Start a MySQL server: `sudo /etc/init.d/mysql start`
2. Start the security script prompts: `sudo mysql_secure_installation`
3. The first prompt will ask whether you’d like to set up the Validate Password Plugin, which can be used to test the strength of your MySQL password. You will then set a password for the MySQL root user, decide whether or not to remove anonymous users, decide whether to allow the root user to login both locally and remotely, decide whether to remove the test database, and, lastly, decide whether to reload the privilege tables immediately.

To open the MySQL prompt, enter: `sudo mysql`

To see what databases you have available, in the MySQL prompt, enter: `SHOW DATABASES;`

To create a new database, enter: `CREATE DATABASE database_name;`

To delete a database, enter: `DROP DATABASE database_name;`

For more about working with MySQL databases, see the [MySQL docs](https://dev.mysql.com/doc/mysql-getting-started/en/).

To work with with MySQL databases in VS Code, try the [MySQL extension](https://marketplace.visualstudio.com/items?itemName=cweijan.vscode-mysql-client2).

------

## 安装Bug： cannot create directory '//.cache': Permission denied

然而，在执行`sudo mysql`这一步时，出现下面错误：

```java
 * Starting MySQL database server mysqld                                                                                No directory, logging in with HOME=/
-su: 2: /etc/profile.d/wsl-integration.sh: [[: not found
mkdir: cannot create directory '//.cache': Permission denied
-su: 27: /etc/profile.d/wsl-integration.sh: cannot create //.cache/wslu/integration: Directory nonexistent
```

这里我搜索到wsl官方issue[Cannot create directory "//.cache": Permission denied](https://github.com/wslutilities/wslu/issues/101)，按照里面大牛建议修改wsl-integration.sh文件

`sudo vim /etc/profile.d/wsl-integration.sh`：

在`WSL_INTEGRATION_CACHE=$HOME/.cache/wslu/integration`前加入这么一段：

```shell
# Check if we have HOME folder
if [ "${HOME}" = "/" ]; then
  return
fi
```

然而并未解决问题，`sudo service mysql start`启动时，会出现以下的错误：

```java
 * Starting MySQL database server mysqld                                                                                No directory, logging in with HOME=/
                                                                                                                 [fail]
```

按照建议，依次执行命令：

```shell
service mysql stop
usermod -d /var/lib/mysql/ mysql
```

这时报出：

```shell
usermod: user mysql is currently used by process 6293
```

那么接下来 kill process即可：

```shell
 sudo mysql
 
 nohup kill 6293; sleep 2; usermod -d /home/mysql mysql &
```

然后再：

```shell
 sudo service mysql start
```

问题解决！！！

## 密码设置错误：Your password does not satisfy the current policy requirements

查看MySQL账号信息：

```shell
mysql> SELECT user,authentication_string,plugin,host FROM mysql.user;
+------------------+-------------------------------------------+-----------------------+-----------+
| user             | authentication_string                     | plugin                | host      |
+------------------+-------------------------------------------+-----------------------+-----------+
| root             |                                           | mysql_native_password | localhost |
| mysql.session    | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
| mysql.sys        | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
| debian-sys-maint | *84BC3A2C2D276D9B3CA185467416ECC3DD25121E | mysql_native_password | localhost |
+------------------+-------------------------------------------+-----------------------+-----------+
4 rows in set (0.00 sec)
```

添加账户：

```shell
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'xxxxxx';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

查看密码设置相关要求：

```shell
mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_check_user_name    | OFF    |
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.01 sec)
```

这里length至少8且policy是MEDIUM，要求远远高于随便设置的几个数字密码：

```shell
mysql> set global validate_password_policy = 0;
Query OK, 0 rows affected (0.00 sec)

mysql> set global  validate_password_length = 6;
Query OK, 0 rows affected (0.00 sec)
```

退出后在执行：`mysql_secure_installation`设置密码或者继续：

```shell
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'xxxxxx';
Query OK, 0 rows affected (0.01 sec)
```

问题解决！！！