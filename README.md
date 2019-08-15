# Fame博客系统文档

## 项目结构

### 文件结构

```
└─Fame
    ├─fame-admin          // 博客管理后台，基于Vue+elementui
    ├─fame-docker         // docker部署文件
    ├─fame-front          // 博客前端，基于Nuxt
    ├─fame-server         // 博客服务端，基于spring-boot
    ├─docker-compose.yml  // docker-compose文件
```

### 部署结构(docker)

![fame-structure](https://raw.githubusercontent.com/zzzzbw/blog_source/master/images/FameDocker/fame-structure.jpg)

## 部署方式

> 博客管理后台初始的账号：fame，密码：123456，登录管理后台后可修改。

### Docker方式部署(推荐)

首先保证有Docker和Docker compose的环境，如果没有可参考[这里](https://github.com/zzzzbw/Fame/wiki/Docker%E5%92%8CDocker-compose%E5%AE%89%E8%A3%85)

1. 克隆项目到本地

   ```
   git clone https://github.com/zzzzbw/Fame.git
   ```

3. 启动项目

    ```
    docker-compose up 或 docker-compose up -d
    ```
    第一次启动推荐`docker-compose up`，可以看到启动日志，由于要下载镜像和maven依赖，时间可能较久，视网络环境和性能而定

    ```
    [root@localhost Fame]# docker-compose up -d
    Starting fame-front ... 
    Starting fame-admin ... 
    Starting fame-front ... done
    Starting fame-admin ... done
    Starting fame-nginx ... done
    ```
    
4. 访问地址
  
    启动完成后，在浏览器访问
    
    `http://xx.xxx.xx.xx/` 为博客前端首页
    
    `http://xx.xxx.xx.xx/admin` 为博客管理后台首页
    
    > xx.xxx.xx.xx为你的域名或者ip地址，本地的话即为127.0.0.1

### 开发环境快速启动

首先保证有 `java8` `mysql5.7.x` `maven3.3.x` `node10.x` `npm6.x`的环境(版本不一定要完全一样，但避免奇怪的问题出现，最好相同)

1. 克隆项目到本地

   ```
   git clone https://github.com/zzzzbw/Fame.git
   ```

2. 部署服务端 (项目使用lombok插件，如果要在ide中调试要有lombok插件)

    2.1 进入服务端文件夹

        `cd fame-server`

    2.2 修改spring-boot配置文件

      `vi src/main/resources/application-dev.properties`

      ```
      spring:
        datasource:
          driverClassName: com.mysql.cj.jdbc.Driver
          url: jdbc:mysql://localhost:3306/fame?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai
          username: root
          password: root
      ```
      将配置中的`username`和`password`修改成对应你数据库的用户名密码

    > 你的`Mysql`中，要先有`fame`数据库。
    >
    > ```sql
    > CREATE DATABASE fame CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
    > ```

    2.3 启动fame-server

      `mvn clean spring-boot:run -Dmaven.test.skip=true`

3. 部署博客前端

    3.1 进入前端文件夹

      `cd fame-front`

    3.2 安装依赖和启动服务

      ```
    npm install
    npm run dev
      ```

    3.3 访问地址

      启动完成后，浏览器访问 `http://localhost:3000`

4. 部署博客后端

    4.1 进入后端文件夹

      `cd fame-admin`

    4.2 安装依赖和启动服务

     ```
    npm install
    npm run serve
     ```

    4.3 访问地址

      启动完成后，浏览器访问 `http://localhost:8010/admin`

## 开发文档

###  fame-server

fame-server是Fame博客系统的服务端，开发语言为Java，使用技术栈为

* SpringBoot
* SpringDataJpa
* flexmark(Markdown解析)
* Mysql

#### 接口文档

该服务为Restful接口，可以参考[接口文档](https://zzzzbw.github.io/Fame/swagger/)

### fame-front

fame-front是Fame博客的前端部分，由于前端博客有搜索引擎的SEO需求，而Vue等单页应用框架对搜索引擎收录不大友好，所以使用了前端框架Nuxt。Nuxt是SSR版的Vue，可以参考[官方](https://zh.nuxtjs.org/)。

#### sitemap

支持sitemap功能

启动服务后访问`http://xx.xxx.xx.xx/sitemap.xml`或`http://localhost:3000/sitemap.xml`（开发模式启动）

#### rss

支持rss功能

启动服务后访问`http://xx.xxx.xx.xx/feed.xml`或`http://localhost:3000/feed.xml`（开发模式启动）

### fame-admin

fame-admin是Fame博客的后台管理界面部分，使用了Vue+Elementui框架。

一些关于网站的系统设置如账号密码等可以在管理后台的`网站设置`中修改。

### fame-docker

fame-docker是关于docker部署的各个系统的`Dockerfile`和一些配置文件。

文件结构:

```
└─fame-docker
    ├─fame-admin          
      ├─fame-admin-Dockerfile       // fame-admin服务Dockerfile文件
      └─nginx.conf                  // fame-admin服务nginx配置文件
    ├─fame-mysql
      ├─fame-mysql-Dockerfile       // fame-mysql服务Dockerfile文件
      └─mysqld.cnf                  // fame-mysql数据库配置文件
    ├─fame-nginx
      ├─nginx-Dockerfile            // nginx服务Dockerfile文件
      └─nginx.conf                  // nginx服务nginx配置文件
    ├─fame-front-Dockerfile         // fame-front服务Dockerfile文件
    ├─fame-server-Dockerfile        // fame-server服务Dockerfile文件
```

> `fame-server`和`fame-front`项目都自带了服务器，所以直接用docker直接启动项目即可。
> `fame-admin`项目是静态页面，需要搭配服务器使用，所以这个docker项目增加了一个nginx服务器。

## 备注

### 常用脚本

#### 进入各个容器(服务启动时)

fame-server:

```
docker exec -it fame-server bash
```

fame-mysql:

```
docker exec -it fame-mysql bash
```

fame-front:

```
docker exec -it fame-front sh
```

fame-admin:

```
docker exec -it fame-admin sh
```

fame-nginx:

```
docker exec -it fame-nginx sh
```

#### 备份mysql容器的sql数据

```
docker exec fame-mysql /usr/bin/mysqldump -u root --password=root fame > `date '+%Y-%m-%dT%H%M%S'`backup.sql
```

### 常见问题

* mysql的数据存在哪里？如果docker镜像被删除我的数据会丢失吗？

  在`docker-compose.yml`中的`fame-mysql`配置可以看到有个配置:
  ```
  volumes:
    - ~/.fame/mysql/mysql_data:/var/lib/mysql
  ```
  将docker镜像中的文件夹挂载到外部系统的文件夹，所以即使docker镜像被删除或者重装mysql数据都不会丢失，在系统的`~/.fame/mysql/mysql_data`文件夹中。

* 在管理后台上传的图片资源文件放在哪里？如果docker镜像被删除我的数据会丢失吗？

  在`docker-compose.yml`中的`fame-server`配置可以看到有个配置:
  ```
      volumes:
    - ~/.fame/upload:/root/.fame/upload
  ```
  将docker镜像中的文件夹挂载到外部系统的文件夹，所以即使docker镜像被删除或者重装上传的资源文件数据都不会丢失，在系统的`~/.fame/upload`文件夹中。

* 我想要看系统的日志，在哪里能看到

  在系统的`~/.fame/logs/`文件夹下会有Fame博客生成的日志，其中nginx文件夹为nginx生成的日志，server文件夹为fame-server的SpringBoot生成的日志

### 升级须知

由于以前fame-server项目的orm框架用的是`Mybaits`，现改成`SpringDataJpa`。为了使用`SpringDataJpa`的特性修改了一些数据库的结构，所以若升级版本需要再在数据库执行一下命令:

```sql
update fame.article set type = 'note' where type = 'page';
update fame.article set `status` = 'PUBLISH' where `status` = 'publish';
update fame.article set `status` = 'DRAFT' where `status` = 'draft';
update fame.article set `status` = 'DELETE' where `status` = 'delete';
alter table fame.comment modify column status VARCHAR(32);
update fame.comment set `status` = 'NORMAL' where `status` = 0;
update fame.comment set `status` = 'DELETE' where `status` = '-1';
update fame.article set allow_comment = 1 where type = 'post';
alter table fame.comment change p_id parent_id INT(11);
alter table fame.middle change a_id article_id INT(11);
alter table fame.middle change m_id meta_id INT(11);
```

如果是用docker-compose启动项目可以用一下命令进入fame-mysql服务容器中

```bash
docker exec -it fame-mysql bash
```