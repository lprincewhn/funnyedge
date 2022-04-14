---
title: "动态接口加速"
date: 2022-04-12T14:32:23+08:00
weight: 40
draft: false
---

Amazon CloudFront在全球拥有数以千计的第1/2/3层电信运营商，与所有主要接入网络连接良好，可实现最佳性能，并具有数百TB的部署容量。CloudFront边缘站点通过AWS网络骨干网与AWS区域连接。骨干网络是一种完全冗余的、环绕全球的多个100GbE平行光纤，并与数以万计的网络连接，以改进来源获取和动态内容加速。

1. 打开CloudFront控制台，点击"创建分配"按钮
https://us-east-1.console.aws.amazon.com/cloudfront/v3/home#/distributions
![创建分配](/images/create_distribution.png?classes=border)
2. 创建过程可指定的参数共分4部分: "源", "默认缓存行为", "函数关联", "设置"。

    2.1 在“源”->”源域“中填入源站域名，并选择源站支持的协议。
    ![配置源站](/images/create_distribution_origin.png?classes=border)
    2.2 在“默认缓存行为”->”查看器“中选择缓存策略和源请求策略。
    ![配置策略](/images/create_distribution_httpmethod.png?classes=border)
    2.3 在“默认缓存行为”->”缓存键和源请求“中选择缓存策略和源请求策略。
    ![配置策略](/images/create_distribution_policy.png?classes=border)
    2.4 在”设置“->“备用域名(CNAME)”中“添加项目”，指定用来访问该分配的域名，也称加速域名，并选择证书及TLS安全策略。
    ![配置域名](/images/create_distribution_setting.png?classes=border)

3. 其余保留默认配置，点击"创建分配"即可完成，刚创建的分配的状态是“部署“，稍等几分钟待配置下发到全球所有边缘节点加速效果即可生效。
![配置域名](/images/create_distribution_deploying.png?classes=border)








