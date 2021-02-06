---
title: spring boot 单独控制某个类的日志级别
date: 2020-10-26 14:54:39
tags: java
---
日常开发中，有些同事对日志输出没有好好设计，导致不重要的日志满天飞，掩盖了真实重要的日志，为了方便调试，经常需要对某个类进行单独配置日志级别。
``` yaml
logging:
  level:
    io:
      seata:
        root: debug #core包全部 debug
        core:
          rpc:
            netty:
              AbstractRpcRemoting: info #针对单独类的配置 info 
              AbstractRpcRemotingClient: info #针对单独类的配置 info 
```