---
title: "证书管理(可选)"
date: 2022-04-12T14:22:31+08:00
weight: 20
draft: false
---

### 打开ACM证书管理控制台
https://us-east-1.console.aws.amazon.com/acm/home?region=us-east-1#/certificates

**注意：**
1. 只有“弗吉尼亚北部”区域的证书可以被CloudFront使用，申请和导入CloudFront使用的证书时需要注意所在区域。
2. 虽然ACM上可以支持4096位的RSA证书，但是CloudFront目前只支持2048位的RSA证书，因此大于2048位的RSA证书无法在CloudFront控制台看到。

![证书管理控制台区域](/images/acm_region.png?classes=border)

### 两种方式获得证书
1. 直接申请由AWS颁发的证书(免费)
![申请AWS颁发的证书](/images/reqest_certificate.png?classes=border)
![证书验证](/images/certifcate_verification.png?classes=border)
请求成功后回到证书列表页面，点击”证书ID“进入证书详情页面，在“域”中可看到用于验证的CNAME记录，将此处所有记录配置到DNS解析记录中，稍等几分钟即可完成验证获取证书
![DNS验证](/images/certificates_cname.png?classes=border)

    **注意：证书在过期前60天AWS会自动对符合条件的证书进行续订，续订时需要再次验证这些CNAME记录，因此可保留这些记录供续订使用**

    AWS证书特征请参考：https://docs.aws.amazon.com/zh_cn/acm/latest/userguide/acm-certificate.html

    AWS证书的续订条件请参考：https://docs.aws.amazon.com/zh_cn/acm/latest/userguide/dns-renewal-validation.html

2. 通过第三方购买证书然后将证书导入
![导入第三方证书](/images/import_certificate.png?classes=border)
![上传证书及私钥](/images/import_certificate_details.png?classes=border)

**注意：出于证书和私钥安全考虑，无论是AWS颁发的或是导入的第三方证书，一旦上传，没有任何办法可以导出或下载，因此需要在本地对证书和私钥进行妥善保管，以供未来使用**




