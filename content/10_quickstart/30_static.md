---
title: "静态内容加速"
date: 2022-04-12T14:22:31+08:00
weight: 30
draft: false
---

1. 打开CloudFront控制台，点击"创建分配"按钮
https://us-east-1.console.aws.amazon.com/cloudfront/v3/home


2. 创建过程可指定的参数共分4部分: "源", "默认缓存行为", "函数关联", "设置"。
在"源域"中填入源站域名，其余保留默认配置，点击"创建分配"即可完成。

3. 创建过程其他常用参数意义如下：
- 源
    - 协议: 访问源站时使用HTTP还是HTTPS，如果选择"匹配查看器"，则使用和客户端一样的协议
    - HTTP端口和HTTPS端口: 源站的服务端口




