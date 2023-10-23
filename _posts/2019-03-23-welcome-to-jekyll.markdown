---
layout: post
title:  "Notes of User-Center"
date:   2023-10-22 13:06:00 +8
categories: Java Yml
---
# 用户中心笔记

### 1.初始化

spring initializer常用依赖：

lombok,

springboot devtools,

spring configuration processor,

 mysql driver,

 spring web,

mybatis framework,

junit，

Spring data redis

### 2.新建并测试数据库

```
mysql -uroot -p
create database yupi;
use yupi;
```

新建测试表：

```sql
DROP TABLE IF EXISTS user;

CREATE TABLE user
(
    id BIGINT(20) NOT NULL COMMENT '主键ID',
    name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
    age INT(11) NULL DEFAULT NULL COMMENT '年龄',
    email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
    PRIMARY KEY (id)
);
```

```sql
DELETE FROM user;

INSERT INTO user (id, name, age, email) VALUES
(1, 'Jone', 18, 'test1@baomidou.com'),
(2, 'Jack', 20, 'test2@baomidou.com'),
(3, 'Tom', 28, 'test3@baomidou.com'),
(4, 'Sandy', 21, 'test4@baomidou.com'),
(5, 'Billie', 24, 'test5@baomidou.com');
```

### 3.整合mp并测试

##### (1)引入baomidou依赖

```java
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.2</version>
</dependency>

```

##### (2)进入application.propoties,改成yml,并配置数据库

```yml
spring:
  application:
      name: user-center
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/yupi
    username: root
    password: uii084929
server:
  port: 8080
```

##### (3)在 Spring Boot 启动类中添加 `@MapperScan` 注解，扫描 Mapper 文件夹：

```java
@SpringBootApplication
@MapperScan("com.example.tlm.mapper")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

##### (4)在com.example.yupi下创建mapper,model文件夹，并在mapper中编写一个接口：

```java
public interface UserMapper extends BaseMapper<User> {
}
```

在model文件夹下新建一个实体类：

```java
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

##### (5)创建测试类

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class SampleTest {

    @Resource
    private UserMapper userMapper;

    @Test
    public void testSelect() {
        System.out.println(("----- selectAll method test ------"));
        List<User> userList = userMapper.selectList(null);
        Assert.assertEquals(5, userList.size());
        userList.forEach(System.out::println);
    }
}
```

或者在同名测试类中加入一下代码：

```java
    @Resource
    private UserMapper userMapper;

    @Test
    public void testSelect() {
        System.out.println(("----- selectAll method test ------"));
        List<User> userList = userMapper.selectList(null);
        Assert.assertEquals(5, userList.size());
        userList.forEach(System.out::println);
    }
```

### 4.引入redis

##### (1)引入commons依赖：

```java
<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-pool2 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    <version>2.11.1</version>
</dependency>
```

##### (2)在application.yml配置地址信息:

```yml
spring:
  redis:
    host: 127.0.0.1
    port: 6379
    password: uii084929
    lettuce:
      pool:
        max-active: 8
        max-idle: 8
        min-idle: 1
        max-wait: 10s
         
```

##### (3)在测试类编写测试代码：

```java
@Autowired
private StringRedisTemplate stringRedisTemplate;
@Test
void testStringTemplate(){
  stringRedisTemplate.opsForValue().set("name","hui");
  String name = stringRedisTemplate.opsForValue().get("name");
  System.out.println("name:" + name);
}
```

### 5.设计数据库

设计用户表yupi_user:

id(主键) bigint

userAccount(账号) varchar(256)

userPassword(密码) varchar(256)

avatarUrl(头像) varchar(1024)

gender(性别) tinyint

userStatus(权限) int default 0

createTime(创建时间) datatime

isDelete(是否删除) tinyint default 0

### 6.新建文件夹



<img src="/Users/wuhuihui/Desktop/ssm/1.png" alt="1" style="zoom:50%;" />

### 7.mybatisX generator生成相关实体类，xml以及接口

### 8.修改mybatis plus配置

```yml
mybatis-plus:
  configuration:
    map-underscore-to-camel-case: false
```



### 9.写逻辑

##### 1工具类：

uuid生成token

```java
String token = UUID.randomUUID().toString();
```



##### 2.用户登录逻辑

校验账号密码输入是否合理

查询账号是否在数据库中存在

查询密码并脱敏

判断是否与数据库中的密码是否相同

相同就生成token，存redis

##### 3.拦截器设置

新建interceptor文件夹，新建loginInterceptor类实现HandlerInterceptor接口，加上component注解，重写方法，true是放行

新建config文件夹，新建mvcConfig继承WebMvcConfig类，并添加Configuration注解，重写addInterceptor方法

### 10.多环境

#### 前端多环境

- 请求地址
  - 开发环境：localhost:8080
  - 线上环境：user-backend.code-nav.cn

#### 后端多环境

- 新建application-prod.yml，原application.yml作为公共配置

- 在application-prod里添加线上数据库地址

- 创建sql文件记录要用的sql

- 执行maven里的package插件，如果报错点击maven的小闪电跳过测试

- 打包后的jar包可以用命令启动并且传入环境变量

  ```bash
  java -jar jar包位置 --spring.profiles.active=prod
  ```

- 主要是改依赖的环境地址：
  - 数据库地址
  - 缓存地址
  - 消息队列地址
  - 项目端口号
  - 服务器配置

### 10.5.Docker

下载镜像

```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

官方镜像网站 https://hub.docker.com/

例如

