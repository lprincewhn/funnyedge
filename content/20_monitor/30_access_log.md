---
title: "使用Athena分析访问日志"
date: 2022-04-12T14:32:23+08:00
weight: 30
draft: false
---

1. 点击分配ID进入详情页，确认标准日志已经打开。
![分配详情](/images/cloudfront_standard_log.png?classes=border)

2. 点击“编辑”按钮进入编辑页面，确认或设置日志存储路径
![打开标准日志](/images/cloudfront_log_destication.png.png?classes=border)

3. （可选）如果还没有用于存储日志的S3桶，按照以下步骤进行场景。

    3.1 使用以下URL打开S3控制台：https://s3.console.aws.amazon.com/s3/buckets
    ![创建S3桶](/images/s3_create_bucket.png?classes=border)
    3.2 输入存储桶名称和选择区域，其余参数保留默认配置
    ![创建S3桶](/images/s3_create_bucket_for_cloudfront.png?classes=border)

3. 设置Athena
4. 创建表格
5. （可选）给日志存储桶添加生命周期策略，自动删除过期的日志，从而节省S3存储和Athena查询的成本。


