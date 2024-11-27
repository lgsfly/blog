## skywalking安装包

SkyWalking OAP ：https://github.com/apache/skywalking/releases/tag/v8.9.1

```Shell
[root@1fvbz6ehcv2u7p skywalking]# ll apache-skywalking-apm-8.9.1.tar.gz
-rw-r--r-- 1 root root 138319549 Mar  7 16:43 apache-skywalking-apm-8.9.1.tar.gz
[root@1fvbz6ehcv2u7p skywalking]#
[root@1fvbz6ehcv2u7p skywalking]# cd apache-skywalking-apm-bin
[root@1fvbz6ehcv2u7p apache-skywalking-apm-bin]# ll
total 92
drwxr-xr-x  2 root root   260 Mar  7 17:15 bin
drwxr-xr-x 12 root root  4096 Mar  7 16:51 config
drwxr-xr-x  2 root root    68 Mar  7 16:44 config-examples
-rwxrwxr-x  1 1001 1002 29794 Dec 10  2021 LICENSE
drwxrwxr-x  3 1001 1002  4096 Mar  7 16:44 licenses
drwxr-xr-x  2 root root    80 Mar  7 16:52 logs
-rwxrwxr-x  1 1001 1002 32519 Dec 10  2021 NOTICE
drwxrwxr-x  2 1001 1002 12288 Dec 10  2021 oap-libs
-rw-rw-r--  1 1001 1002  1951 Dec 10  2021 README.txt
drwxr-xr-x  3 root root    30 Mar  7 16:44 tools
drwxr-xr-x  2 root root   126 Mar  8 16:19 webapp
```

SkyWalking Java Agent：https://archive.apache.org/dist/skywalking/java-agent/8.10.0/apache-skywalking-java-agent-8.10.0.tgz

```Shell
[root@1fvbz6ehcv2u7p skywalking]# ll apache-skywalking-java-agent-8.10.0.tgz
-rw-r--r-- 1 root root 30206940 Mar 10 14:51 apache-skywalking-java-agent-8.10.0.tgz
[root@1fvbz6ehcv2u7p skywalking]#
[root@1fvbz6ehcv2u7p skywalking]#
[root@1fvbz6ehcv2u7p skywalking]# cd skywalking-agent/
[root@1fvbz6ehcv2u7p skywalking-agent]# ll
total 19952
drwxr-xr-x 2 501 games     4096 Mar 10 14:51 activations
drwxr-xr-x 2 501 games      131 Mar 10 14:51 bootstrap-plugins
drwxr-xr-x 2 501 games       26 Mar 10 14:51 config
-rw-r--r-- 1 501 games    12911 Apr 12  2022 LICENSE
drwxr-xr-x 2 501 games       29 Mar 10 14:51 licenses
drwxr-xr-x 2 501 games        6 Apr 12  2022 logs
-rw-r--r-- 1 501 games    10003 Apr 12  2022 NOTICE
drwxr-xr-x 2 501 games     4096 Mar 10 14:51 optional-plugins
drwxr-xr-x 2 501 games      131 Apr 12  2022 optional-reporter-plugins
drwxr-xr-x 2 501 games     8192 Mar 10 14:51 plugins
-rw-r--r-- 1 501 games 20378894 Apr 12  2022 skywalking-agent.jar
```

## SkyWalking OAP Server搭建

1. 配置 /apache-skywalking-apm-bin/config/application.yml

