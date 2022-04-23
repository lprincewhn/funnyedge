---
title: "检查域名解析情况"
date: 2022-04-12T14:22:31+08:00
weight: 10
draft: false
---

通过dig命令可检查当前访问的PoP点及其IP：
```
# 正向解析获得节点IP
dig a.test.com +short
dig -x dxxxxxxx.cloudfront.net +short

# 反向解析查看IP归属
dig 1.1.1.1 +short
```
### 例子
![dig](/images/dig.png?classes=border)

### 工具

1. 检查全球DNS解析情况
    - https://dnschecker.org/
    - https://cachecheck.opendns.com/

2. 检查IP归属
    - http://ipip.net/
    - https://bgp.he.net/

3. 检查用户的dns resolver
    - https://ts.twister5.com/index.html?cdn=true


