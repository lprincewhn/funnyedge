---
title: "CloudFront控制台介绍"
date: 2022-04-12T14:22:31+08:00
weight: 10
draft: false
---

### 打开控制台
使用以下URL打开CloudFront控制台: 
https://console.aws.amazon.com/cloudfront/v3/home?#/distributions
![查看分配列表](/images/list_distributions.png?classes=border)

### 字段说明
- ID：自动分配的13位随机字符串标识，如：EXXXXXXXXXXXX
- 说明：用户输入的描述性文字，可用于描述该分配的用途，业务等信息
- 域名：CloudFront分配的.cloudfront.net后缀域名，一般用于配置DNS解析中的CNAME值
- 备用域名（可选）：用户指定用来访问该分配的域名，一般称为加速域名，可配置多个。配置备用域名必须先上传包含该域名的证书，请参考["证书管理(可选)"](/10_quickstart/20_certificate/)
- 源：源站域名，可配置多个。有多个源站时可通过配置”缓存行为“选择源站。

### 其他操作
1. 如果控制台语言是英文，可按照以下步骤切换语言至中文: 
![控制台设置](/images/setting.png?classes=border)
![控制台语言](/images/language.png?classes=border)

2. 在控制台顶部搜索栏中输入AWS服务名称快速切换服务控制台
![切换服务](/images/services.png?classes=border)
