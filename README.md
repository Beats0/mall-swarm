# mall-swarm

### 部署

 - windows: 搭建docker本地服务, 进入docker文件夹执行对应docker-compose即可, 安装 [nacos](https://github.com/alibaba/nacos/releases) 并 `startup.cmd -m standalone` 启动
 - [linux docker-compose 部署](https://www.macrozheng.com/mall/deploy/mall_deploy_docker_compose.html)
 - [linux docker 容器部署](https://www.macrozheng.com/mall/deploy/mall_swarm_deploy_docker.html)
 - [mall-swarm微服务K8S](https://www.macrozheng.com/mall/deploy/mall_swarm_deploy_k8s.html)
## 安装docker

执行docker文件夹中对应的 `docker-compose.yml`, 使用 `docker-compose up -d` 命令即可创建

### mysql 导入表

新建数据库 `mall`

导入 `document/sql/mall.sql` 文件即可

### logstash 安装 logstash-codec-json_lines

```shell
# 以root身份进入logstash容器
docker exec -it --user root 1837cf8cd4b9 /bin/bash

apt-get update
apt-get install vim -y
vim /usr/share/logstash/Gemfile
```

更改默认的 https://rubygems.org 为https://mirrors.tuna.tsinghua.edu.cn/rubygems

```diff
- source "https://rubygems.org"
+ source "https://mirrors.tuna.tsinghua.edu.cn/rubygems"
```

安装 logstash-codec-json_lines

```shell
/usr/share/logstash/bin/logstash-plugin install --no-verify logstash-codec-json_lines
```

然后重启container即可

### es-head 406 错误

进入 es-head 容器

```shell
docker exec -it bce0a7da59f7 sh
cd _site/
```

编辑vendor.js  共有两处

6886行
```diff
- contentType: "application/x-www-form-urlencoded",
+ contentType: "application/json;charset=UTF-8",
```

7574行
```diff
- var inspectData = s.contentType === "application/x-www-form-urlencoded" &&
+ var inspectData = s.contentType === "application/json;charset=UTF-8" &&
```


## 服务列表

配置见对应的docker-compose.yml

服务:

按顺序启动

| 名称           | 地址                             | 账号密码         | 备注                                                              |
|--------------|-----------------------------------|------------------|-------------------------------------------------------------------|
| 前端vue      | http://localhost:81               | admin:macro123   | 项目地址 [mall-admin-web](https://github.com/Beats0/mall-admin-web)|
| mall-gateway | http://localhost:8201             |                  | 网关服务                                                           |
| mall-auth    | http://localhost:8401             |                  | 认证中心                                                           |
| mall-admin   | http://localhost:8080             |                  | 后台管理服务                                                       |
| mall-portal  | http://localhost:8085             |                  | 前台服务                                                           |
| mall-search  | http://localhost:8081             |                  | 搜索服务                                                           |
| mall-monitor | http://localhost:8101             | macro:123456     | 监控中心                                                           |
| knife4j文档  | http://localhost:8201/doc.html    |                   | knife4j文档                                                       |

环境:

| 名称          | 地址                          | 账号密码                     | 备注                                                                                 |
|---------------|------------------------------|-----------------------------|--------------------------------------------------------------------------------------|
| mysql         | 3306                         | root:mysqlroot              |                                                                                      |
| redis         | 6379                         | redisroot@redisroot         |                                                                                      |
| mongodb       | 27017                        | admin:admin                 |                                                                                      |
| elasticsearch | http://localhost:9200        |                             | [测试elasticsearch](https://www.macrozheng.com/mall/architect/mall_arch_07.html)     |
| logstash      | 4560                         |                             |                                                                                      |
| kibana        | http://localhost:5601        |                             |                                                                                      |
| es-head       | http://localhost:9100        |                             |                                                                                      |
| nacos         | http://localhost:8848/nacos/ | nacos:nacos                 |                                                                                      |
| minio         | http://localhost:9001        | minioadmin:minioadmin       | 设置 Bucket 为 mall, 并将 Access Policy 设置为 Public                                  |
| rabbitmq      | http://localhost:15672       | guest:guest                 |                                                                                      |
| nginx         | 80/443                       |                             |                                                                                      |


监控:

| 名称                | 地址                           | 账号密码                  | 备注                                                            |
|---------------------|------------------------------|---------------------------|-----------------------------------------------------------------|
| SpringBootAdmin     | http://localhost:8101        | macro:123456              |                                                                 |
| prometheus          | http://localhost:9090        |                           |                                                                 |
| grafana             | http://localhost:3001        | admin:password            |                                                                 |
| cadvisor            | http://localhost:8090        |                           |                                                                 |
| node-exporter       | http://localhost:9101        |                           |                                                                 |
| mysqld-exporter     | http://localhost:9104        |                           |                                                                 |
| mogno-exporter      | http://localhost:9216        |                           |                                                                 |

## 项目介绍

`mall-swarm`是一套微服务商城系统，采用了 Spring Cloud 2021 & Alibaba、Spring Boot 2.7、Oauth2、MyBatis、Elasticsearch、Docker、Kubernetes等核心技术，同时提供了基于Vue的管理后台方便快速搭建系统。`mall-swarm`在电商业务的基础集成了注册中心、配置中心、监控中心、网关等系统功能。文档齐全，附带全套Spring Cloud教程。

## 系统架构图

![系统架构图](http://img.macrozheng.com/mall/project/mall_micro_service_arch.jpg)

## 组织结构

``` lua
mall
├── mall-common -- 工具类及通用代码模块
├── mall-mbg -- MyBatisGenerator生成的数据库操作代码模块
├── mall-auth -- 基于Spring Security Oauth2的统一的认证中心
├── mall-gateway -- 基于Spring Cloud Gateway的微服务API网关服务
├── mall-monitor -- 基于Spring Boot Admin的微服务监控中心
├── mall-admin -- 后台管理系统服务
├── mall-search -- 基于Elasticsearch的商品搜索系统服务
├── mall-portal -- 移动端商城系统服务
├── mall-demo -- 微服务远程调用测试服务
└── config -- 配置中心存储的配置
```

### 后端技术

| 技术                   | 说明                 | 官网                                                 |
| ---------------------- | -------------------- | ---------------------------------------------------- |
| Spring Cloud           | 微服务框架           | https://spring.io/projects/spring-cloud              |
| Spring Cloud Alibaba   | 微服务框架           | https://github.com/alibaba/spring-cloud-alibaba      |
| Spring Boot            | 容器+MVC框架         | https://spring.io/projects/spring-boot               |
| Spring Security Oauth2 | 认证和授权框架       | https://spring.io/projects/spring-security-oauth     |
| MyBatis                | ORM框架              | http://www.mybatis.org/mybatis-3/zh/index.html       |
| MyBatisGenerator       | 数据层代码生成       | http://www.mybatis.org/generator/index.html          |
| PageHelper             | MyBatis物理分页插件  | http://git.oschina.net/free/Mybatis_PageHelper       |
| Knife4j                | 文档生产工具         | https://github.com/xiaoymin/swagger-bootstrap-ui     |
| Elasticsearch          | 搜索引擎             | https://github.com/elastic/elasticsearch             |
| RabbitMq               | 消息队列             | https://www.rabbitmq.com/                            |
| Redis                  | 分布式缓存           | https://redis.io/                                    |
| MongoDb                | NoSql数据库          | https://www.mongodb.com/                             |
| Docker                 | 应用容器引擎         | https://www.docker.com/                              |
| Druid                  | 数据库连接池         | https://github.com/alibaba/druid                     |
| OSS                    | 对象存储             | https://github.com/aliyun/aliyun-oss-java-sdk        |
| MinIO                  | 对象存储             | https://github.com/minio/minio                       |
| JWT                    | JWT登录支持          | https://github.com/jwtk/jjwt                         |
| LogStash               | 日志收集             | https://github.com/logstash/logstash-logback-encoder |
| Lombok                 | 简化对象封装工具     | https://github.com/rzwitserloot/lombok               |
| Seata                  | 全局事务管理框架     | https://github.com/seata/seata                       |
| Portainer              | 可视化Docker容器管理 | https://github.com/portainer/portainer               |
| Jenkins                | 自动化部署工具       | https://github.com/jenkinsci/jenkins                 |
| Kubernetes             | 应用容器管理平台     | https://kubernetes.io/                               |

### 前端技术

| 技术       | 说明                  | 官网                           |
| ---------- | --------------------- | ------------------------------ |
| Vue        | 前端框架              | https://vuejs.org/             |
| Vue-router | 路由框架              | https://router.vuejs.org/      |
| Vuex       | 全局状态管理框架      | https://vuex.vuejs.org/        |
| Element    | 前端UI框架            | https://element.eleme.io/      |
| Axios      | 前端HTTP框架          | https://github.com/axios/axios |
| v-charts   | 基于Echarts的图表框架 | https://v-charts.js.org/       |


## 环境搭建

### 开发环境

| 工具          | 版本号 | 下载                                                         |
| ------------- | ------ | ------------------------------------------------------------ |
| JDK           | 1.8    | https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html |
| Mysql         | 5.7    | https://www.mysql.com/                                       |
| Redis         | 7.0    | https://redis.io/download                                    |
| Elasticsearch | 7.17.3 | https://www.elastic.co/cn/downloads/elasticsearch            |
| Kibana        | 7.17.3 | https://www.elastic.co/cn/downloads/kibana                   |
| Logstash      | 7.17.3 | https://www.elastic.co/cn/downloads/logstash                 |
| MongoDb       | 5.0    | https://www.mongodb.com/download-center                      |
| RabbitMq      | 3.10.5 | http://www.rabbitmq.com/download.html                        |
| nginx         | 1.22   | http://nginx.org/en/download.html                            |

### 搭建步骤

> Windows环境部署

- Windows环境搭建请参考：[mall-swarm在Windows环境下的部署](https://www.macrozheng.com/mall/deploy/mall_swarm_deploy_windows.html);
- `mall-admin-web`项目的安装及部署请参考：[mall前端项目的安装与部署](https://www.macrozheng.com/mall/deploy/mall_deploy_web.html);
- `ELK`日志收集系统的搭建请参考：[SpringBoot应用整合ELK实现日志收集](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html);
- 使用MinIO存储文件请参考：[前后端分离项目，如何优雅实现文件存储](https://www.macrozheng.com/mall/technology/minio_use.html);
- 读写分离解决方案请参考：[你还在代码里做读写分离么，试试这个中间件吧](https://www.macrozheng.com/project/gaea.html);
- `分布式事务`解决方案请参考：[使用Seata彻底解决Spring Cloud中的分布式事务问题！](https://www.macrozheng.com/cloud/seata.html) 。

> Docker环境部署

- 使用虚拟机安装CentOS7.6请参考：[虚拟机安装及使用Linux，看这一篇就够了](https://www.macrozheng.com/tool/linux_install.html);
- Docker环境的安装请参考：[开发者必备Docker命令](https://www.macrozheng.com/mall/reference/linux_command.html);
- 本项目Docker镜像构建请参考：[使用Maven插件为SpringBoot应用构建Docker镜像](https://www.macrozheng.com/mall/reference/docker_maven.html);
- 本项目在Docker容器下的部署请参考：[mall-swarm在Linux环境下的部署（基于Docker容器）](https://www.macrozheng.com/mall/deploy/mall_swarm_deploy_windows.html);
- 本项目使用Jenkins自动化部署请参考：[mall-swarm使用Jenkins实现自动化部署](https://www.macrozheng.com/mall/deploy/mall_swarm_deploy_jenkins.html) 。

> Kubernetes环境部署

- 本项目使用Kubernetes部署请参考：[mall-swarm微服务项目在K8S下的实践！](https://www.macrozheng.com/mall/deploy/mall_swarm_deploy_k8s.html)

## 运行效果展示

- 查看注册中心注册服务信息，访问地址：http://192.168.3.101:8848/nacos/

![](http://img.macrozheng.com/mall/project/mall_swarm_run_new_01.png)

- 监控中心应用信息，访问地址：http://192.168.3.101:8101

![](http://img.macrozheng.com/mall/project/mall_swarm_run_new_02.png)

![](http://img.macrozheng.com/mall/project/mall_swarm_run_new_03.png)

![](http://img.macrozheng.com/mall/project/mall_swarm_run_new_04.png)

- API文档信息，访问地址：http://192.168.3.101:8201

![](http://img.macrozheng.com/mall/project/mall_swarm_run_05.png)

- 日志收集系统信息，访问地址：http://192.168.3.101:5601

![](http://img.macrozheng.com/mall/project/mall_swarm_run_new_06.png)

- 可视化容器管理，访问地址：http://192.168.3.101:9000

![](http://img.macrozheng.com/mall/project/mall_swarm_run_new_07.png)

![](http://img.macrozheng.com/mall/project/mall_swarm_run_new_08.png)


## 许可证

[Apache License 2.0](https://github.com/macrozheng/mall-swarm/blob/master/LICENSE)

Copyright (c) 2018-2022 macrozheng
