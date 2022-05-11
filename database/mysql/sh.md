## 1，查看mysql服务

### 方式一

```sh
ps -ef|grep mysql
```

### 方式二

```sh
netstat -nlp
```

## 2，启动mysql服务

### 命令行方式

```sh
.mysql/bin/mysqld_safe &

# 自定义 my.cnf 文件
./mysqld_safe --defaults-file=/xxx/my.cnf &
```

### 服务方式

```sh
service mysql start
# 如果服务在启动状态，直接重启服务用以下命令
service mysql restart
```

## 3，关闭mysql服务

### 命令行方式

```sh
./mysqladmin -u root shutdown

# 或者
./mysqladmin -uroot -p -P3306 -h127.0.0.1 shutdown
```

### 服务方式

```sh
service mysql stop
```

### MySQL 命令

```sql
sql>shutdown;
sql>exit;
```

