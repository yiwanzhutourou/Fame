# Fame博客系统文档

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

### 手动前后端分步部署

> 以下以Centos7为例

> 首先保证服务器拥有`java8` `mysql5.7.x` `maven3.3.x` `node10.x` `npm6.x`的环境（版本不一定要完全一样，但避免奇怪的问题出现，最好相同），并且已经安装了`git`。

#### 编译项目

> 如果不想手动编译项目可直接跳到[部署项目](#部署项目)


1. 克隆项目到本地

   ```
   git clone https://github.com/zzzzbw/Fame.git && cd Fame
   ```

2. 编译后端服务(fame-server)

  ```
    cd fame-server
    mvn -DskipTests=true package
  ```

  编译完成后`target/Fame-x.x.jar`即为后端服务包

3. 编译前端博客页面服务(fame-front)

  ```
    cd fame-front
    npm install && npm run build
  ```

  `.nuxt`为编译后的文件，但启动服务也需要`config``plugins``nuxt.config.js``package.json`，所以要移动文件的话不要忘了这几个一起移动。

4. 编译前端后台管理服务(fame-admin)
  
  ```
    cd fame-admin
    npm install && npm run build
  ```

  编译完成后dist文件夹即为前端后台管理页面文件

#### 部署项目

1. 下载编译完成文件(如果执行了上面手动编译则可以跳过这步)

   ```
   wget https://github.com/zzzzbw/Fame/releases/download/v1.0.0/Fame-release.tar.gz
   tar zxvf Fame-release.tar.gz
   ```

2. 启动后端服务器(fame-server)

    2.1 进入文件夹

    `cd fame-server`

    2.2 创建spring-boot生产环境配置文件

    `vi src/main/resources/application-prod.yml`

    ```yaml
    #datasource
    spring:
     datasource:
      driverClassName: com.mysql.cj.jdbc.Driver
       url: jdbc:mysql://localhost:3306/fame?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai
       username: root
       password: root
     jpa:
      show-sql: false
    #log
    logging:
     level:
      root: INFO
      org:
       springframework:
            web: INFO
     file: log/fame.log
    ```
    
    将配置中的`username`和`password`修改成对应你数据库的用户名密码。如果mysql不是安装在同一个服务器或者端口不是3306，则要对应修改`url`。

    > 你的`Mysql`中，要先有`fame`数据库。
    >
    > ```sql
    > CREATE DATABASE fame CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
    > ```

    日志文件会保存在`项目/fame-server/log`文件夹下，如果想要修改日志保存目录则修改`file`。

    2.3 启动项目

    `nohup java -jar Fame-1.0.jar --spring.profiles.active=prod`

    当执行`netstat -ntulp | grep 9090`有列出线程时，说明启动完成。

3. 启动前端博客页面服务(fame-front)

    3.1 进入文件夹

    `cd fame-front`

    3.2 安装依赖

    `npm install`

    3.3 启动服务

    `nohup npm run start`

    当执行`netstat -ntulp | grep 3000`有列出线程时，说明启动完成。


4. 配置nginx及前端后台管理服务(fame-admin)

    > 由于fame-admin项目是静态页面，必须要依靠服务器来使用，所以这里和nginx配置一起启动

    4.1 设置nginx配置

    ```
    # 反向代理server配置
    server {
      listen       80;
      charset utf-8;
   
      location /api/ {
        proxy_set_header   X-Real-IP $remote_addr; #转发用户IP
        proxy_pass http://127.0.0.1:9090; #fame-server
      }
       
      location /media/ {
        proxy_pass http://127.0.0.1:9090; #fame-server 资源文件
      }
       
      location /admin {
        proxy_pass   http://127.0.0.1:3001/; #fame-admin
      }
       
      location / {
        proxy_pass   http://127.0.0.1:3000/; #fame-front nuxt项目 监听端口
      }
       
      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
        root   /usr/share/nginx/html;
      }
    }
   
    # fame-admin server配置
    server {
      listen 3001;
      charset utf-8;
   
      location / {
        root  /usr/share/nginx/fame-admin/;
        try_files $uri $uri/ /index.html;
      }
    }
    ```

    4.2 拷贝fame-admin文件到nginx目录

    `cp -r fame-admin/ /usr/share/nginx/fame-admin/`

    > 这里拷贝的目标文件夹要和nginx配置文件中的fame-admin server配置里的root目录一致

    然后启动nginx即部署完成

    `systemctl start nginx.service`

### 开发环境启动

首先保证有 `java8` `mysql5.7.x` `maven3.3.x` `node10.x` `npm6.x`的环境(版本不一定要完全一样，但避免奇怪的问题出现，最好相同)

1. 克隆项目到本地

   ```
   git clone https://github.com/zzzzbw/Fame.git
   ```

2. 启动服务端 (项目使用lombok插件，如果要在ide中调试要有lombok插件)

    2.1 进入服务端文件夹

        `cd fame-server`

    2.2 修改spring-boot配置文件

      `vi src/main/resources/application-dev.yml`

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

3. 启动博客前端

    3.1 进入前端文件夹

      `cd fame-front`

    3.2 安装依赖和启动服务

      ```
    npm install
    npm run dev
      ```

    3.3 访问地址

      启动完成后，浏览器访问 `http://localhost:3000`

4. 启动博客管理后台

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