## 介绍
该项目使用Golang进行构建，具体参见：https://mlog.club


## 部署

### supervisor 守护进程的方式部署
安装 supervisor
```bash
sudo apt install supervisor
```
启动supervisor服务:
```bash
sudo supervisord -c /etc/supervisord.conf
```
编辑配置文件：
```yaml
[program:bbs-go-site]  ;程序名称
user=root  ;执行程序的用户
command=/home/senyas/go/bbs-min/server/bbs-go  ;执行的命令
directory=/home/senyas/go/bbs-min/ ;命令执行的目录
stopsignal=TERM  ;重启时发送的信号
autostart=true  
autorestart=true  ;是否自动重启
stdout_logfile=/var/log/bbs-go-stdout.log  ;标准输出日志位置
stderr_logfile=/var/log/bbs-go-stderr.log  ;标准错误日志位置
```

重启supervisor服务:
````bash 
sudo supervisorctl update # 更新配置文件并重启相关的程序
````

查看的运行状态：
```bash 
sudo supervisorctl status bbs-go-site
```

supervisr管理命令：
```bash 
supervisorctl status       # 查看所有任务状态
supervisorctl shutdown     # 关闭所有任务
supervisorctl start 程序名  # 启动任务
supervisorctl stop 程序名   # 关闭任务
supervisorctl reload       # 重启supervisor
```

> ⚠️注意⚠️\
一、Sql脚本会初始化默认管理员用户，用户名：admin，密码：123456 \
二、该Sql脚本只会创建启动时必须的表，bbs-go系统使用的其他表会在系统正确启动后自动创建；

```sql
CREATE DATABASE IF NOT EXISTS `bbsgo_db` DEFAULT CHARACTER SET utf8mb4;

USE bbsgo_db;
SET NAMES utf8mb4;

-- 初始化用户表
CREATE TABLE `t_user`
(
    `id`                 bigint(20) NOT NULL AUTO_INCREMENT,
    `username`           varchar(32)         DEFAULT NULL,
    `email`              varchar(128)        DEFAULT NULL,
    `email_verified`     tinyint(1) NOT NULL DEFAULT '0',
    `nickname`           varchar(16)         DEFAULT NULL,
    `avatar`             text,
    `background_image`   text,
    `password`           varchar(512)        DEFAULT NULL,
    `home_page`          varchar(1024)       DEFAULT NULL,
    `description`        text,
    `score`              bigint(20) NOT NULL,
    `status`             bigint(20) NOT NULL,
    `topic_count`        bigint(20) NOT NULL,
    `comment_count`      bigint(20) NOT NULL,
    `roles`              text,
    `forbidden_end_time` bigint(20) NOT NULL DEFAULT '0',
    `create_time`        bigint(20)          DEFAULT NULL,
    `update_time`        bigint(20)          DEFAULT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `username` (`username`),
    UNIQUE KEY `email` (`email`),
    KEY `idx_user_score` (`score`),
    KEY `idx_user_status` (`status`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

-- 初始化用户数据（用户名：admin、密码：123456）
INSERT INTO t_user (`id`, `username`, `nickname`, `avatar`, `email`, `password`, `status`, `create_time`, `update_time`,
                    `roles`, `description`, `topic_count`, `comment_count`, `score`)
SELECT 1,
       'admin',
       'bbsgo站长',
       '',
       'a@example.com',
       '$2a$10$ofA39bAFMpYpIX/Xiz7jtOMH9JnPvYfPRlzHXqAtLPFpbE/cLdjmS',
       0,
       (UNIX_TIMESTAMP(now()) * 1000),
       (UNIX_TIMESTAMP(now()) * 1000),
       'owner',
       '轻轻地我走了，正如我轻轻的来。',
       0,
       0,
       0
FROM DUAL
WHERE NOT EXISTS(SELECT * FROM `t_user` WHERE `id` = 1);


-- 初始化话题节点
CREATE TABLE `t_topic_node`
(
    `id`          bigint(20) NOT NULL AUTO_INCREMENT,
    `name`        varchar(32) DEFAULT NULL,
    `description` longtext,
    `sort_no`     bigint(20)  DEFAULT NULL,
    `status`      bigint(20) NOT NULL,
    `create_time` bigint(20)  DEFAULT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `name` (`name`),
    KEY `idx_sort_no` (`sort_no`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

INSERT INTO `t_topic_node` (`id`, `name`, `description`, `sort_no`, `status`, `create_time`)
SELECT 1, '默认节点', '', 0, 0, (UNIX_TIMESTAMP(now()) * 1000)
FROM DUAL
WHERE NOT EXISTS(SELECT * FROM `t_topic_node` WHERE `id` = 1);

-- 初始化系统配置表
CREATE TABLE `t_sys_config`
(
    `id`          bigint(20)   NOT NULL AUTO_INCREMENT,
    `key`         varchar(128) NOT NULL,
    `value`       text,
    `name`        varchar(32)  NOT NULL,
    `description` varchar(128) DEFAULT NULL,
    `create_time` bigint(20)   NOT NULL,
    `update_time` bigint(20)   NOT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `key` (`key`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

-- 初始化系统配置数据
INSERT INTO t_sys_config(`key`, `value`, `name`, `description`, `create_time`, `update_time`)
SELECT 'siteTitle',
       'bbs-go演示站',
       '站点标题',
       '站点标题',
       (UNIX_TIMESTAMP(now()) * 1000),
       (UNIX_TIMESTAMP(now()) * 1000)
FROM DUAL
WHERE NOT EXISTS(SELECT * FROM `t_sys_config` WHERE `key` = 'siteTitle');

INSERT INTO t_sys_config (`key`, `value`, `name`, `description`, `create_time`, `update_time`)
SELECT 'siteDescription',
       'bbs-go，基于Go语言的开源社区系统',
       '站点描述',
       '站点描述',
       (UNIX_TIMESTAMP(now()) * 1000),
       (UNIX_TIMESTAMP(now()) * 1000)
FROM DUAL
WHERE NOT EXISTS(SELECT * FROM `t_sys_config` WHERE `key` = 'siteDescription');

INSERT INTO t_sys_config (`key`, `value`, `name`, `description`, `create_time`, `update_time`)
SELECT 'siteKeywords',
       'bbs-go',
       '站点关键字',
       '站点关键字',
       (UNIX_TIMESTAMP(now()) * 1000),
       (UNIX_TIMESTAMP(now()) * 1000)
FROM DUAL
WHERE NOT EXISTS(SELECT * FROM `t_sys_config` WHERE `key` = 'siteKeywords');

INSERT INTO t_sys_config (`key`, `value`, `name`, `description`, `create_time`, `update_time`)
SELECT 'siteNavs',
       '[{\"title\":\"首页\",\"url\":\"/\"},{\"title\":\"话题\",\"url\":\"/topics\"},{\"title\":\"文章\",\"url\":\"/articles\"}]',
       '站点导航',
       '站点导航',
       (UNIX_TIMESTAMP(now()) * 1000),
       (UNIX_TIMESTAMP(now()) * 1000)
FROM DUAL
WHERE NOT EXISTS(SELECT * FROM `t_sys_config` WHERE `key` = 'siteNavs');

INSERT INTO t_sys_config (`key`, `value`, `name`, `description`, `create_time`, `update_time`)
SELECT 'defaultNodeId',
       '1',
       '默认节点',
       '默认节点',
       (UNIX_TIMESTAMP(now()) * 1000),
       (UNIX_TIMESTAMP(now()) * 1000)
FROM DUAL
WHERE NOT EXISTS(SELECT * FROM `t_sys_config` WHERE `key` = 'defaultNodeId');

INSERT INTO t_sys_config (`key`, `value`, `name`, `description`, `create_time`, `update_time`)
SELECT 'tokenExpireDays',
       '365',
       '用户登录有效期(天)',
       '用户登录有效期(天)',
       (UNIX_TIMESTAMP(now()) * 1000),
       (UNIX_TIMESTAMP(now()) * 1000)
FROM DUAL
WHERE NOT EXISTS(SELECT * FROM `t_sys_config` WHERE `key` = 'tokenExpireDays');

INSERT INTO t_sys_config (`key`, `value`, `name`, `description`, `create_time`, `update_time`)
SELECT 'scoreConfig',
       '{"postTopicScore":1,"postCommentScore":1,"checkInScore":1}',
       '积分配置',
       '积分配置',
       (UNIX_TIMESTAMP(now()) * 1000),
       (UNIX_TIMESTAMP(now()) * 1000)
FROM DUAL
WHERE NOT EXISTS(SELECT * FROM `t_sys_config` WHERE `key` = 'scoreConfig');
```

