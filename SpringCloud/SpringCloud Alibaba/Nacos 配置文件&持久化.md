

# Nacos 持久化

**1，持久化配置**

```properties
#*************** Spring Boot Related Configurations ***************#
### Default web context path:
server.servlet.contextPath=/nacos
### Default web server port:
server.port=8848


spring.datasource.platform=mysql
### Count of DB:
db.num=1
### Nacos 持久化:
db.url.0=jdbc:mysql://MySQL地址:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=用户名
db.password.0=密码

### Connection pool configuration: hikariCP
db.pool.config.connectionTimeout=30000
db.pool.config.validationTimeout=10000
db.pool.config.maximumPoolSize=20
db.pool.config.minimumIdle=2

### 开启权限控制:
nacos.core.auth.enabled=true

```

**2，创建nacos 持久化数据库**

> 该sql放置在conf目录下。同时插入默认的用户密码不然无法登录

