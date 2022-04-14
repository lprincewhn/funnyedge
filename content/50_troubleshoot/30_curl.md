---
title: "检查HTTP连接"
date: 2022-04-12T14:22:31+08:00
weight: 30
draft: false
---

### 有用的参数
- -v：显示连接细节
- -s：不显示进度条
- -w：打印指定参数
- -o /dev/null：不显示响应内容，一般做故障排查时不需要关心响应的内容
- -H：指定标头
- --resolve：指定连接地址

### 例子：

1. 指定节点检查HTTP连接过程
![HTTP连接](/images/http_connect.png?classes=border)

2. 测试HTTP下载性能
![HTTP下载性能](/images/http_performance.png?classes=border)