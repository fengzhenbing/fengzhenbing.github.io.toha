---
title: "pulsar"
date: 2021-02-23T11:26:253+08:00
description: pulsar
menu:
  sidebar:
    parent: message
    name: pulsar
    identifier: pulsar
    weight: 5
---

### 相关概念



### 安装测试

```shell
# 下载
wget https://mirrors.bfsu.edu.cn/apache/pulsar/pulsar-2.7.1/apache-pulsar-2.7.1-bin.tar.gz
tar xvfz apache-pulsar-2.7.1-bin.tar.gz
cd apache-pulsar-2.7.1

# 单机启动
bin/pulsar standalone

# 消费消息
bin/pulsar-client consume my-topic -s "first-subscription"

# 生产消息
bin/pulsar-client produce my-topic --messages "hello-pulsar"
```



### 相关资料

[官网快速启动](http://pulsar.apache.org/docs/zh-CN/standalone/)