![img](https://nhegc93vpd.feishu.cn/space/api/box/stream/download/asynccode/?code=MjA5MDc2ZDJkNWU3YTNkYjI4OWQ5Y2IzYzA1YWIzZDdfQXpDVUM3bjFHZXlQOW11VDVRVkhmMEJSZU9ZUGlIVHFfVG9rZW46Ym94Y25ZbGluVXF1c3NOMnFKMFR2Q05kRGxmXzE3MzI2NzI4OTc6MTczMjY3NjQ5N19WNA)

1. 启动 SkyWalking OAP 服务

```Shell
[root@1fvbz6ehcv2u7p bin]# sh oapService.sh
SkyWalking OAP started successfully!
```

## SkyWalking UI 搭建

1. 配置 apache-skywalking-apm-bin/webapp/webapp.yml，主要是根据需要设置 UI端口以及 oap-service对应服务地址
2. 启动 UI

```Shell
[root@1fvbz6ehcv2u7p bin]# sh webappService.sh
SkyWalking Web Application started successfully!
```

1. 通过服务器加webapp.yml设置的端口可以直接访问界面

## SkyWalking UI登录认证

由于从6.2.0开始因为登录认证的安全漏洞问题移除了登录认证，这里采用nginx作权限配置，具体可以参考：

- nginx安装: https://blog.csdn.net/t8116189520/article/details/81909574
- nginx登录认证实现：https://www.imooc.com/article/294840

## SkyWalking Java Agent【Shell】

在启动项目的 Shell 脚本上，可以通过 `-javaagent` 参数进行配置 SkyWalking Java Agent。

- 上传 Agent 软件包
  -  我们需要将 `skywalking-agent` 目录，拷贝到 Java 应用所在的服务器上。这样，Java 应用才可以配置使用该 SkyWalking Agent。
- 配置 Java 启动脚本
  - ```Java
    # SkyWalking Agent 配置
    export SW_AGENT_NAME=demo-application # 配置 Agent 名字。一般来说，我们直接使用 Spring Boot 项目的 `spring.application.name` 。
    export SW_AGENT_COLLECTOR_BACKEND_SERVICES=127.0.0.1:11800 # 配置 Collector 地址。
    export SW_AGENT_SPAN_LIMIT=2000 # 配置链路的最大 Span 数量。一般情况下，不需要配置，默认为 300 。主要考虑，有些新上 SkyWalking Agent 的项目，代码可能比较糟糕。
    export JAVA_AGENT=-javaagent:/usr/lgs/skywalking/skywalking-agent/skywalking-agent.jar # SkyWalking Agent jar 地址。
    
    # Jar 启动
    java -jar $JAVA_AGENT -jar shareservice.RELEASE.jar
    ```
- 通过环境变量，进行配置。
- 更多的变量，可以在 `skywalking/skywalking-agent/config/agent.config` 查看。

## SkyWalking Java Agent【IDEA】

考虑到偶尔我们需要在 IDE 中，也希望使用 SkyWalking Agent，可参考下图配置：

![img](https://nhegc93vpd.feishu.cn/space/api/box/stream/download/asynccode/?code=N2I5OWE2NDk1YzU2NmIwNzgyN2RhYmViOTBiYmI1NDlfMGI3bk9SR0F1QTBmc0FIVjh2WlFTRFZ4UW1zd2VzMWpfVG9rZW46Ym94Y25FVWNrd3NaYXZsVjhYSkRaaEJXclZlXzE3MzI2NzI4OTc6MTczMjY3NjQ5N19WNA)

## SkyWalking 修改前端源码后重新发布前端包步骤

github地址

9.0.0+版本：https://github.com/apache/skywalking-booster-ui

8.0+版本：https://github.com/apache/skywalking-rocketbot-ui

1. 在webstorm进行二次开发
2. npm run build进行构建
3. 将构建好的dist/目录内容替换skywalking-[webapp](https://so.csdn.net/so/search?q=webapp&spm=1001.2101.3001.7020).jar 中\BOOT-INF\classes\public 下的内容
4. 解压等相关命令

```JSON
// 将原jar包解压
jar xf skywalking-webapp.jar
// 重新打包
jar cf0M skywalking-webapp.jar *
// 检测jar包
java -jar skywalking-webapp.jar
// 修改权限
chown -R 1001:1002 skywalking-webapp.jar
```

1. 在机器上，先kill掉原来的skywalking-webapp.jar包进程，将新打包的skywalking-webapp.jar包替换掉
2. 在/bin目录，sh webappService.sh 启动skywalking-webapp.jar包。