```
docker pull mysql:latest
```

给mysql镜像设置登录账号密码以及启动，并映射到外网

```
docker run --name mysql -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -d mysql:版本号
```

让mysql所有人可访问

```bash
use mysql;
update user set user.Host=’%’ where user.User=‘root’;
flush privileges;
ALTER USER ‘test’@’%’ IDENTIFIED WITH mysql_native_password BY ‘12345’;
#vi /etc/my.cnf
加入下面内容：default_authentication_plugin=mysql_native_password
```

查看镜像

```
docker images
```

查看运行中镜像(实例)，-a是显示隐藏

```
docker ps -a
```

删除镜像

```
docker rmi 镜像id
```

删除实例

```bash
docker rm 实例id
```

docker运行redis

创建挂载目录

```
mkdir -p /usr/local/docker/data
mkdir -p /usr/local/docker/conf
```

添加redis.conf配置

在刚刚创建的/usr/local/docker/conf目录下新建一个redis.conf文件，内容如下：

```
#bind 127.0.0.1    #注释掉后或者改成0.0.0.0表示允许远程连接
protected-mode no
appendonly yes #持久化
requirepass 123456 #密码
```

创建redis容器(实例)并启动

- **–appendonly yes** 打开redis持久化

```
docker run -p 6379:6379 --name redis -v /usr/local/docker/redis.conf:/etc/redis/redis.conf -v /usr/local/docker/data:/data -d redis:6.2.11 redis-server /etc/redis/redis.conf --appendonly yes
```

docker exec -it 从宿主机，进入容器内部 :

```bash
docker ps  #查看正在运行的容器
docker exec -it mynginx /bin/bash 
docker exec -it mynginx /bin/sh /root/runoob.sh  #进入容器的同时，运行runoob.sh脚本
exit #退出容器，退出后不影响该容器的正常运行（如果是通过dock run直接进入的容器，会影响）
```

dockerfile

```bash
FROM java:8-alpine
WORKDIR /app
COPY ./tlm-0.0.1-SNAPSHOT.jar .
EXPOSE 8888
CMD ["java","-jar","/app/tlm-0.0.1-SNAPSHOT.jar","--spring.profiles.active=prod"]
```



### 11.部署

##### (1)原始部署:

需要linux服务器（Centos8+)

前端:

  需要web服务器：nginx,apache,tomcat

  安装nginx服务器：

​     1.centos自带yum包管理器

​     2.去官网用指令下载

```bash
curl -o 文件名 连接
```

​     解压

```bash
tar -zxvf 文件名
```

   ...

后端：java, maven

```bash
yum install -y java-1.8.0-openjdk*
```



```bash
curl -o maven 链接
git clone xxx
mvn package -DskipTests
chmod a+x 文件名                             添加可执行权限
nohup java -jar jar包位置 --spring.profiles.active=prod &
```

##### (2)宝塔linux部署

Linux运维面板

方便管理服务器，方便安装软件

需要宝塔Linux系统

下载nginx,tomcat(自动下java)

新建文件夹-->把前端文件粘到下面-->在php下添加站点

新建文件夹-->把后端编译的jar包粘进去-->新建java项目,执行命令:

```bash
/usr/bin/java -jar -Xmx1024M -Xms256M jar包位置 --server.adress=0.0.0.0 --spring.profiles.active=prod
```

（把tomcat停止）-->云服务器和宝塔打开8080端口

##### (3)docker部署

宝塔安装docker

Dockerfile用于指定构建镜像的方法，去GitHub,gitee上找别人写好的就行，在根目录新建文件Dockerfile，粘贴

将后端代码粘到linux

在Linux上用docker根据Dockerfile建立镜像

```
sudo docker build -t user-center-backend:v0.0.1 .
```

前端一样

Docker构建优化：减少体积，缩短构建时间(比如多阶段构建)

```bash
sudo docker run -p 8081:8081 -d user-center-backend:v0.0.1
```

进入容器

```bash
docker exec -it 容器id bash
```

docker命令参考https://www.runoob.com/docker/docker-command-manual.html

常用：

```
docker ps      docker images     docker kill
```

##### (4)docker平台部署

1.云服务商的容器平台(腾讯云，阿里云)

2.面向某个领域的容器平台(后端微信云托管)

容器平台(收费)好处：

  1.不用输命令来操作，更方便省事

  2.不用在控制台操作，更傻瓜式，更简单

  3.大厂运维，省心

  4.有其他功能（监控，告警等）

直接把后端代码带上Dockerfile上传即可

##### (5)前端webify部署

更傻瓜式，代码更新时自动构建，但代码需要在代码托管平台上

### 12.绑定域名

在云服务商搜域名解析

前端项目访问流程：用户输入网址==>域名解析服务器(把网址解析为ip地址/交给其他的域名解析服务)

==>nginx接受请求，找到对应文件，返回文件给前端==>前端加载文件(css,js)到浏览器中==>渲染

后端项目访问流程：用户输入网址==>域名解析服务器==>服务器==>nginx接受请求==>后端项目

### 13.跨域问题

浏览器为了用户的安全，仅允许向**同域名，同端口**的服务器发送请求

解决方案：

  1.把前后端域名，端口改成相同的

让服务器告诉浏览器，允许跨域(返回cross-origin-allow响应头)

  2.网关支持(Nginx)：

  3.修改后端服务

​    1.配置@CrossOrigin注解

​    2.添加web全局请求拦截器

### 14.项目优化

1.功能扩充

2.项目登陆改为单点登录

