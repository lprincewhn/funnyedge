---
title: "静态内容加速"
date: 2022-04-12T14:22:31+08:00
weight: 30
draft: false
---

1. 打开CloudFront控制台，点击"创建分配"按钮
https://us-east-1.console.aws.amazon.com/cloudfront/v3/home#/distributions
![创建分配](/images/create_distribution.png?classes=border)
2. 创建过程可指定的参数共分4部分: "源", "默认缓存行为", "函数关联", "设置"。

    2.1 在“源”->”源域“中填入源站域名，并选择源站支持的协议。
    ![配置源站](/images/create_distribution_origin.png?classes=border)
    2.2 在”设置“->“备用域名(CNAME)”中“添加项目”，指定用来访问该分配的域名，也称加速域名，并选择证书及TLS安全策略。
    ![配置域名](/images/create_distribution_setting.png?classes=border)

3. 其余保留默认配置，点击"创建分配"即可完成，刚创建的分配的状态是“部署“，稍等几分钟待配置下发到全球所有边缘节点加速效果即可生效。
![配置域名](/images/create_distribution_deploying.png?classes=border)




