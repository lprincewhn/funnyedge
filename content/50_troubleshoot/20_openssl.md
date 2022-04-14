---
title: "检查SSL连接"
date: 2022-04-12T14:22:31+08:00
weight: 20
draft: false
---

通过openssl命令向服务器发起SSL连接，可检查证书交互过程以及相关参数，连接建立后也可直接发送数据，因此可以非常方便的构造HTTP请求进行测试和验证。

### 有用的参数
- -servername：指定要访问的域名，即SSL协议中的SNI字段。当同一个IP上承载多个域名的服务时，服务器需要通过该字段确定应该返回哪个域名的证书。
- -reconcet：多次使用同一个session id进行连接，可用于测试对基于session id的SSL会话恢复的支持
- -quiet：不显示连接细节
- -sess_out：把服务器返回的session ticket保存到本地文件，供未来重用
- -sess_in：指定session ticet文件，可用于测试对基于session ticket的SSL会话恢复的支持
- -no_ssl，-no_tls1，-no_tls1_1，-no_tls1_2，-no_tls1_3，禁用TLS版本，用于测试服务器对TLS版本的支持情况
- -tls1，-tls1_1，-tls1_2，-tls1_3，指定TLS版本，用于测试服务器对TLS版本的支持情况

### 例子
1. 通过openssl检查服务器证书
![openssl连接](/images/openssl_connect.png?classes=border)
2. 通过openssl检查SSL会话恢复情况和发送请求
![openssl会话恢复](/images/openssl_ticket.png?classes=border)
3. 测试服务器的keep-alive时间
![连接保持](/images/openssl_keep_alive.png?classes=border)


