+++
title = "有趣的CDN"
date = 2022-04-12T14:01:29+08:00
weight = 5
+++

# Amazon CloudFront

为了以更低的延迟向终端用户提供内容，Amazon CloudFront使用了一个包含超过310个节点（超过300个边缘站点和13 个区域性边缘缓存）的全球网络，该网络覆盖47个国家/地区的90余个城市。
![CloudFront边缘节点分布](/images/cloudfront_map.png?classes=border)

### 功能特性
- 通过具有自动化网络映射和智能路由的超过310个全球分散节点(PoPs)提供数据，从而减少延迟。
- 通过流量加密和访问控制提高安全性，并使用AWS Shield Standard防御DDoS攻击，无需额外费用。
- 通过整合请求、可自定义的定价选项以及免费从AWS源传出数据来降低成本。
- 使用无服务器计算功能自定义您在AWS内容分发网络(CDN)边缘运行的代码，以平衡成本、性能和安全性。

### 参考文档
1. 用户指南：
https://docs.aws.amazon.com/zh_cn/AmazonCloudFront/latest/DeveloperGuide/Introduction.html

2. 边缘节点IP地址范围：
https://d7uri8nf7uskq.cloudfront.net/tools/list-cloudfront-ips



