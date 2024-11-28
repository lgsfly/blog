---
layout: post
title:  "skywalking服务监控接入"
date:   2024-11-28 18:03:02 +0800
categories: skywalking
---

### 服务监控接入

1. ####  技术栈要求：

   * 采用版本 8.9.1 skywalking 作为链路和性能监控 APM工具，国产开源框架

   - 前端源码：https://github.com/apache/skywalking-rocketbot-ui/releases/tag/v8.9.1

   - 后端源码：https://github.com/apache/skywalking/releases/tag/v8.9.1

   - 客户端JavaScript 异常、性能和追踪库：https://github.com/apache/skywalking-client-js/releases/tag/v0.7.0

   - SkyWalking 的 Java 代理：https://github.com/apache/skywalking-java/releases/tag/v8.10.0

     

2. ####   业务服务应用如何接入服务监控，以下基于Linux机器进行操作：

   1. 下载 SkyWalking Java Agent：https://archive.apache.org/dist/skywalking/java-agent/8.10.0/apache-skywalking-java-agent-8.10.0.tgz


   2. 将agnet包解压安装到Java应用所在服务器，如下：

      ```shell
      [root@xxx skywalking-agent]# pwd
      /software/skywalking/skywalking-agent
      [root@xxx skywalking-agent]#
      [root@xxx skywalking-agent]# ll
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

      

   3. 通过环境变量，进行基本配置如下，更多配置可以参考：`/software/skywalking/skywalking-agent/config/agent.config`

      ```shell
      # SkyWalking Agent 配置
      export SW_AGENT_NAME=sharesevice # 配置 Agent 名字。一般来说，我们直接使用 Spring Boot 项目的 `spring.application.name` 。
      export SW_AGENT_COLLECTOR_BACKEND_SERVICES=127.0.0.1:11800 # 配置 Collector 地址，一般由业务中台提供
      export JAVA_AGENT=-javaagent:/software/skywalking/skywalking-agent/skywalking-agent.jar # SkyWalking Agent jar 地址。
      
      # Jar 启动
      java -jar $JAVA_AGENT -jar shareservice.jar
      ```

      

   4.  启动后可查看日志 `/software/skywalking/skywalking-agent/logs/skywalking-api.log` ，触发业务调用后可在业务中前端查看相关监控信息

      

1. ####  定制化Tag

   - 什么是Tag（标签）？

     -   Skywalking默认采集的信息不包括应用系统的业务信息，比如业务流水号、客户号等有助于识别和定位的关键信息。因此需要采集更多信息提供给Skywalking。这些信息被称为TAG，即key=value对

   - Tag的定制化一般有以下两类实现机制

     - **application-toolkit-trace**。 需要对应用代码进行简单修改，包括注解和源代码两种方式。可参考：https://skywalking.apache.org/docs/skywalking-java/v8.9.0/en/setup/service-agent/java-agent/application-toolkit-trace/

       <p style="color: red; font-weight: bold;">**注意：采用这类实现机制需要启用自定义增强插件，如果不做，将看不到增强信息。自定义增强插件即 apm-customize-enhance-plugin-8.10.0.jar 。这个官方插件的作用是使探针可以收集到@Trace之类自定义增加的注解。其位于agent目录中的optional-plugins目录中,需要将其拷贝至plugins目录!**</p>

         以下为采用注解方式示例：

       1.  在被采集的应用中增加依赖，pom.xml如下:

          ```java
          <dependency>
              <groupId>org.apache.skywalking</groupId>
              <artifactId>apm-toolkit-trace</artifactId>
              <version>8.9.0</version>
          </dependency>
          ```

          

       2.  在业务服务的代码中，使用**@Trace**、**@Tags**、**@****Tag**注解，自定义需要采集的信息，如下：**
     
          ```java
          @Trace(operationName = "setSeq")
          @Tags({@Tag(key = "mgmttestparam", value = "arg[1]"), @Tag(key = "mgmttestresult", value = "returnedObj")})
          @PostMapping("/business/setSeq")
          @ResponseBody
          public Order setOrderSeqTest(@RequestBody Order order) {
              Order orderRet = new Order();
              orderRet.setSeqId("tetsSeq_001");
              return orderRet;
          }
          ```

          

       3.  为了在Skywalking服务端界面中可以用Tag搜索，需要在Skywalking服务端config/application.yml文件中，在searchableTracesTags配置的tag列表中增加以上自定义的Tag名，如下图。然后重启Skywalking服务端。

          ![img](https://nhegc93vpd.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTI2YWIzNzA2YTYzNzZkZTgxOTIzY2Y1OGZjYzdhZGZfeUZWd0o5Y04wS1FMeXBYMXRZMFJsT3Q1MUExOWxjVm5fVG9rZW46Ym94Y243eGNtNEtuZjdFUHFYT3ozSFFhRzFnXzE3MzI3ODUyODM6MTczMjc4ODg4M19WNA)

       4.  重启业务服务，并发起调用，在业务中台可查看监控界面可以看到如下信息，且能通过自定义的Tag搜索

          ![img](https://nhegc93vpd.feishu.cn/space/api/box/stream/download/asynccode/?code=MDY4MmU1Mzg0ZjdhYTY0ZWNiZDgwMzJmODc0YzRhZTlfbUVXN2I1NEd6WXlMbFQ1TzVGNGhZb3p0VUFlRXlBN0tfVG9rZW46Ym94Y244TGdkd0l3WFNzWUpETW95THhtYkpiXzE3MzI3ODUyODM6MTczMjc4ODg4M19WNA)

       ![img](https://nhegc93vpd.feishu.cn/space/api/box/stream/download/asynccode/?code=NmMwODIzMzA4MjIzMjI2ODQxM2VlMTNiMjFkMDhmYTVfYlgzSFdURlRLVWY1ck5VMTh2clMyUm4zRUtMV2FjVWFfVG9rZW46Ym94Y25CR0gxQkl0R2NROTFiMVRXTGt0QXdnXzE3MzI3ODUyODM6MTczMjc4ODg4M19WNA)

       <p style="color: red; font-weight: bold;">以上注解方式存在的缺点：通过@Trace注解虽然能达到效果，但在调用链路会多一层，如上面的 setSeq。</p>
     
       以下为采用修改应用源代码方式示例：
     
       1. 通过源代码能获取到当前的Span，并在当前Span追加Tag信息，可解决上面注解方式调用链路多加一层的问题，代码如下：
          ```Java
          @PostMapping("/business/setSeq")
          @ResponseBody
          public Order setOrderSeqTest(@RequestBody Order order) {
              ActiveSpan.tag("mgmtApp", Thread.currentThread().getStackTrace()[2].getMethodName());
              Order orderRet = new Order();
              orderRet.setSeqId("tetsSeq_001");
              return orderRet;
          }
          ```
       2. 重启应用服务发起业务调用，在业务中台可查看监控效果如下：

          ![img](https://nhegc93vpd.feishu.cn/space/api/box/stream/download/asynccode/?code=MGIzNzJlOGY2NjdmNjMyYjM4MjMzODZiNWE3NzQwZTRfeW1kMmExakZMbGdpVmJRd3VJbHRMZk5DRFFzR1YxQUpfVG9rZW46Ym94Y25JQnJSd0ZnNGVHYXRKeWczTVZ5Q1lSXzE3MzI3ODUyODM6MTczMjc4ODg4M19WNA)

          ![img](https://nhegc93vpd.feishu.cn/space/api/box/stream/download/asynccode/?code=YjU5MGE1MDlhNDgxMzU2YTBmNTQ5OWRhYTVmZmRhYjBfTmpyR2FHU212b2tCN3RtUzF6Q292MGk0b1BTU2ZQTzBfVG9rZW46Ym94Y25KZDRVaHJ5cG8xamV4QnNPZDZNcXVjXzE3MzI3ODUyODM6MTczMjc4ODg4M19WNA)

       
     
     - **customize-enhance-trace** 。通过使用插件，完全不修改代码。可参考https://skywalking.apache.org/docs/skywalking-java/v8.9.0/en/setup/service-agent/java-agent/customize-enhance-trace 以上两种方式都需要修改源代码，存在侵入性。 还可以通过插件+配置方式，达到完全的零侵入性。
     
       1.  在客户端skywalking-agent目录下，将插件apm-spring-annotation-plugin-8.10.0.jar和apm-apm-customize-enhance-plugin-8.10.0.jar，从目录optional-plugins/拷贝至plugin/目录。
     
       2.  指定该插件的配置文件。在skywalking-agent目录下，将config/agent.config文件中，新增**plugin.customize.enhance_file** 属性，指向该插件配置文件的绝对路径。
     
       3.  新建该插件配置文件customize_enhance_trace.xml
     
          ```xml
          <?xml version="1.0" encoding="UTF-8"?>
          <enhanced>
              <class class_name="com.lgs.sharedservice.controller.TestController">
                  <method method="setOrderSeqTest(com.lgs.sharedservice.entity.Order)" operation_name="设置流水号测试接口" static="false">
                      <tag key="mgmtApp">arg[0]</tag>
                  </method>
              </class>
          </enhanced>
          ```
     
          
     
       4.  最终效果与采用修改应用源代码方式相同（**此处自行验证未通过，待查明原因？**）