# 运行 site 端代码

## 处理 node 版本：
```baah
#切换到 v16.12.1 版本
sudo n 

# npm的软件源是在国外服务器的，安装起来可能比较慢，你也可以使用cnpm来安装依赖。首先要安装cnpm，安装命令如下：
npm install cnpm -g --registry=https://r.npm.taobao.org

# cnpm安装完后，在bbs-go/site目录下执行以下命令安装依赖：
cnpm install
```

## 运行
```bash
npm run dev
```

## 打包
我们线上部署是不能使用：npm run dev方式的，该方式为开发者模式启动，系统限制最多只能同时打开三个网页。

线上部署方式如下：
+ 使用npm run build编译site模块，编译成功后会在目录中生成.nuxt目录。
+ 使用npm run start启动服务，服务同样启动在：3000 端口。


# 配置 admin 端
## 安装依赖
```bash
cnpm install 
```

## 配置
admin 模块的配置文件分为开发环境和生产环境，文件分别为：
```yaml
/admin/.env.development
/admin/.env.production
```
在使用开发模式运行服务时使用/admin/.env.development配置文件，在打包时使用/admin/.env.production配置文件。
配置文件中主要有两个配置项：
```yaml
# 接口请求地址HOST，用于admin模块请求服务端接口
# 该配置的值一般设置为server端的HOST，或者site端的HOST（因为site端代理了server端的所有接口）
VUE_APP_BASE_API = 'http://localhost:8082'

# site模块访问根目录，作用：例如后台点击帖子标题时，能够正确跳转到帖子site端的访问路径
VUE_APP_BASE_URL = 'http://localhost:3000'
```

## 运行
```bash 
npm run serve
```
来启动admin服务，使用该命令启动服务，会使用配置：/admin/.env.development


## 打包
确保依赖安装成功、配置正确后，可以使用命令：

```yaml
npm run build
```
来打包bbs-go-admin模块，打包时会使用配置：/admin/.env.production
打包的成果为：/admin/dist/文件夹，将该文件夹部署到nginx或者其他web容器中即可正常访问。


# 参考安装地址
`https://docs.bbs-go.com/introduction.html`